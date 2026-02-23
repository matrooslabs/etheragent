# EtherAgent

Claude Code plugin marketplace for Ethereum and EVM chain tooling.

## Install

```bash
# Add the marketplace
/plugin marketplace add matrooslabs/etheragent

# Install a plugin
/plugin install tx-scan@etheragent
```

## Plugins

### tx-scan

Analyze Ethereum transactions — fetches semantic trace data and generates an interactive HTML visualization of internal calls, token flows, and DeFi actions.

```
/tx-scan 0xTRANSACTION_HASH
```

Requires `curl` and `jq`. Optionally set `MEVSCAN_API_URL` (defaults to `http://localhost:3001`).

## License

MIT
