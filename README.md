# Ramp Monitoring

Every Ramp purchase gets checked against your approved-vendor list — anything that looks off is flagged in #procurement before it's reconciled.

## Prerequisites
- A [Ramp](https://ramp.com) business account with API access (client ID + secret) — or a Zapier account connected to Ramp
- A Slack workspace where you can install the agent's bot and invite it to `#procurement` (or wherever you want anomalies to land)
- **Note**: Ramp is not in the Valet catalog — this template requires manual connector setup post-deploy. See [AGENTS.md](./AGENTS.md).

<table>
  <tr>
    <td><strong>CHANNELS</strong></td>
    <td><code>slack</code> · <code>heartbeat</code> — every 5 min</td>
  </tr>
  <tr>
    <td><strong>CONNECTORS</strong></td>
    <td><code>ramp-mcp</code> (custom — see <a href="./AGENTS.md">AGENTS.md</a>)</td>
  </tr>
  <tr>
    <td colspan="2" align="center">
      <br />
      <a href="https://valet.dev/deploy?from=github.com/valet-agents/ramp-monitoring">
        <img src="https://raw.githubusercontent.com/valet-agents/ramp-monitoring/main/.github/deploy-button.svg" alt="Deploy Agent →" height="40" />
      </a>
      <br /><br />
    </td>
  </tr>
</table>
