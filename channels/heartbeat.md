# Spend Watcher (Heartbeat)

The heartbeat channel fires once a day. There is no
payload to parse — your job is to poll Ramp for new
transactions, classify them against the approved-vendor list,
and post a Slack card for anything that isn't APPROVED.

## Steps

1. **Read MEMORY.md** for the `ramp-monitoring` block.
   Extract `last_transaction_id` and the `recent` window. If
   the block is missing, this is the first run after deploy
   — record the current latest transaction id from Ramp via
   `ramp-mcp`, seed an empty `recent`, and exit silently. Do
   not backfill historical spend.
2. **Pull new transactions** via `ramp-mcp` — list
   transactions newer than `last_transaction_id`, ordered by
   `created_at` ascending. If zero new transactions, exit
   silently.
3. **Classify** each new transaction per SOUL **Phase 2**
   (DUPLICATE > HIGH-VALUE > POLICY-FLAG > UNRECOGNIZED >
   APPROVED — first match wins). APPROVED transactions get
   no Slack post.
4. **Resolve target channels** per the SOUL **Where to post**
   rules — prefer a channel matching `procurement` /
   `finance` / `spend` / `expenses` among the bot's
   memberships; fall back to every channel the bot is in;
   fall back to a DM to the install user if the bot is in
   zero channels.
5. **Post non-APPROVED cards** per the SOUL **Card Format**
   section — one Slack `mrkdwn` message per flagged
   transaction. Cap to 5 per fire (priority order:
   DUPLICATE > HIGH-VALUE > POLICY-FLAG > UNRECOGNIZED). If
   capped, append one trailing line: `…and N more flagged
   this tick — run \`@ramp-monitoring anything unusual
   today?\` for the full list.`
6. **Update MEMORY.md** — set `last_transaction_id` to the
   newest id processed, set `last_seen_at` to the current ISO
   timestamp, append every processed transaction to `recent`,
   and trim `recent` entries older than 7 days.
7. Your turn ends. No follow-ups, no thread replies, no
   retries on failure — the next heartbeat is the recovery.

## Skip conditions

Skip posting (and stop silently) if any of these are true:

- Zero new transactions since `last_transaction_id`.
- Every new transaction classifies as APPROVED.
- This is the first run after deploy. Seed
  `last_transaction_id` to the current latest and start
  watching forward — don't backfill.
