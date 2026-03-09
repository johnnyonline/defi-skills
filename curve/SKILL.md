# Curve

DEX for stableswaps and volatile pairs. Uses StableSwap and CryptoSwap invariants.

Docs: https://docs.curve.finance/developer/amm/curve-amm-overview

## Pool Types

- **StableSwap-NG**: Pegged assets (stables, LSTs). Supports plain pools + metapools. Asset types: standard ERC20 (0), oracle/rate (1), rebasing (2), ERC4626 (3)
- **TwoCrypto-NG**: 2-coin volatile pairs
- **TriCrypto-NG**: 3-coin volatile pairs (e.g. ETH/BTC/USD)

## Contracts (Mainnet)

- Address Provider: `0x0000000022D53366457F9d5E68Ec105046FC4383`
- Router NG: `0xD16d5eC345Dd86Fb63C6a9C43c517210F1027914`
- StableSwap-NG Factory: query Address Provider
- TwoCrypto-NG Factory: query Address Provider
- TriCrypto-NG Factory: query Address Provider

Source: https://github.com/curvefi/stableswap-ng | https://github.com/curvefi/curve-router-ng

## Direct Pool Swap

For swapping directly on a known pool:

```
# Approve token first
cast send <token> "approve(address,uint256)" <pool> <amount> --rpc-url $RPC --private-key $PK

# Exchange: i = input index, j = output index
cast send <pool> "exchange(int128,int128,uint256,uint256)" <i> <j> <amount> <min_out> --rpc-url $RPC --private-key $PK

# Get expected output
cast call <pool> "get_dy(int128,int128,uint256)(uint256)" <i> <j> <amount> --rpc-url $RPC

# Get coin at index
cast call <pool> "coins(uint256)(address)" <index> --rpc-url $RPC
```

## Router Swap

For multi-hop swaps across pools. Routing must be determined off-chain.

```
exchange(
  _route: address[11],       # [tokenIn, pool1, tokenMid, pool2, tokenOut, 0x0...]
  _swap_params: uint256[5][5], # [[i, j, swap_type, pool_type, n_coins], ...]
  _amount: uint256,
  _expected: uint256,        # min output (slippage protection)
  _pools: address[5],        # only for swap_type=3 (zap)
  _receiver: address
)
```

### swap_type values
1. `exchange` (standard)
2. `exchange_underlying` (metapool)
3. underlying via zap
4. coin → LP token (add_liquidity)
5. lending underlying → LP token
6. LP token → coin (remove_liquidity_one_coin)
7. LP token → lending underlying
8. ETH ↔ WETH, ETH → stETH/frxETH, stETH ↔ wstETH, frxETH ↔ sfrxETH
9. SNX synth swaps

### pool_type values
1 = stable, 2 = crypto, 3 = tricrypto, 4 = llamma

## exchange_received

Swap without approvals — send tokens directly to the pool, then call:
```
pool.exchange_received(i, j, _dx, _min_dy, _receiver)
```
Uses balance diff instead of transferFrom. Good for aggregators/bots.

## Fees

Dynamic fees based on pool balance. Off-peg = higher fees. Base fee queryable via `pool.fee()`, multiplier via `pool.offpeg_fee_multiplier()`.

## Gotchas
- Router params must be computed off-chain (no on-chain route finder)
- Always set `_min_dy` / `_expected` properly — never use 0 in production
- Pool coin indices start at 0, query `coins(uint256)` to verify
