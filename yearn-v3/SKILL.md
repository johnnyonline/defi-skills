# Yearn V3

Decentralized yield-generating vaults. All ERC-4626 compliant.

Docs: https://docs.yearn.fi/developers/v3/overview
AI corpus: https://docs.yearn.fi/llms.txt

## Concepts

- **Vault** (Allocator Vault): ERC-4626 vault that allocates deposits across multiple strategies
- **TokenizedStrategy**: Single-strategy ERC-4626 vault. Can be standalone or plugged into an Allocator Vault
- **Periphery**: Optional add-ons (Accountant for fees, Debt Allocator, APR Oracle, Router, etc.)

## Core Contracts (Mainnet, v3.0.4)

- Vault original: `0xd8063123BBA3B480569244AE66BFE72B6c84b00d`
- VaultFactory: `0x770D0d1Fb036483Ed4AbB6d53c1C88fb277D812F`
- TokenizedStrategy: `0xD377919FA87120584B21279a491F82D5265A139c`

## Key Periphery (Mainnet)

- Address Provider: `0x775F09d6f3c8D2182DFA8bce8628acf51105653c`
- Release Registry: `0x0377b4daDDA86C89A0091772B79ba67d0E5F7198`
- V3 Registry: `0xd40ecF29e001c76Dcc4cC0D9cd50520CE845B038`
- 4626 Router: `0x1112dbCF805682e828606f74AB717abf4b4FD8DE`
- APR Oracle: `0x1981AD9F44F2EA9aDd2dC4AD7D075c102C70aF92`
- Role Manager: `0xb3bd6b2e61753c311efbcf0111f75d29706d9a41`
- Accountant: `0x5A74Cb32D36f2f517DB6f7b0A0591e09b22cDE69`
- Role Manager Factory: `0xca12459a931643BF28388c67639b3F352fe9e5Ce`

## Interacting

### Deposit
```
token.approve(vault, amount)
vault.deposit(amount, receiver)
```

### Withdraw
Use `redeem` over `withdraw`. Optional `maxLoss` param (basis points).
```
vault.redeem(shares, receiver, owner, maxLoss)
```
- `redeem` defaults maxLoss to 10000 (100%)
- `withdraw` defaults maxLoss to 0 (0%)
- Multi-strategy vaults accept optional `strategies` array to pick withdrawal order

### Pricing
- `convertToShares(assets)` / `convertToAssets(shares)` — standard 4626, safe for on-chain use
- `pricePerShare()` — legacy helper, off-chain only (precision loss)

### Registry Queries
- `getAllEndorsedVaults()` — nested array by asset
- `getEndorsedVaults(asset)` — all vaults for one asset
- `vaultInfo(vault)` — asset, version, type, tag

## Multi-Chain

Contracts use create2, addresses are stable across chains. Active on: Ethereum, Polygon, Base, Arbitrum, Katana.

## Source

- Vaults: https://github.com/yearn/yearn-vaults-v3
- TokenizedStrategy: https://github.com/yearn/tokenized-strategy
