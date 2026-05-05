This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

## Setup

### Connectors

**Vanta is not in the Valet catalog.** This template ships ready to deploy on Slack alone — the heartbeat will detect the missing connector and DM you a one-time setup hint, then stay silent until you wire Vanta in.

To enable the Vanta heartbeat, install a Vanta MCP server as a custom connector after deploy. A reasonable starting point is wrapping Vanta's REST API (https://developer.vanta.com/) in a generic MCP server; pick the implementation that matches your runtime (`stdio` or `streamable-http`).

The agent expects a Vanta MCP server that exposes (at minimum):

- `list_tests` — filterable by `failing` status
- `list_findings` — by severity and affected system
- `list_evidence_needed_soon` — controls with due dates in a window
- `list_access_changes` — access grants/revocations since a timestamp

Once you've chosen an implementation, run (substituting your transport, command, and args):

```
valet connectors create mcp-server vanta-mcp \
  --org <org> \
  --transport <stdio|streamable-http> \
  --command <cmd> \
  --args <args> \
  --env VANTA_API_TOKEN={{VANTA_API_TOKEN}}

valet connectors attach vanta-mcp --agent vanta-checks

valet secrets set VANTA_API_TOKEN=<your-token> --org <org>
```

The next heartbeat (within 5 minutes) will detect the connector and start polling Vanta.

### Channels

- **slack** (slack): The agent's per-agent Slack bot. Listens for @mentions and replies in-thread, and posts per-event compliance cards to whichever channels the bot has been invited to. Slack writes use the auto-injected outbound Slack connector.
- **heartbeat** (heartbeat): Fires every 5 minutes (`every: 5m`). Declared inline in `valet.yaml`, so it's created automatically by the dashboard setup flow. Polls Vanta for failing tests, evidence due in the next 14 days, and recent access changes.

### Secrets

- **VANTA_API_TOKEN**: Your Vanta API token, used by the custom `vanta-mcp` connector to authenticate against the Vanta API. Generate one in the Vanta admin console (Settings → API Tokens) with read scopes for tests, findings, controls, and access records. **Set after** you've created and attached the `vanta-mcp` connector — the agent doesn't need this secret until then.

### External Setup

1. **Deploy the agent first** — it works on Slack alone (interactive @mention Q&A will reply with the setup hint until Vanta is wired in).
2. After deploy, invite the agent's Slack bot to your compliance/audit channel(s). The agent posts per-event cards to every channel it's a member of. If the bot has not been invited anywhere, the first heartbeat sends the cards as a DM to the workspace install user with a one-line nudge.
3. Generate a Vanta API token in the Vanta admin console (Settings → API Tokens) with read access to tests, findings, controls, and access records.
4. Pick a Vanta MCP server implementation (or write your own around https://developer.vanta.com/) that exposes the four tools listed under **Connectors** above.
5. Run the three `valet …` commands from the **Connectors** section to create the connector, attach it, and store the token.
6. The next 5-minute heartbeat will detect the connector and start posting per-event cards. To smoke-test sooner, @mention the bot in Slack with *"what's failing right now?"* — that exercises the Slack + Vanta path without waiting for the heartbeat.

## Customizing

- **Change the heartbeat interval**: edit `every` on the `heartbeat` channel in `valet.yaml` (e.g. `15m`, `1h`), then redeploy. Five minutes is the default for live audits; one hour is reasonable in steady state.
- **Watched categories**: the SOUL workflow polls failing tests, evidence due in 14 days, and access changes. Edit the **Heartbeat Workflow** section in `SOUL.md` to add or remove categories (e.g. drop access changes if your workspace audit log is too noisy, or add `list_findings` for a separate severity-grouped post).
- **Control where cards post**: invite or remove the bot from channels in Slack — that's the only signal the agent uses. There is no channel name in the configuration.
