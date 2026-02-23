# Brontes Tree JSON Reference

Each JSON object = one **transaction** with its metadata and a reconstructed **call trace tree** mirroring the EVM call stack. Nodes are classified into normalized actions (Swap, Transfer, Mint, etc.) when possible — protocol-agnostic (Uniswap V3 swap and Curve swap share the same `Swap` shape).

## Top-Level Transaction Object

- `block_number` (string): Block number
- `tx_hash` (string): `0x`-prefixed transaction hash
- `tx_idx` (string): Transaction position in block (0-indexed)
- `from` (string): Sender address (EOA)
- `to` (string|null): Recipient address. Null for contract creation
- `gas_details` (object): `{ coinbase_transfer: int|null, priority_fee: int, gas_used: int, effective_gas_price: int }` — all in wei
- `timeboosted` (boolean): Arbitrum Timeboost flag
- `trace_tree` (array): Root-level trace nodes (typically one)

## Trace Node

Each node = one EVM call frame.

- `trace_idx` (string): Sequential execution index (depth-first order)
- `trace_address` (string[]): Hierarchical position — `[]`=root, `["0"]`=first sub-call, `["0","1"]`=second sub-call of first sub-call
- `action_kind` (string|null): Normalized action type (see below). Null = no classification
- `action` (object|null): Action data (shape depends on `action_kind`). Null when `action_kind` is null
- `children` (array): Nested child trace nodes

## Action Kinds

`Swap` | `SwapWithFee` | `Transfer` | `EthTransfer` | `Mint` | `Burn` | `Collect` | `FlashLoan` | `Liquidation` | `Batch` | `Aggregator` | `NewPool` | `PoolConfigUpdate` | `SelfDestruct` | `Unclassified` | `Revert`

## Action Schemas

### Swap
`{ protocol, trace_index, from, recipient, pool, token_in: TokenInfo, token_out: TokenInfo, amount_in: Rational, amount_out: Rational, msg_value }`

### SwapWithFee
`{ swap: Swap, fee_token: TokenInfo, fee_amount: Rational }`

### Transfer
`{ trace_index, from, to, token: TokenInfo, amount: Rational, fee: Rational, msg_value }`

### EthTransfer
`{ trace_index, from, to, value: hex_wei, coinbase_transfer: bool }`

### Mint / Burn / Collect (shared schema)
`{ protocol, trace_index, from, recipient, pool, token: TokenInfo[], amount: Rational[] }` — `token` and `amount` arrays are parallel

### FlashLoan
`{ protocol, trace_index, from, pool, receiver_contract, assets: TokenInfo[], amounts: Rational[], aave_mode: object|null, child_actions: Action[], repayments: Transfer[], fees_paid: Rational[], msg_value }`

### Liquidation
`{ protocol, trace_index, pool, liquidator, debtor, collateral_asset: TokenInfo, debt_asset: TokenInfo, covered_debt: Rational, liquidated_collateral: Rational, msg_value }`

### Batch
`{ protocol, trace_index, solver, settlement_contract, user_swaps: Swap[], solver_swaps: Swap[]|null, msg_value }`

### Aggregator
`{ protocol, trace_index, from, to, recipient, child_actions: Action[], msg_value }`

### Unclassified
`{ trace: { type, action: { from, to, callType, gas, input, value }, result: { gasUsed, output }, subtraces, traceAddress }, logs: [], msg_sender, trace_idx, decoded_data: object|null }` — raw parity-style trace

### NewPool / PoolConfigUpdate
`{ trace_index, protocol, pool_address, tokens: string[] }`

## Common Types

**TokenInfo**: `{ address, symbol, decimals }`

**Rational** (exact, decimal-adjusted): `{ s: bool (true=positive), n: hex_string (numerator), d: hex_string (denominator) }` — human amount = `n / d`. Values are already divided by `10^decimals`.
