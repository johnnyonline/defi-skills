# Addresses

Verified contract and token addresses for onchain interactions.

> **CRITICAL:** Never guess or hallucinate an address. Wrong address = lost funds.

## Lookup Order

1. Check this file first
2. Check the community address list: https://ethskills.com/addresses/SKILL.md
3. Look up on block explorer or protocol docs
4. Verify with `cast code <address> --rpc-url $RPC` before interacting

## Custom Addresses

### Tokens (Mainnet)

| Token | Address |
|-------|---------|
| BOLD (Liquity V2) | `0x6440f144b7e50D6a8439336510312d2F54beB01D` |

### Yearn Vaults (Mainnet)

| Vault | Address |
|-------|---------|
| yvUSDC-1 | `0xBe53A109B494E5c9f97b9Cd39Fe969BE68BF6204` |
| yvUSDS-1 | `0x182863131F9a4630fF9E27830d945B1413e347E8` |

### Common Tokens (Mainnet)

| Token | Address |
|-------|---------|
| WETH | `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` |
| USDC | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` |
| USDT | `0xdAC17F958D2ee523a2206206994597C13D831ec7` |
| DAI | `0x6B175474E89094C44Da98b954EedeAC495271d0F` |
| wstETH | `0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0` |
| ETH (native) | `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE` |

## Verification

```bash
# Check contract exists
cast code <address> --rpc-url $RPC

# Check token name/symbol
cast call <address> "name()(string)" --rpc-url $RPC
cast call <address> "symbol()(string)" --rpc-url $RPC
cast call <address> "decimals()(uint8)" --rpc-url $RPC
```
