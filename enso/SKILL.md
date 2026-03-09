# Enso

DeFi routing API. Finds optimal swap/deposit/redeem paths across protocols and chains. Returns ready-to-sign tx calldata.

Docs: https://docs.enso.build/pages/build/get-started/overview
API: https://api.enso.build
Dashboard (API keys): https://developers.enso.build

## Auth

`Authorization: Bearer $ENSO_API_KEY` header on all requests. Key stored at `/root/.alfred/enso-api-key` (chmod 600).

If key is missing, create one at https://developers.enso.build or ask the user to provide one.

Rate limit: 10 RPS (600/min).

## Router Address

Dynamic per chain — returned as `tx.to` in route response. For approvals, use the `/approve` endpoint which returns `spender`.

## Route API

```bash
# Get swap calldata
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
```

### Execute with cast

```bash
# 1. Approve token to router (skip for ETH in)
cast send $TOKEN_IN "approve(address,uint256)" $ROUTER $AMOUNT --rpc-url $RPC --private-key $PK

# 2. Execute swap
cast send $ROUTER $CALLDATA --value $VALUE --rpc-url $RPC --private-key $PK
```

### Params

- `routingStrategy`: `router` (EOA, needs approval) or `delegate` (smart contract wallets, no approval)
- `slippage`: basis points. `100` = 1%, `50` = 0.5%
- `amountIn`: raw wei (token decimals)
- ETH address: `0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee`

### Crosschain

Add `destinationChainId` to request body. Use higher slippage (3-5%).

### Response

- `tx.to` — router address
- `tx.data` — calldata
- `tx.value` — ETH value
- `amountOut` — simulated output
- `gas` — estimated gas

## Approval Endpoint

```bash
curl -s -X GET "https://api.enso.build/api/v1/wallet/approve?chainId=1&fromAddress=$WALLET&routingStrategy=router&tokenAddress=$TOKEN&amount=$AMOUNT" \
  -H "Authorization: Bearer $ENSO_API_KEY"
```

Returns `tx` object + `spender` address.

## Wallet Balances

Get all tokens held by an address with prices.

```bash
curl -s "https://api.enso.build/api/v1/wallet/balances?chainId=1&eoaAddress=$ADDRESS" \
  -H "Authorization: Bearer $ENSO_API_KEY"
```

Returns array of `{token, amount, chainId, decimals, price, name, symbol}`.

## Bundle API

For multi-step workflows (borrow, claim, CLMM positions) beyond simple route.

Endpoint: `POST /api/v1/shortcuts/bundle`

## Gotchas
- Router address varies — always use `tx.to` from the response
- For `router` strategy: must approve tokens to the router first
- `delegate` strategy only works with smart contract wallets
- Higher slippage for crosschain and volatile pairs
