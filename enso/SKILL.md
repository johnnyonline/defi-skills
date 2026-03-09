# Enso

DeFi routing API. Finds optimal swap/deposit/redeem paths across protocols and chains.

Docs: https://docs.enso.build/pages/build/get-started/overview
API: https://api.enso.build

## Auth

All requests need `Authorization: Bearer $ENSO_API_KEY` header.

## Route API

Single endpoint for swaps, vault zaps, crosschain transfers. Returns ready-to-sign tx calldata.

```bash
curl -s -X POST "https://api.enso.build/api/v1/shortcuts/route" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ENSO_API_KEY" \
  -d '{
    "chainId": 1,
    "fromAddress": "0xYOUR_ADDRESS",
    "routingStrategy": "router",
    "tokenIn": ["0xTOKEN_IN"],
    "tokenOut": ["0xTOKEN_OUT"],
    "amountIn": ["AMOUNT_IN_WEI"],
    "slippage": "100"
  }'
```

### Response fields
- `tx.to` — router address to call
- `tx.data` — calldata to execute
- `tx.value` — ETH value (for native ETH swaps)
- `amountOut` — simulated output amount
- `gas` — estimated gas

### Routing strategies
- `router` — caller approves tokens to Enso router, router pulls tokens. Needs separate approval tx.
- `delegate` — uses delegatecall, no approval needed (for smart contract wallets)

### Slippage
Basis points. `100` = 1%, `50` = 0.5%, `300` = 3%.

### Special addresses
- ETH: `0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee`

### Crosschain
Add `destinationChainId` to the request body. Use higher slippage (3-5%) for crosschain.

## Using with cast

```bash
# 1. Get route
ROUTE=$(curl -s -X POST "https://api.enso.build/api/v1/shortcuts/route" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ENSO_API_KEY" \
  -d '{
    "chainId": 1,
    "fromAddress": "'$WALLET'",
    "routingStrategy": "router",
    "tokenIn": ["'$TOKEN_IN'"],
    "tokenOut": ["'$TOKEN_OUT'"],
    "amountIn": ["'$AMOUNT'"],
    "slippage": "100"
  }')

ROUTER=$(echo $ROUTE | jq -r '.tx.to')
CALLDATA=$(echo $ROUTE | jq -r '.tx.data')
VALUE=$(echo $ROUTE | jq -r '.tx.value')

# 2. Approve token to router (skip for ETH in)
cast send $TOKEN_IN "approve(address,uint256)" $ROUTER $AMOUNT --rpc-url $RPC --private-key $PK

# 3. Execute swap
cast send $ROUTER $CALLDATA --value $VALUE --rpc-url $RPC --private-key $PK
```

## Using from Vyper/Solidity

```python
# Approve input token to the router
assert extcall IERC20(token_in).approve(swap.router, amount_in, default_return_value=True)

# Execute the swap
raw_call(swap.router, swap.data)
```

## Bundle API

For custom multi-step workflows (borrow, claim rewards, CLMM positions). Use when Route API isn't enough.

Endpoint: `POST /api/v1/shortcuts/bundle`

## Gotchas
- `amountIn` is in wei (raw token decimals)
- Always use `jq` to parse response — extract `tx.data`, `tx.to`, `tx.value`
- For `router` strategy: must approve tokens to the router address first
- For `delegate` strategy: no approval needed but only works with smart contract wallets
- Higher slippage for crosschain ops and volatile pairs
