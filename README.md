# WorldMonitor for Hermes

Connect [Hermes Agent](https://hermes-agent.nousresearch.com/) to
[WorldMonitor](https://worldmonitor.app) for source-aware global intelligence:
country risk, conflict events, market context, maritime chokepoints, energy
and natural hazards.

This is a configuration-only Agent Plugins v1 package. It ships a remote MCP
connection and one workflow skill. There is no local runtime code, Python
plugin, shell hook, background process, binary or self-updater.

## Endpoint and access

- Product MCP: **https://worldmonitor.app/mcp**, Streamable HTTP.
- Official MCP Registry name:
  [`app.worldmonitor/mcp`](https://registry.modelcontextprotocol.io/v0.1/servers/app.worldmonitor%2Fmcp/versions/1.50.0).
- Discovery is public. `get_sources` is the sole credential-free data tool.
- Live intelligence requires an authenticated account. The server currently
  offers signed-in free accounts a small cached-data allowance; subscription
  tools and larger allowances need an eligible plan. Read each tool's live
  `worldmonitor/access` metadata and the current allowance, rather than
  assuming everything is free or that every signed-in account has full access.
- Authentication options: OAuth 2.1 with PKCE, or a dashboard-issued API key
  sent as `X-WorldMonitor-Key`. Keys never belong in this repository.
- [Product MCP documentation](https://worldmonitor.app/mcp) is authoritative
  for current access rules. Its prose may lag the live schemas: recheck both.

## Install

Target: **Hermes Agent v0.23.0 or newer** with its MCP support installed.
The package has passed JSON-schema checks, but runtime validation remains
blocked: the official installer could not download its pinned Python/FFmpeg
artifacts because of TLS errors on the test box. Do not interpret the commands
below as recorded successes.

After the repository is approved, created and published:

```bash
hermes plugins install koala73/worldmonitor-hermes-plugin --no-enable
hermes plugins enable worldmonitor
hermes plugins list
```

The proposed repository URL is not published by this preparation task.
Once admitted to the catalog, installation will also be available as
`hermes plugins install worldmonitor` at the exact reviewed commit.

The loadable skill is `skills/worldmonitor/SKILL.md`, exposed by Hermes as
`worldmonitor:worldmonitor`. Use `skills_list` to discover it, then
`skill_view("worldmonitor:worldmonitor")`. Root `SKILL.md` is an identical
convenience copy; Hermes discovers the nested copy, not the root file.

## Authenticate in Hermes

Portable `mcp.json` supports `type`, `url` and `headers` for remote servers,
not the native `auth`/`oauth` keys. The included default has no credentials
and is intended for public discovery and the public source-inventory test.
It does not unlock gated intelligence. If initialize itself returns 401,
configure OAuth before reconnecting.

### Recommended: OAuth

In the active Hermes profile's MCP configuration, add this same-name native
server entry. Hermes documents that a native entry replaces the package's
entry, including its trust policy, so keep `trust: untrusted` explicitly:

```yaml
mcp_servers:
  worldmonitor:
    url: https://worldmonitor.app/mcp
    auth: oauth
    trust: untrusted
```

Then, from the user's own interactive terminal:

```bash
hermes mcp login worldmonitor
hermes mcp test worldmonitor
```

Hermes handles the browser authorization, token storage and refresh through
its own profile-scoped MCP credential mechanism. This package does not read
or manage that store. No OAuth login or new sign-up was performed during
preparation. WorldMonitor advertises scope `mcp`; interoperability of the
Hermes flow still needs an authorized interactive test.

### Alternative: API key

Use Hermes's profile-scoped secret source to supply `WORLDMONITOR_API_KEY`.
Do not put the actual value in this package or a shared config. A native
configuration can reference the secret:

```yaml
mcp_servers:
  worldmonitor:
    url: https://worldmonitor.app/mcp
    trust: untrusted
    headers:
      X-WorldMonitor-Key: "${env:WORLDMONITOR_API_KEY}"
```

This follows Hermes's documented SecretRef expansion. A missing variable
fails closed. Obtain or rotate a key only through the user's approved
WorldMonitor account flow. This alternative has not been runtime-tested here.

[Hermes MCP config and authentication reference](https://hermes-agent.nousresearch.com/docs/reference/mcp-config-reference)

## Use

- Ask for a source inventory with `get_sources({"view":"summary"})`.
- With authorized account access, ask for structured country risk, a sourced
  country brief, a bounded conflict feed or maritime chokepoint context.
- Inspect the live tool schema instead of guessing arguments. Hermes normally
  exposes a tool as `mcp__worldmonitor__get_sources` (server/tool namespaced).
- Preserve direct source URLs, observation dates, availability and attribution.
  Cached data can be stale; AI briefs and forecasts are assessments, not
  guaranteed facts. This integration is not financial or safety-critical advice.

## Security and network disclosure

- Enabling the MCP connection sends connection/discovery traffic and tool
  arguments to `https://worldmonitor.app/mcp`; it is a hosted external service.
- An OAuth setup also contacts the service's advertised authorization server.
  With API-key configuration, Hermes attaches that key only to the configured
  endpoint. Never commit credentials or paste them into tool arguments.
- This plugin ships no executable runtime, dependency installer, shell commands
  to execute automatically, local data reader, telemetry collector or updater.
  The setup commands above are manual user instructions, not hooks.
- The hosted service can observe requests and is governed by its own policies;
  this package does not make claims about server-side retention or analytics.
- Tool calls may spend the signed-in account's data allowance. Authentication
  and subscription limits are enforced by WorldMonitor, not bypassed here.
- Remote tools and their annotations are served dynamically and are not frozen
  by the plugin repository's SHA. Hermes `trust: untrusted` requests approval
  for calls not annotated read-only under normal approval settings.
- Not every remote action is read-only: `report_issue` submits an external
  report, and `start_upgrade` creates a personal checkout link. The bundled
  skill requires explicit user approval before these actions. This package
  does not itself buy anything or automatically submit reports.
- No files outside the plugin are read by plugin code. Hermes owns its own
  settings, logs and credentials; this package does not inspect those files.

## Validation status

Preparation date: November 6, 2026 (GST).

- The official registry and live `initialize` response identify MCP v1.50.0.
- Anonymous `tools/list` returned 102 tools. That is an observed snapshot,
  not a promise of a fixed future count.
- A direct, no-key `get_sources` call succeeded. This was **not through Hermes**.
- The official shell installer was read before execution and targeted v0.23.0
  in an isolated home, with account setup, gateway, browser and computer-use
  installation skipped. It cloned the release and bootstrapped uv/Python, but
  its package-manager stage failed with `[SSL: WRONG_VERSION_NUMBER]` on
  pinned GitHub Python/FFmpeg downloads. Supported stage retries also failed.
- A read-only package-manager doctor confirmed an incomplete installation.
  No TLS/hash check or distribution guard was bypassed. The earlier pip
  attempt was also rejected by Hermes's explicit wheel/sdist build guard.
- Therefore Hermes plugin installation, skill loading, admission validation /
  security scan and a Hermes-mediated MCP call are **not yet verified**.
  Complete them on a working supported installation before submitting.
  Authentication has not been tested; no account login or key was used.

## Development

On a supported Hermes installation:

```bash
hermes plugins validate /absolute/path/to/worldmonitor-hermes-plugin --install-deps
hermes plugins doctor /absolute/path/to/worldmonitor-hermes-plugin --ci
```

Keep changes configuration-only. Revisions reach catalog users through a new
reviewed full-SHA catalog pin, never through self-updating files. Do not infer
runtime validation from JSON-schema success.

## License

MIT. The hosted service and its upstream data retain their own terms.
