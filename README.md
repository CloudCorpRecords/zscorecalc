# zscorecalc

How close is a public company to financial distress? zscorecalc is a local forwarding client for the hosted Altman Z-Score calculator MCP service: it answers `tools/list` locally from the captured tool definition and forwards every call to the service, returning the answers exactly as the service sends them back.

The one tool computes the Altman Z-Score (public-company, non-financial-services version) for a US-listed ticker — a quick bankruptcy-risk / financial-distress screen from five balance-sheet ratios. The service looks up working capital, total assets, retained earnings, EBIT, market cap, total liabilities, and revenue from SEC EDGAR filings and market data automatically; you just pass a ticker. It returns the score and which zone it falls in: safe (>2.99), grey (1.81–2.99), or distress (<1.81). There is no write, signup, or send tool in this surface.

## The tool

- **`compute_altman_z_score`** — The Altman Z-Score for a US-listed `ticker` (for example `GE`), with its safe / grey / distress zone. `ticker` is required.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/zscorecalc mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "zscorecalc": {
      "command": "npx",
      "args": ["-y", "@reneturcios/zscorecalc", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/zscorecalc
docker run --rm ghcr.io/cloudcorprecords/zscorecalc compute_altman_z_score '{"ticker":"GE"}'
```

## CLI

One binary serves both the command line and the MCP server. The tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
zscorecalc compute_altman_z_score '{"ticker":"GE"}'
zscorecalc --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `ZSCORECALC_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://www.edgethirteen.com/api/mcp/altman-z-score-calculator`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `zscorecalc error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (the 1 tool, deep-equal, with no `title` key served — the capture carries none — and the `required` list exactly as captured), exercises the session handshake and its reuse, verifies that no `Authorization` or `X-API-Key` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
