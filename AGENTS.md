This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

Ramp Monitoring polls Ramp every 5 minutes for new card transactions, classifies each one against an approved-vendor list (APPROVED, UNRECOGNIZED, DUPLICATE, HIGH-VALUE, POLICY-FLAG), and posts a Slack card for everything except APPROVED. It also answers @mentioned questions about spend — *"anything unusual today?"*, *"trailing-30-day spend with merchant X?"*, *"is vendor Y approved?"*. Approving or rejecting a transaction is a write and always confirm-then-execute.

## Setup

### Connectors

**Important: Ramp is not in the Valet catalog.** This template ships ready to deploy on Slack alone — the heartbeat won't post anything until a Ramp connector is attached, and SOUL.md handles that gracefully (one-time DM hint to the install user, then silence). To enable the heartbeat, pick **one** of the two options below.

#### Option 1 — Custom Ramp MCP server

Wrap Ramp's REST API in a generic MCP server (Ramp's API docs are at <https://docs.ramp.com/developer-api/v1/overview/introduction>). At minimum the server needs to expose:

- `list_transactions(since_id, limit)` — list new card transactions newer than a cursor.
- `get_transaction(id)` — fetch a single transaction by id.
- `update_transaction(id, status)` — approve / reject (only used by the interactive write flow).
- `list_vendors()` — optional; if present the agent can suggest adding new merchants to your Ramp vendor list.

Then register it as a Valet connector:

```
valet connectors create mcp-server ramp-mcp \
  --org <org> \
  --transport <stdio|streamable-http> \
  --command <cmd> \
  --args <args> \
  --env RAMP_CLIENT_ID={{RAMP_CLIENT_ID}} \
  --env RAMP_CLIENT_SECRET={{RAMP_CLIENT_SECRET}}
```

…and attach it to the agent (`valet agents connect ramp-monitoring ramp-mcp`).

#### Option 2 — Zapier bridge

`zapier-mcp` IS in the Valet catalog and bridges to Ramp via Zapier integrations. This is the lower-lift path if you don't want to host a Ramp MCP server yourself.

1. In Zapier, create an integration to your Ramp account and expose the actions you need (list transactions, etc).
2. Create the catalog connector: `valet connectors create zapier-mcp --org <org>`.
3. Attach it to the agent.
4. Set `ZAPIER_MCP_TOKEN` (see Secrets) so the connector can authenticate.

### Channels

- **slack** (slack): The agent's per-agent Slack bot. Listens for @mentions and replies in-thread, and posts anomaly cards to whichever channels the bot has been invited to (preferring `#procurement` / `#finance` / `#spend` / `#expenses` if any of those exist among its memberships). Slack writes use the auto-injected outbound Slack connector.
- **heartbeat** (heartbeat): Fires every 5 minutes. Polls Ramp for new transactions, classifies them, posts non-APPROVED to Slack, and updates MEMORY.md. Declared inline in `valet.yaml`, so it's created automatically by the dashboard setup flow.

### Secrets

Set these only if the corresponding option is in use.

- **RAMP_CLIENT_ID** + **RAMP_CLIENT_SECRET** (Option 1): OAuth client credentials for Ramp's REST API. Create at Ramp Dashboard → Developer → API Keys. The custom MCP server uses these to authenticate every request. Org-scoped.
- **ZAPIER_MCP_TOKEN** (Option 2): Auth token for the Zapier MCP server. Generate it in Zapier when you set up the MCP integration. Org-scoped.

### External Setup

1. **Pick a connector path** (Custom MCP or Zapier bridge — see above) and complete it post-deploy. Until that's done, the heartbeat will DM the install user once with a setup hint and then stay silent.
2. **Slack**: invite the agent's bot to `#procurement` (or wherever you want anomaly cards to land). The agent posts to every channel it's a member of — invite it to one focused channel for cleanest signal, plus any channel where teammates should be able to @mention it for ad-hoc spend questions.
3. **Configure the approved-vendor list**: set `APPROVED_VENDORS` on the agent to a JSON array (`["AWS", "Notion", "Linear", "Vercel"]`) or comma-separated string of merchant names you've already vetted. If `APPROVED_VENDORS` is unset, the agent runs in review-everything mode and treats every merchant as UNRECOGNIZED — fine for the first day or two while you build the list, noisy after that.

## Customizing

- **Change the heartbeat interval**: edit `every: 5m` on the `heartbeat` channel in `valet.yaml`, then redeploy. `1m` is the lower bound; `15m` reduces load if your spend volume is low.
- **Tune the approved-vendor list**: edit `APPROVED_VENDORS` env on the agent. New merchants flagged as UNRECOGNIZED can be added directly from a Slack @mention (`@ramp-monitoring add Notion to the approved list` — confirm-then-execute).
- **Tune classification thresholds**: set `HIGH_VALUE_THRESHOLD` (in dollars, default `1000`) to change the high-value cutoff. Set `POLICY_KEYWORDS` (comma-separated) to override the default policy-flag keywords (`subscription, recurring, auto-renew`).
- **Watch a specific Ramp account or department**: most Ramp MCP wrappers expose a `department_id` or `card_program_id` filter — pass it in via env var on the agent and reference it in the SOUL workflow.
