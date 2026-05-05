# Ramp Monitoring

Every day, the day's Ramp purchases get checked against your approved-vendor list — anything off-contract, duplicate, or out-of-policy is flagged.

## Prerequisites
- A [Ramp](https://ramp.com) business account where you can authorize the Ramp MCP via OAuth
- A Slack workspace where you can install the agent's bot and invite it to one or more channels (typically `#procurement` or `#finance`)

<table>
  <tr>
    <td><strong>CHANNELS</strong></td>
    <td><code>slack</code> · <code>heartbeat</code> — daily</td>
  </tr>
  <tr>
    <td><strong>CONNECTORS</strong></td>
    <td><code>ramp-mcp</code></td>
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
