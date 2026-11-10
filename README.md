# CurrencyCore MCP

Connect any MCP-capable AI client to the **CurrencyCore API** — live FX rates,
any-pair currency conversion, PPP (purchasing-power-parity) adjustment,
historical trend and risk analysis, and country/currency reference data — through
the hosted MCP server at **`https://mcp.currency-core.com/mcp`**.

This repo is both a **Claude Code plugin + marketplace** and a set of manifests
for other clients/registries, all pointing at the same remote server.

- **Website:** https://currency-core.com
- **Documentation:** https://currency-core.com/docs/mcp
- **Server:** `https://mcp.currency-core.com/mcp`
- **Version:** 1.0.0

## What you get

**Public reference tools — no authentication**
- `list_countries` — supported countries + their official currencies
- `list_currencies` — supported ISO 4217 codes, names, and symbols

**Core tools — authorized connection, all plans**
- `convert` — convert an amount between currencies, optionally PPP-adjusted (`from=USD:USA&to=INR:IND` style for PPP)
- `get_rates` — full USD-base exchange-rate snapshot for a date
- `get_rates_by_base` — the snapshot re-expressed against any base currency
- `get_history` — historical rate time series (Free: last 30 days; paid: full history)
- `get_ppp_factor` — a single country's IMF PPP conversion factor for a year

**Analytics tools — Growth plan or higher**
- `analyze_history` — trends, comparisons, movers, change %, volatility, high/low, trend series
- `analyze_ppp` — IMF PPP factor change and trend over time
- `analyze_volatility` — rank or compare currencies by annualized volatility
- `analyze_correlation` — how closely currencies move together
- `analyze_drawdown` — peak-to-trough decline and recovery speed
- `analyze_safe_haven` — safe-haven scoring and ranking
- `analyze_mean_reversion` — how strongly currencies revert to their historical mean

**Resources** — the CurrencyCore API docs plus the OpenAPI 3.1 spec, readable in-client.
**Prompts** — guided workflows (convert-with-PPP, compare cost-of-living, build a rate table, explain an error).

The server uses **OAuth**: the first time you connect, your client opens a
browser to sign in to CurrencyCore and pick an organization. Calls then count
against that org's plan + usage (and you can disable MCP per-org in the
dashboard any time).

---

## Install in your client

### Claude Code
```bash
claude plugin marketplace add CurrencyCore/currency-core-mcp
claude plugin install currency-core-mcp@currencycore
# in a session: /mcp → currency-core → Authenticate
```

### Oh My Pi (omp)
omp is Claude-Code-marketplace-compatible, so the same plugin installs straight from this repo:
```bash
omp plugin marketplace add CurrencyCore/currency-core-mcp
omp plugin install currency-core-mcp@currencycore
# first use opens a browser sign-in (or run: /mcp reauth currency-core)
```
…or add to `.omp/mcp.json` (project) / `~/.omp/agent/mcp.json` (user) — omp needs `"type": "http"`:
```json
{
  "mcpServers": {
    "currency-core": { "type": "http", "url": "https://mcp.currency-core.com/mcp" }
  }
}
```
> Same agent, two names: the `pi` lineage and the `omp` CLI share this config (`PI_PROFILE` / `OMP_PROFILE`); there is no separate `.pi` path.

