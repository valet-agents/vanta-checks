# Slack Message Received

The Slack event payload is appended directly after these
instructions in the user message. Parse it inline — do not fetch,
list, or search for the payload elsewhere. Do NOT use tools to
read the payload.

## Quick Filter — Exit Early If Not Relevant

Before doing anything else, check whether this message is worth
responding to. **Stop immediately and take no action** if ANY of
these are true:

- The message is from a bot (check for `bot_id` or
  `subtype: "bot_message"` in the payload).
- The message is from yourself.
- The message is a channel join/leave, topic change, pin, or other
  system event (any non-empty `subtype` that isn't a real user
  message).
- The message body, after stripping your @mention, is empty or
  just a greeting / thank-you / emoji.
- You're not @mentioned and the message isn't in a thread you
  already replied in.

If you are unsure whether the message is relevant, err on the side
of NOT responding.

## Scope

Extract the `channel` and `ts` (or `thread_ts`) from the payload.
All replies MUST go to this channel and thread. Do not read or
act on messages from other channels or threads.

## Steps

1. Extract `channel`, `ts`, `thread_ts` (if present), `user`, and
   `text` from the event payload.
2. Apply the Quick Filter above. If the message fails the filter,
   **stop here — do nothing**.
3. Strip your @mention token from `text` to get the raw question.
4. **Check connector availability**: if the `vanta-mcp` connector
   is not attached to this agent, reply once in-thread with the
   AGENTS.md setup snippet — the three `valet …` lines from the
   SOUL "Skip-if-not-configured" rule. Do not attempt any Vanta
   queries. Stop.
5. Otherwise, treat the message as a read-only Vanta question
   (failing tests, evidence due, access changes, control status).
   Pick the smallest set of `vanta-mcp` queries that answer the
   question.
6. Format the result per the SOUL "Read-only questions" guidance
   — short, identifier-prefixed bullets. Always cite the Vanta
   artifact link.
7. Reply in the thread using `thread_ts` if present, otherwise
   `ts`. One reply per mention.
