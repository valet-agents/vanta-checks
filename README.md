# Vanta Checks

Every day, the day's policy-relevant Vanta events land in Slack with evidence links attached — your compliance lead can answer auditors live.

## Prerequisites
- A [Vanta](https://www.vanta.com) account with an API token
- A Slack workspace where you can install the agent's bot and invite it to a compliance channel
- **Note**: Vanta is not in the Valet catalog — this template requires manual connector setup post-deploy. See [AGENTS.md](./AGENTS.md).

<table>
  <tr>
    <td><strong>CHANNELS</strong></td>
    <td><code>slack</code> · <code>heartbeat</code> — every 24h</td>
  </tr>
  <tr>
    <td><strong>CONNECTORS</strong></td>
    <td><code>vanta-mcp</code> (custom — see <a href="./AGENTS.md">AGENTS.md</a>)</td>
  </tr>
  <tr>
    <td colspan="2" align="center">
      <br />
      <a href="https://valet.dev/deploy?from=github.com/valet-agents/vanta-checks">
        <img src="https://raw.githubusercontent.com/valet-agents/vanta-checks/main/.github/deploy-button.svg" alt="Deploy Agent →" height="40" />
      </a>
      <br /><br />
    </td>
  </tr>
</table>
