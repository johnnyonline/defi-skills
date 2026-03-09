# Odos

DEX aggregator with smart order routing. Alternative to Enso for token swaps.

Docs: https://docs.odos.xyz/api/sor/

## Setup

Requires `ODOS_API_KEY` env var. Store at `/root/.alfred/odos-api-key`.

## Swap Flow

Two-step process: Quote → Assemble → Send

### 1. Get Quote

```bash
QUOTE=$(curl -s -X POST "https://api.odos.xyz/sor/quote/v3" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ODOS_API_KEY" \
  -d "{
    \"chainId\": $CHAIN_ID,
    \"inputTokens\": [{\"tokenAddress\": \"$INPUT_TOKEN\", \"amount\": \"$AMOUNT\"}],
    \"outputTokens\": [{\"tokenAddress\": \"$OUTPUT_TOKEN\", \"proportion\": 1}],
    \"userAddr\": \"$SENDER\",
    \"slippageLimitPercent\": 1,
    \"compact\": true
  }")
```

Extract `pathId` from quote response.

### 2. Assemble Transaction

```bash
ASSEMBLE=$(curl -s -X POST "https://api.odos.xyz/sor/assemble" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ODOS_API_KEY" \
  -d "{
    \"pathId\": \"$PATH_ID\",
    \"userAddr\": \"$SENDER\"
  }")
```

Extract `data` (calldata) and `to` (router address) from assemble response.

### 3. Simulate + Send

```bash
# Simulate first
cast call $ROUTER $TX_DATA --from $SENDER --rpc-url $RPC

# If simulation passes, send
cast send $ROUTER $TX_DATA --value 0 --rpc-url $RPC --private-key $PK
```

## Helper Script

`get_odos_swap.sh` — returns hex calldata for a swap:

```bash
./get_odos_swap.sh <chain_id> <input_token> <output_token> <amount> <sender>
```

## IMPORTANT

- Always approve the input token to the Odos router before swapping
- Route data expires quickly — quote, assemble, and send in one shot
- Always simulate before sending (`cast call`)
- Router address comes from the assemble response `transaction.to` field

## Router (Mainnet)

Check assemble response for current router address. Do not hardcode.
