# CoW Swap (CoW Protocol)

Intent-based DEX aggregator. Users sign off-chain orders, solvers find optimal execution (CoWs, AMMs, private MMs). MEV-protected, gasless for users.

## API

Base URL: `https://api.cow.fi/mainnet/api/v1`

### Get a Quote

```bash
curl -X POST https://api.cow.fi/mainnet/api/v1/quote \
  -H "Content-Type: application/json" \
  -d '{
    "from": "<wallet>",
    "sellToken": "<address>",
    "buyToken": "<address>",
    "sellAmountBeforeFee": "<amount_wei>",
    "kind": "sell",
    "priceQuality": "verified"
  }'
```

For buy orders, use `"kind": "buy"` and `"buyAmountAfterFee"` instead of `sellAmountBeforeFee`.

**Response fields:**
- `quote.sellAmount` — amount to sell (after fee)
- `quote.buyAmount` — amount received
- `quote.feeAmount` — protocol fee
- `id` — quote ID
- `verified` — price verification status

### Place a Limit Order (EOA)

For EOA wallets, sign the order with EIP-712 and submit:

```bash
# 1. Approve sell token to GPv2VaultRelayer
cast send <sellToken> "approve(address,uint256)" \
  0xC92E8bdf79f0507f65a392b0ab4667716BFE0110 <sellAmount> \
  --private-key $KEY --rpc-url $RPC

# 2. Sign the order (EIP-712) and POST to API
curl -X POST https://api.cow.fi/mainnet/api/v1/orders \
  -H "Content-Type: application/json" \
  -d '{
    "sellToken": "<address>",
    "buyToken": "<address>",
    "sellAmount": "<amount>",
    "buyAmount": "<min_amount>",
    "validTo": <unix_timestamp>,
    "appData": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "feeAmount": "0",
    "kind": "sell",
    "partiallyFillable": false,
    "receiver": "<wallet>",
    "from": "<wallet>",
    "sellTokenBalance": "erc20",
    "buyTokenBalance": "erc20",
    "signingScheme": "eip712",
    "signature": "<eip712_signature>"
  }'
```

### Place a Limit Order (Smart Contract / Safe)

Smart contracts use `presign` signing scheme — sign on-chain via the settlement contract:

```python
# Key fields for presign orders:
order = {
    "sellToken": sell_token,
    "buyToken": buy_token,
    "sellAmount": str(amount),
    "buyAmount": str(min_amount),
    "validTo": deadline_timestamp,
    "appData": "0x2B8694ED30082129598720860E8E972F07AA10D9B81CAE16CA0E2CFB24743E24",  # Yearn
    "feeAmount": "0",
    "kind": "sell",
    "partiallyFillable": True,
    "receiver": receiver,
    "signature": wallet_address,
    "from": wallet_address,
    "sellTokenBalance": "erc20",
    "buyTokenBalance": "erc20",
    "signingScheme": "presign",
}
# POST to /orders → returns order_uid
# Then call settlement.setPreSignature(order_uid, True) on-chain
```

### Check Order Status

```bash
curl https://api.cow.fi/mainnet/api/v1/orders/<order_uid>
```

### Get Trades for an Order

```bash
curl https://api.cow.fi/mainnet/api/v1/trades?orderUid=<order_uid>
```

## Key Contracts

| Contract | Address (Mainnet) |
|----------|-------------------|
| GPv2Settlement | `0x9008D19f58AAbD9eD0D60971565AA8510560ab41` |
| GPv2VaultRelayer | `0xC92E8bdf79f0507f65a392b0ab4667716BFE0110` |

- **Approve tokens to VaultRelayer**, not Settlement
- Settlement contract handles execution; VaultRelayer pulls tokens

## EIP-712 Signing (EOA with Foundry)

CoW Protocol orders use EIP-712 typed data. The domain:

```
name: "Gnosis Protocol"
version: "v2"
chainId: 1
verifyingContract: 0x9008D19f58AAbD9eD0D60971565AA8510560ab41
```

Order type hash fields: `sellToken, buyToken, receiver, sellAmount, buyAmount, validTo, appData, feeAmount, kind, partiallyFillable, sellTokenBalance, buyTokenBalance`

To sign with cast, compute the struct hash and sign with `cast wallet sign --private-key`.

## Order Types

| Type | Description |
|------|-------------|
| Market | Executes ASAP at best price. Set `validTo` short (5-30 min) |
| Limit | Executes when price target hit. Set `validTo` up to 1 year |
| TWAP | Split large order into smaller parts over time (via ComposableCoW) |

## Flow

1. Get quote → know expected amounts
2. Approve sell token to VaultRelayer (exact amount, never max)
3. Sign order (EIP-712 for EOA, presign for contracts)
4. POST order to API
5. Solvers compete to fill — MEV-protected
6. Check status / trades

## Benefits over Direct DEX Swaps

- **MEV protection** — solvers execute, users never exposed on-chain
- **Gasless** — users sign intents, solvers pay gas
- **Surplus capture** — if execution beats the quote, user gets the surplus
- **CoWs** — peer-to-peer matching skips AMM fees entirely
- **No failed tx fees** — only pay if order fills

## Explorer

Track orders: `https://explorer.cow.fi/orders/<order_uid>`

## Docs

- https://docs.cow.fi
- API spec: https://api.cow.fi/mainnet/api/v1
