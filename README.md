# safesignal

Is that stroller under an active recall, and is that bargain collectible listing too good to be true? safesignal is a local forwarding client for the hosted safe-check MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

The three tools all work on products and listings, never on people. `recall_lookup` searches live US government recall feeds (a CPSC/FDA/NHTSA-style aggregate) and returns matching published recall records with the official agency source link and last-updated date. `counterfeit_risk` assesses counterfeit risk for a collectible listing — in the captured framing, a risk signal, never proof of fake — combining marketplace base-risk with a price-versus-retail-median check when the item sits in a known collectible niche, and returns a risk level (low/med/high), a score, the signals, and authenticity advice. `safe_check` runs both reads in a single call.

## The tools

- **`recall_lookup`** — Active US product recalls for a product or brand. `query` (product name or brand, e.g. "Peloton bike") is required.
- **`counterfeit_risk`** — Counterfeit risk signal for a collectible listing. `query` (product or brand, e.g. "Funko Batman" or "LEGO 10307") is required; `price`, the listed price to compare against the retail median, and `marketplace`, where it is listed (e.g. "aliexpress", "ebay", "etsy"), are optional.
- **`safe_check`** — Both a recall lookup and a counterfeit-risk check in one call, the best general-purpose product-safety screen. `query` is required; `price` and `marketplace` are optional.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/safesignal mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "safesignal": {
      "command": "npx",
      "args": ["-y", "@reneturcios/safesignal", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/safesignal
docker run --rm ghcr.io/cloudcorprecords/safesignal recall_lookup '{"query":"stroller"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
safesignal recall_lookup '{"query":"Peloton bike"}'
safesignal counterfeit_risk '{"query":"Funko Batman","price":12,"marketplace":"ebay"}'
safesignal safe_check '{"query":"LEGO 10307"}'
safesignal --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `SAFECHECK_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://safecheck.dropwatchhq.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `safesignal error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (the 3 tools, deep-equal, in order, with no titles invented and `query` required on all three), exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
