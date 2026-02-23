# tx-scan

Analyze Ethereum transactions — fetches semantic trace data and generates an interactive HTML visualization of internal calls, token flows, and DeFi actions.

## Install

```bash
npx skills add matrooslabs/etheragent
```

Or load directly in Claude Code:

```bash
claude --plugin-dir ./path/to/etheragent
```

## Usage

```
/tx-scan 0xTRANSACTION_HASH
```

Requires `curl` and `jq`. Optionally set `MEVSCAN_API_URL` (defaults to `http://localhost:3001`).

## License

MIT
