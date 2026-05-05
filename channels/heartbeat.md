# Vanta Heartbeat

The heartbeat channel fires once on its 5-minute interval. There
is no payload to parse — your job is to poll Vanta for new
compliance events and post per-event cards to Slack.

## Steps

1. **Check connector availability**: if the `vanta-mcp` connector
   is not attached to this agent, follow the SOUL
   **Skip-if-not-configured** rule:
   - First heartbeat after deploy: DM the workspace install user
     once with the AGENTS.md setup snippet, then update MEMORY's
     `vanta_setup_hint_sent: true`.
   - Subsequent heartbeats: stay silent. Stop.
2. Otherwise, follow the **Heartbeat Workflow** in SOUL.md
   (Phases 2–4):
   - Poll `list_tests` (failing), `list_evidence_needed_soon`
     (14 days), and `list_access_changes` (since
     `last_access_check`).
   - Diff against MEMORY.md to find new events in each category.
   - For each new event, post one Slack card per the SOUL
     **Card Formats** section (Failing Test / Evidence Due /
     Access Change). One event → one card.
   - Update MEMORY.md with the new event IDs and a fresh
     `last_heartbeat` and `last_access_check` timestamp.
3. Resolve target Slack destinations per the SOUL **Where to
   post** section: list every channel the bot is a member of
   and post once to each. If the bot is in zero channels, DM the
   workspace install user instead.
4. Do not retry on per-card failure — log the error in your
   session and continue with the remaining events. The next
   heartbeat is the recovery.
5. Do not send any follow-ups, reactions, or thread replies after
   the initial cards. Your turn ends after the posts complete.

## Skip conditions

Skip posting (and stop silently) if any of these are true:

- The `vanta-mcp` connector is not attached AND the
  `vanta_setup_hint_sent` flag in MEMORY is already `true`.
- All three Vanta polls return zero new events since the last
  heartbeat. (This is the steady-state norm — no news is good
  news.)
- A Vanta call errors transiently (timeout, 5xx). Log and skip
  that category for this tick; the next heartbeat retries.
