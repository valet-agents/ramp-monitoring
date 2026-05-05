This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

Ramp Monitoring polls Ramp once a day for new card transactions, classifies each one against an approved-vendor list (APPROVED, UNRECOGNIZED, DUPLICATE, HIGH-VALUE, POLICY-FLAG), and posts a Slack card for everything except APPROVED. It also answers @mentioned questions about spend — *"anything unusual today?"*, *"trailing-30-day spend with merchant X?"*, *"is vendor Y approved?"*. Approving or rejecting a transaction is a write and always confirm-then-execute.

## Setup

### Connectors

- **ramp-mcp**: The Ramp MCP server, OAuth-authenticated. The agent uses it to list and inspect card transactions, classify each new charge against your approved-vendor list, and (only when explicitly asked and confirmed in Slack) approve or reject transactions. Add it from the catalog at the org level so other Ramp-powered agents can share it.

### Channels

- **slack** (slack): The agent's per-agent Slack bot. Listens for @mentions and replies in-thread, and posts anomaly cards to whichever channels the bot has been invited to. Slack writes use the auto-injected outbound Slack connector.
- **heartbeat** (heartbeat): Fires once a day. Polls Ramp for new transactions, classifies them, posts non-APPROVED to Slack, and updates MEMORY.md. Declared inline in `valet.yaml`, so it's created automatically by the dashboard setup flow.

### Secrets

This agent uses the OAuth variant of the Ramp MCP, so no API client ID or secret is needed at the org or agent level. The OAuth grant happens in the dashboard setup flow when you connect Ramp.

### External Setup

1. After deploy, OAuth into Ramp from the dashboard setup flow — no API client ID or secret to paste.
2. **Slack**: invite the agent's bot to whichever channel(s) you want anomaly cards in (typically `#procurement` or `#finance`). The agent posts to every channel it's a member of — invite it to one focused channel for cleanest signal, plus any channel where teammates should be able to @mention it for ad-hoc spend questions.
3. **Configure the approved-vendor list**: set `APPROVED_VENDORS` on the agent to a JSON array (`["AWS", "Notion", "Linear", "Vercel"]`) or comma-separated string of merchant names you've already vetted. If `APPROVED_VENDORS` is unset, the agent runs in review-everything mode and treats every merchant as UNRECOGNIZED — fine for the first day or two while you build the list, noisy after that.

## Customizing

- **Change the heartbeat interval**: edit `every: 24h` on the `heartbeat` channel in `valet.yaml`, then redeploy. `24h` is the default (one daily review); drop to `1h` during active month-end review when you want anomalies caught fast.
- **Tune the approved-vendor list**: edit `APPROVED_VENDORS` env on the agent. New merchants flagged as UNRECOGNIZED can be added directly from a Slack @mention (`@ramp-monitoring add Notion to the approved list` — confirm-then-execute).
- **Tune classification thresholds**: set `HIGH_VALUE_THRESHOLD` (in dollars, default `1000`) to change the high-value cutoff. Set `POLICY_KEYWORDS` (comma-separated) to override the default policy-flag keywords (`subscription, recurring, auto-renew`).
- **Watch a specific Ramp account or department**: most Ramp MCP wrappers expose a `department_id` or `card_program_id` filter — pass it in via env var on the agent and reference it in the SOUL workflow.
