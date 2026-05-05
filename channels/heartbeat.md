# Spend Watcher (Heartbeat)

The heartbeat channel fires every 5 minutes. There is no
payload to parse — your job is to poll Ramp for new
transactions, classify them against the approved-vendor list,
and post a Slack card for anything that isn't APPROVED.

## Steps

1. **Connector check**: if neither `ramp-mcp` nor `zapier-mcp`
   is attached to the agent, follow the SOUL **Skip if not
   configured** rule — DM the install user once on the first
   fire with a setup hint, then stay silent on every
   subsequent fire until a connector is attached.
2. **Read MEMORY.md** for the `ramp-monitoring` block.
   Extract `last_transaction_id` and the `recent` window. If
   the block is missing, this is the first run after deploy
   — record the current latest transaction id from Ramp,
   seed an empty `recent`, and exit silently. Do not backfill
   historical spend.
3. **Pull new transactions** via the connector — list
   transactions newer than `last_transaction_id`, ordered by
   `created_at` ascending. If zero new transactions, exit
   silently.
4. **Classify** each new transaction per SOUL **Phase 3**
   (DUPLICATE > HIGH-VALUE > POLICY-FLAG > UNRECOGNIZED >
   APPROVED — first match wins). APPROVED transactions get
   no Slack post.
5. **Resolve target channels** per the SOUL **Where to post**
   rules — prefer a channel matching `procurement` /
   `finance` / `spend` / `expenses` among the bot's
   memberships; fall back to every channel the bot is in;
   fall back to a DM to the install user if the bot is in
   zero channels.
6. **Post non-APPROVED cards** per the SOUL **Card Format**
   section — one Slack `mrkdwn` message per flagged
   transaction. Cap to 5 per fire (priority order:
   DUPLICATE > HIGH-VALUE > POLICY-FLAG > UNRECOGNIZED). If
   capped, append one trailing line: `…and N more flagged
   this tick — run \`@ramp-monitoring anything unusual
   today?\` for the full list.`
7. **Update MEMORY.md** — set `last_transaction_id` to the
   newest id processed, set `last_seen_at` to the current ISO
   timestamp, append every processed transaction to `recent`,
   and trim `recent` entries older than 7 days.
8. Your turn ends. No follow-ups, no thread replies, no
   retries on failure — the next heartbeat is the recovery.

## Skip conditions

Skip posting (and stop silently) if any of these are true:

- Neither `ramp-mcp` nor `zapier-mcp` is attached **and** the
  one-time setup hint has already been sent (see step 1).
- Zero new transactions since `last_transaction_id`.
- Every new transaction classifies as APPROVED.
- This is the first run after deploy. Seed
  `last_transaction_id` to the current latest and start
  watching forward — don't backfill.
