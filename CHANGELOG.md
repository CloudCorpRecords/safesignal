# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted safe-check MCP server. All 3 tools of the captured surface are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `recall_lookup`, `counterfeit_risk`, `safe_check`. The capture carries no `title` keys, so none are served; every tool requires `query`.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `safesignal error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `SAFECHECK_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
