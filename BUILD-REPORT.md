# BUILD REPORT — safesignal

Clean-room rebuild of `com.dropwatchhq/safe-check`
v1.0.0 (remote-only MCP server at
`https://safecheck.dropwatchhq.com/mcp`), built 2026-10-28
as a forwarding client. Package: `@reneturcios/safesignal`
0.1.0, registry name `io.github.CloudCorpRecords/safesignal`.

## Surface built

Full captured surface: **3 of 3 captured tools built**,
none omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `recall_lookup` | `query` |
| `counterfeit_risk` | `query` |
| `safe_check` | `query` |

Each served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO `title`
keys, so none are served. The tool literals were generated
programmatically from
`harness/fixtures/safecheck-tools-original.json` by the script
at `/tmp/b48-safesignal-scratch/gen.py` (parse + re-serialise; the
emitted `TOOL_DEFS` block was verified to parse back to the
fixture's tool objects, and the generator's own round-trip
assertion confirmed the literals equal the fixture's tool
objects) — never retyped — and the registration-integrity test
suite plus the parity harness confirmed the served literals
deep-equal the fixture's 3 tool objects, in order. Tools are
registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema` handlers),
not McpServer+zod.

## Re-triage flag (RE-TRIAGE before publish wave)

- **FLAGGED FOR RE-TRIAGE:** `counterfeit_risk` returns a
  product risk score for a collectible product/listing,
  kept on the awardsignal distinction (Batch 46 gap 392;
  the same distinction Batch 47 applied to recallpulse's
  0–100 product risk score at gap 402). The subject is a
  product/listing, never a person, provider or
  counterparty being onboarded; no exclusion list is
  checked and nothing is gated, and the captured framing
  is "a risk SIGNAL, never proof of fake". A reader who
  weighs the distinction differently should re-triage
  safesignal before its publish wave.

## Fixture conflicts

**None.** The spec and the fixture agree on all 3 tools'
names, descriptions, required fields (every tool requires
exactly `query`) and property shapes. The spec's note that
the capture carries no `title` keys matches the fixture
exactly.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://safecheck.dropwatchhq.com/mcp`.
- Env override `SAFECHECK_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`SAFECHECK_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and
  that no `X-API-Key` header is ever sent; upstream requests carry
  only `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `safesignal error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP
  JSON-RPC: initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; plain-JSON and SSE replies are
  both parsed. The forwarding core takes an injectable fetch;
  tests stub the upstream — no live network calls in tests (the
  one CLI forwarding test points the CLI at a stub HTTP server
  on 127.0.0.1 via the env override).

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/safesignal`.
- Package entries: **128**.
- sha256: `6afdffad984a6c640d0d6efda0aeef5a003a3d43750cc8e2d5799915ab7bfaac`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Product recall lookup vs US government feeds plus a counterfeit risk signal for listings." —
  **89 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/safesignal:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/safesignal"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/safesignal`
  and its smoke test asserts the server name `safesignal`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **54 tests, 54 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 3
   tools, including the no-title assertions and the all-three-require-
   `query` checks, session handling/reuse, env override, the
   no-Authorization-header assertions under any environment, verbatim
   argument forwarding and result pass-through including upstream
   isError, SSE parsing, error rendering, MCP-layer validation, and
   CLI behaviour including a tool subcommand forwarding to a local
   stub upstream).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/safecheck-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/safesignal/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/safesignal/parity-out` → **verdict PASS, parity 100.0**
   (3/3 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
