# BUILD REPORT — zscorecalc

Clean-room rebuild of `com.edgethirteen.www/altman-z-score-calculator`
v1.0.0 (remote-only MCP server at
`https://www.edgethirteen.com/api/mcp/altman-z-score-calculator`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/zscorecalc` 0.1.0, registry name
`io.github.CloudCorpRecords/zscorecalc`.

## Surface built

Full surface: **1 of 1 captured tools built**, none omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `compute_altman_z_score` | `ticker` |

The served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO
`title` key on the tool and none is served. The tool literal was
generated programmatically from
`harness/fixtures/altmanz-tools.json` by the script at
`/tmp/b50-zscorecalc-scratch/gen.py` (parse + re-serialise; the
generator's round-trip assertions confirmed the emitted literal
parses back to the fixture's tool object) — never retyped — and
the registration-integrity test suite plus the parity harness
confirmed the served literal deep-equals the fixture's tool
object. Tools are registered via the MCP SDK low-level Server
API (`ListToolsRequestSchema` / `CallToolRequestSchema`
handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tool's name,
description, required field (`ticker`) and property shape,
including the absence of a `title` key.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned:
  `https://www.edgethirteen.com/api/mcp/altman-z-score-calculator`.
- Env override `ZSCORECALC_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization`
  header is sent under any environment — including with key-like
  env vars (`ZSCORECALC_API_KEY`, `API_KEY`, `AUTHORIZATION`)
  set — and that no `X-API-Key` header is ever sent; upstream
  requests carry only `Content-Type`, `Accept` and, when issued,
  `Mcp-Session-Id`.
- Error rendering `zscorecalc error:` text for upstream
  failures — labelled reconstruction.
- Upstream results pass through verbatim, including
  `structuredContent`, `isError` and extra fields. tools/call is
  forwarded as streamable-HTTP JSON-RPC: initialize
  (protocolVersion `2025-03-26`) → notifications/initialized →
  tools/call, honouring `Mcp-Session-Id` when issued; plain-JSON
  and SSE replies are both parsed. The forwarding core takes an
  injectable fetch; tests stub the upstream — no live network
  calls in tests (the real upstream is slow and flaky).

## Deviations

Process deviations only (no surface or code deviations):

- The offline lockfile route (Route 3) failed with ENOTCACHED
  for the `@modelcontextprotocol/sdk` packument, so the fallback
  egress-proxy route was used (see Lockfile).
- In-session runs of the fallback were killed by the VM
  write-guard window (EPERM on npm's writes) and one run was
  lost to a VM replacement (boot 19:53 PT, /tmp wiped). The
  successful lockfile generation, full install, typecheck,
  build, test and parity chain ran setsid-detached (the
  documented escape for guard windows), invoking the direct
  `tsc`/`tsx` binaries instead of the `npm`/`npx` wrappers; all
  outputs were verified from disk afterwards.

## Lockfile

- Route: fallback (offline Route 3 failed, ENOTCACHED):
  `HTTPS_PROXY=http://hatch-egress-proxy:3128
  HTTP_PROXY=http://hatch-egress-proxy:3128 npm install
  --package-lock-only --ignore-scripts --registry
  https://registry.npmjs.org`, generated clean from birth in the
  build dir with no `node_modules` and no pre-existing lockfile,
  executed setsid-detached during a write-guard window.
- Parses as JSON; `name` == `@reneturcios/zscorecalc`.
- Package entries: **128**.
- sha256: `385195a37e3d49d1f834ba712352faf976ffad89b0f4e1d9edb004a253917c74`
  (re-verified unchanged after the subsequent full `npm install`).

## server.json

- Description exactly: "Altman Z-Score for a US-listed ticker from SEC EDGAR data: score plus safe, grey or distress zone." —
  **98 characters** (verified programmatically).
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/zscorecalc:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/zscorecalc"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/zscorecalc`
  and its smoke test asserts the server name `zscorecalc`.

## Verification (all run 2026-10-28)

1. `tsc --noEmit` (direct binary, detached chain) → exit 0.
2. `tsc` build → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. Tests (`tsx --test test/forward.test.ts`, fake-fetch only) →
   **53 tests, 53 pass, 0 fail** (registration-integrity suite
   deep-equals the served tools/list against the fixture's tool,
   including required-key exactness and the NO-`title`-key
   assertion, plus session handling/reuse, env override, the
   no-Authorization / no-X-API-Key assertions under any
   environment, verbatim argument forwarding and result
   pass-through including structuredContent and upstream
   isError, SSE parsing, error rendering for both the
   JSON-RPC-error and transport-failure paths, MCP-layer
   validation, and CLI behaviour including a working subcommand
   for the tool against a local stub server).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/altmanz-tools.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/zscorecalc/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/zscorecalc/parity-out` → **verdict PASS, parity 100.0**
   (1/1 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
