---
name: trace-tx
description: Analyze Ethereum transactions and explain them as plain text with ASCII art directly in the terminal. Use this skill whenever a user wants a quick text-based trace of a transaction without opening a browser. Trigger on phrases like "trace this tx", "what did this tx do", "break down this transaction", "summarize this tx", or when a user provides a 0x-prefixed 66-character transaction hash and asks for a text explanation. Also use when the user says "trace-tx" explicitly. Prefer this over visual-tx when the user asks for plain text, terminal output, or ASCII output.
allowed-tools: Bash(curl *)
---

# Transaction Explainer (Plain Text)

Fetch semantic trace data for an Ethereum transaction and explain it directly in the terminal as structured plain text with ASCII art. No HTML, no Mermaid, no browser — just clear, readable output.

## Workflow

### 1. Fetch the trace tree

Extract the transaction hash from `$ARGUMENTS`. Fetch the trace data:

```bash
curl -sf "https://mevscan.matroos.xyz/api/tree/<tx_hash>"
```

If the request fails, tell the user and stop.

Read `./references/tree-json.md` to understand the JSON schema.

### 2. Parse the JSON

Extract these from the response:

**Header info:**
- `tx_hash`, `block_number`, `tx_idx`
- `from` and `to` addresses
- Gas: `gas_used`, `effective_gas_price` (convert to Gwei), total cost in ETH
- `timeboosted` flag

**Actions:**
Recursively walk `trace_tree` -> `children`. For each node with a non-null `action_kind`, extract the action type, key fields (addresses, tokens, amounts), and depth from `trace_address`.

Token amounts are already human-readable decimals — use them directly.

### 3. Render as plain text

Output everything directly as text in your response. Use the format below.

#### Transaction Header

```
=============================================================
  TX: 0xabcd...ef12
  Block: 19,234,567  |  Index: 42
  From:  0xd8dA...6045
  To:    0x68b3...Fc45
  Gas:   245,312 used  |  32.4 Gwei  |  0.00795 ETH
  Status: Timeboosted
=============================================================
```

Show full tx hash on the TX line (no truncation). Shorten From/To addresses to `0xXXXX...XXXX` (first 4 + last 4 hex chars after 0x).

#### Action Summary

```
  Actions: 8 total
  |- Swaps:     3
  |- Transfers: 2
  |- ETH Sends: 2
  |- Mints:     1
```

Only show action types that actually appear.

#### Call Trace Tree

This is the core of the explanation. Render the call tree with ASCII box-drawing characters to show the hierarchy:

```
  CALL TRACE
  ==========

  [0] Aggregator  Aggregate via 1inch (0x1111...1111)
   |
   +--[1] Swap  1.500 ETH -> 3,012.45 USDC  (Uniswap V3)
   |   |
   |   +--[2] Transfer  3,012.45 USDC -> 0x68b3...Fc45
   |
   +--[3] Swap  3,012.45 USDC -> 0.0842 WBTC  (Curve)
   |
   +--[4] ETH Send  0.005 ETH -> 0xBuil...lder  (priority fee)
   |
   +--[5] Mint  0.842 WBTC + 1,200 USDC into Uniswap V3 pool
   |   |
   |   +--[6] Swap  0.100 WBTC -> 4,250.00 DAI  (Balancer V2)
   |
   +--[7] ETH Send  0.002 ETH -> 0xCoin...base  (coinbase transfer)
```

Rules for the tree:
- `[N]` is the `trace_idx`
- Action kind comes next as a label (Swap, Transfer, ETH Send, Mint, Burn, etc.)
- Then a one-line summary of what happened (amounts, tokens, protocol)
- Use `+--` for child branches and `|` for continuing trunk lines
- Indent children under their parent
- For **Swap**: `{amount_in} {token_in} -> {amount_out} {token_out}  ({protocol})`
- For **Transfer**: `{amount} {token} -> {to_address}`
- For **EthTransfer**: `{value} ETH -> {to_address}  ({note if coinbase})`
- For **Mint/Burn**: `{amounts joined by " + "} {tokens} into/from {pool}  ({protocol})`
- For **Liquidation**: `Liquidate {debtor}: repay {debt_amount} {debt_token} for {collateral_amount} {collateral_token}`
- For **FlashLoan**: `FlashLoan {amounts} via {protocol}`
- For **Batch**: `Batch settlement via {solver}`
- For **Aggregator**: `Aggregate via {protocol} ({address})`
- For **Revert**: `REVERTED call {from} -> {to}` (mark clearly)
- For **Unclassified**: `Call {from} -> {to}`

#### Token Flow Table

After the trace tree, show a table of all token movements:

```
  TOKEN FLOWS
  ===========
  #  | Action     | From           | To             | Token      | Amount
  ---+------------+----------------+----------------+------------+------------------
  1  | Swap       | 0xd8dA...6045  | 0x88e6...5640  | ETH->USDC  | 1.500 -> 3,012.45
  2  | Transfer   | 0x88e6...5640  | 0x68b3...Fc45  | USDC       | 3,012.45
  3  | Swap       | 0x68b3...Fc45  | 0xD51a...AE46  | USDC->WBTC | 3,012.45 -> 0.0842
  4  | ETH Send   | 0xd8dA...6045  | 0xBuil...lder  | ETH        | 0.005
  5  | Mint       | 0x68b3...Fc45  | 0xUniV...Pool  | WBTC+USDC  | 0.0842 + 1,200
```

Use simple ASCII table formatting with `|` separators and `-`+`+` for the header line.

#### Plain English Summary

End with a brief 2-4 sentence narrative explaining what the transaction did in plain English. This is the "so what" — it helps someone who doesn't want to read the trace tree understand the gist.

Example:
```
  SUMMARY
  =======
  This transaction used the 1inch aggregator to swap 1.5 ETH into
  USDC via Uniswap V3, then routed the USDC through Curve to acquire
  WBTC. The WBTC and remaining USDC were minted into a Uniswap V3
  liquidity position. Builder and coinbase tips totaled 0.007 ETH.
```

Write this summary yourself based on the actions you see — it should read naturally and highlight the key economic activity (what was swapped, what was the end goal, any notable patterns like arbitrage, liquidation, sandwich, etc.).

### 4. No file output

Do not write any files. Do not open a browser. Output everything as plain text directly in your response to the user.

## Quality Checks

Before responding:
- All amounts use the human-readable decimals from the API (no wei conversion needed)
- Addresses are shortened consistently to `0xXXXX...XXXX`
- The trace tree correctly reflects parent-child relationships from `trace_address`
- The summary accurately describes the transaction's economic activity
- Reverted calls are clearly marked
