  <tr>
    <th><code>agent.connected</code></th>
    <td>An agent has connected to the API</td>
  </tr>
  <tr>
    <th><code>agent.lost</code></th>
    <td>An agent has been marked as lost. This happens when Buildkite stops receiving pings from the agent</td>
  </tr>
  <tr>
    <th><code>agent.disconnected</code></th>
    <td>An agent has disconnected. This happens when the agent shuts down and disconnects from the API</td>
  </tr>
  <tr>
    <th><code>agent.stopping</code></th>
    <td>An agent is stopping. This happens when an agent is instructed to stop from the API. It first transitions to stopping and finishes any current jobs</td>
  </tr>
  <tr>
    <th><code>agent.stopped</code></th>
    <td>An agent has stopped. This happens when an agent is instructed to stop from the API. It can be graceful or forceful</td>
  </tr>
  <tr>
    <th><code>agent.blocked</code></th>
    <td>An agent has been blocked. This happens when an agent's IP address is no longer included in the agent token's <a href="/docs/pipelines/clusters/manage-clusters#restrict-an-agent-tokens-access-by-ip-address">allowed IP addresses</a></td>
  </tr>
