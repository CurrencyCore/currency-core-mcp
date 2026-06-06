# CurrencyCore MCP — Claude Code plugin

A Claude Code plugin (and marketplace) that connects Claude to the **CurrencyCore MCP server**
(`https://mcp.currency-core.com/mcp`). Once installed, Claude can call:

- **`convert`** — convert an amount between currencies, optionally PPP-adjusted
- **`get_rates`** / **`get_rates_by_base`** — full exchange-rate snapshots
- **`list_countries`** — supported countries + official currencies
- **`get_ppp`** — purchasing-power-parity factors

…plus the API docs (as readable resources) and guided prompts.

The server uses **OAuth** — the first time you use it, Claude opens your browser to sign in
to CurrencyCore and pick an organization. Requests count against that org's usage/limits.

## Install

```bash
# 1. Register this marketplace (once)
claude plugin marketplace add logn10/currency-core-mcp

# 2. Install the plugin
claude plugin install currency-core-mcp@currencycore
```

Then in a Claude Code session, run `/mcp`, select **currency-core**, and **Authenticate**.

## Verify

```bash
claude mcp list          # currency-core listed
```
In-session: `/mcp` shows the server connected with its tools, resources, and prompts.

## Repo layout

```
.
├── .claude-plugin/
│   └── marketplace.json          # marketplace manifest (lists the plugin)
└── currency-core-mcp/            # the plugin
    └── .claude-plugin/
        └── plugin.json           # plugin manifest — the MCP connector is the
                                  #   inline "mcpServers" block (remote HTTP server)
```

## Local testing (before publishing)

```bash
# Validate the marketplace + plugin without pushing
claude plugin marketplace add /absolute/path/to/currency-core-mcp
claude plugin install currency-core-mcp@currencycore
# or load the plugin directly:
claude --plugin-dir ./currency-core-mcp
```

## Prefer not to use a plugin?

A single user can skip all of this and just add the server directly:

```bash
claude mcp add --transport http currency-core https://mcp.currency-core.com/mcp
```

The plugin/marketplace route is for **distributing** the connector to a team or community
with one install command + versioned updates.
