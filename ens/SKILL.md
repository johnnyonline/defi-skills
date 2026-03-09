# ENS

Ethereum Name Service. Human-readable names → addresses.

Docs: https://docs.ens.domains
LLM docs: https://docs.ens.domains/llms.txt
App: https://app.ens.domains

## Contracts (Mainnet)

- Registry: `0x00000000000C2E074eC69A0dFb2997BA6C7d2e1e`
- ETH Registrar Controller: `0x253553366Da8546fC250F225fe3d25d0C782303b`
- Base Registrar: `0x57f1887a8BF19b14fC0dF6Fd9B2acc9Af147eA85`
- Public Resolver: `0xF29100983E058B709F3D539b0c765937B804AC15`
- Name Wrapper: `0xD4416b13d2b3a9aBae7AcD5D6C2BbDBE25686401`
- Reverse Registrar: `0xa58E81fe9b61B5c3fE2AFD33CF304c454AbFc7Cb`
- Universal Resolver: `0xeEeEEEeE14D718C2B47D9923Deab1335E144EeEe`

## Pricing (.eth, per year)

| Length | USD/year |
|--------|----------|
| 5+ chars | $5 |
| 4 chars | $160 |
| 3 chars | $640 |

Price paid in ETH (USD-denominated via Chainlink oracle). Send 5-10% extra for price slippage — excess is refunded.

## Register a .eth Name

Two-step commit-reveal process (prevents frontrunning). Wait 60s between commit and register.

### 1. Check availability & price

```bash
CONTROLLER=0x253553366Da8546fC250F225fe3d25d0C782303b

# Check available
cast call $CONTROLLER "available(string)(bool)" "myname" --rpc-url $RPC

# Get price (returns (base, premium) in wei)
cast call $CONTROLLER "rentPrice(string,uint256)((uint256,uint256))" "myname" 31536000 --rpc-url $RPC
```

### 2. Commit

Generate a random 32-byte secret, then compute and submit commitment.

```bash
# Generate secret
SECRET=$(cast keccak $(cast to-hex $(date +%s%N)$RANDOM))

# Make commitment hash
COMMITMENT=$(cast call $CONTROLLER "makeCommitment(string,address,uint256,bytes32,address,bytes[],bool,uint16)(bytes32)" \
  "myname" $OWNER 31536000 $SECRET $RESOLVER "[]" false 0 --rpc-url $RPC)

# Submit commit
cast send $CONTROLLER "commit(bytes32)" $COMMITMENT --rpc-url $RPC --private-key $PK
```

### 3. Wait 60 seconds

### 4. Register

```bash
# Send with ~5% extra ETH for price slippage (excess refunded)
cast send $CONTROLLER "register(string,address,uint256,bytes32,address,bytes[],bool,uint16)" \
  "myname" $OWNER 31536000 $SECRET $RESOLVER "[]" false 0 \
  --value $PRICE_PLUS_SLIPPAGE --rpc-url $RPC --private-key $PK
```

### Parameters

- `name`: label only (e.g. `"myname"` not `"myname.eth"`)
- `owner`: address that will own the name
- `duration`: seconds (31536000 = 1 year)
- `secret`: random bytes32 (must match between commit and register)
- `resolver`: use Public Resolver `0xF29100983E058B709F3D539b0c765937B804AC15`
- `data`: encoded resolver calls (e.g. `setAddr`). Use `[]` for none
- `reverseRecord`: `true` to set as primary name
- `ownerControlledFuses`: `0` for default

## Resolve & Lookup

```bash
# Name → address
cast resolve-name myname.eth --rpc-url $RPC

# Address → primary name
cast lookup-address 0x1234... --rpc-url $RPC

# Read text record
cast call $UNIVERSAL_RESOLVER "resolve(bytes,bytes)(bytes,address)" ... --rpc-url $RPC
```

## Renew

Anyone can renew any name. Same pricing as registration.

```bash
cast send $CONTROLLER "renew(string,uint256)" "myname" 31536000 --value $PRICE --rpc-url $RPC --private-key $PK
```

## Set Address Record (Forward Resolution)

Registration with `data: []` does NOT set the forward address record. You must set it separately so `name.eth` → `address` works.

```bash
NAMEHASH=$(cast namehash myname.eth)
cast send $RESOLVER "setAddr(bytes32,address)" $NAMEHASH $OWNER --rpc-url $RPC --private-key $PK
```

Alternatively, pass encoded `setAddr` call in the `data` param during registration to do it in one tx.

## Gotchas
- Commit-reveal: must wait 60s–24h between commit and register
- Price is in ETH but denominated in USD — send 5-10% extra
- Name must be 3+ characters
- Registration with `data: []` only sets ownership, NOT the address record — call `setAddr` separately or pass it in `data`
- `reverseRecord: true` sets reverse (address → name) but NOT forward (name → address)
- After expiry: 90-day grace period, then 21-day dutch auction