### Cursor
One-click: [**Add to Cursor**](https://cursor.com/install-mcp?name=currency-core&config=eyJ1cmwiOiJodHRwczovL21jcC5jdXJyZW5jeS1jb3JlLmNvbS9tY3AifQ==)

…or add to `.cursor/mcp.json` (project) / Cursor Settings → MCP:
```json
{
  "mcpServers": {
    "currency-core": { "url": "https://mcp.currency-core.com/mcp" }
  }
}
```

### VS Code (Copilot / MCP)
Add to `.vscode/mcp.json` (or user `settings.json` under `"mcp"`):
```json
{
  "servers": {
    "currency-core": { "type": "http", "url": "https://mcp.currency-core.com/mcp" }
  }
}
```

### Gemini CLI
As an extension (reads `gemini-extension.json` from this repo):
```bash
gemini extensions install https://github.com/CurrencyCore/currency-core-mcp
```
…or add to `~/.gemini/settings.json` (Gemini uses `httpUrl` for streamable HTTP):
```json
{
  "mcpServers": {
    "currency-core": { "httpUrl": "https://mcp.currency-core.com/mcp" }
  }
}
```

### GitHub Copilot CLI
Register the server in Copilot CLI's MCP configuration with the URL
`https://mcp.currency-core.com/mcp` (HTTP transport). Check the current Copilot
CLI MCP docs for the exact config path — the server itself is fully compatible.

### Any other MCP client (Claude Desktop, Cline, Continue, Zed, …)
It's a standard **remote, streamable-HTTP** MCP server with OAuth. Point your
client's MCP config at `https://mcp.currency-core.com/mcp` and complete the
browser sign-in on first use.

> ℹ️ One-click deeplink schemes (Cursor / VS Code) and per-client config keys
> change occasionally — if a deeplink misbehaves, use the manual JSON snippet,
> which is stable.

---

## Publish / list on registries

- **Official MCP Registry** (`registry.modelcontextprotocol.io`) — remote-only server, publish [`server.json`](./server.json) with the official `mcp-publisher` CLI:
  ```bash
  # Install (macOS): brew install mcp-publisher
  # Or download the release binary from github.com/modelcontextprotocol/registry/releases
  mcp-publisher validate
  mcp-publisher login github
  mcp-publisher publish
  ```
  GitHub auth verifies the `io.github.currencycore` namespace; no npm package is required for a remote-only server.
- **Smithery** (`smithery.ai`) — submit the remote server via the Smithery site (their repo-based config is for servers Smithery builds/hosts; a hosted remote server is added through their UI).
- **Directories** — list on **Glama** (`glama.ai/mcp`), **mcp.so**, **PulseMCP**, and open a PR to **`awesome-mcp-servers`**.
- **Claude.ai / Claude Desktop connectors** — submit the remote server to Anthropic's connectors directory (this is exactly the remote-OAuth shape it expects).

---

## Usage tips

- **PPP conversions need a country on every currency**: pass `from=USD:USA` and `to=INR:IND` (a currency alone can't imply a country). `amount` defaults to `1`, so omit it to get the raw rate.
- **Dates are UTC** and can't be in the future; omit `date` for the latest snapshot.
- **Pick the right org at connect time** — the connection is bound to one organization; reconnect to switch. Usage/limits/billing apply to that org exactly like a normal API key.
- **Turn it off any time** — disable MCP for an org from the CurrencyCore dashboard (**Developer → *MCP server***) to stop MCP traffic without revoking anything.
- **Ask in natural language** — e.g. *"Convert 250 USD to EUR and INR, PPP-adjusted for Germany and India"* or *"Which is cheaper for the same basket, USA or India?"* — the model will pick the right tools.

---

## Repo layout

```
.
├── .claude-plugin/
│   └── marketplace.json          # Claude Code marketplace (lists the plugin)
├── currency-core-mcp/
│   └── .claude-plugin/
│       └── plugin.json           # Claude Code plugin (inline mcpServers, type: http)
├── server.json                   # Official MCP Registry manifest
├── gemini-extension.json         # Gemini CLI extension manifest
└── README.md
```

## Updating

Bump `version` in `currency-core-mcp/.claude-plugin/plugin.json` (+ the
marketplace entry, `server.json`, `gemini-extension.json`), commit, and push to
`main`. Installed Claude Code / Gemini users pick up the update; re-publish
`server.json` to the registry for a new registry version.

---

Self-hosting or the API itself? See **https://currency-core.com** and the API
docs. The MCP server is a thin proxy over the public API — every call respects
the same auth, rate limits, and billing.

## License

MIT — see [LICENSE](./LICENSE).
