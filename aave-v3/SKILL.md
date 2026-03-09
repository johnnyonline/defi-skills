# Aave V3

Non-custodial lending/borrowing protocol. Supply assets to earn yield, borrow against collateral.

Docs: https://aave.com/docs/aave-v3/overview

## Contracts (Mainnet)

- Pool: `0x87870Bca3F3fD6335C3F4ce8392D69350B4fA4E2`
- PoolAddressesProvider: `0x2f39d218133AFaB8F2B819B1066c7E434Ad94E9e`
- PoolDataProvider: query from PoolAddressesProvider

Multi-chain addresses in the addresses skill or at https://aave.com/docs

## Concepts

- **Supply**: deposit tokens → receive aTokens (interest-bearing, balance grows over time)
- **Borrow**: post collateral → borrow other tokens → receive variableDebtTokens (debt grows over time)
- **Health Factor**: collateral value / debt value. Below 1 = liquidatable
- **E-Mode**: higher LTV for correlated assets (e.g. ETH/wstETH, stablecoins)
- **aToken**: ERC-20 receipt token, balance auto-increases with interest
- **variableDebtToken**: ERC-20 debt token, balance auto-increases with interest

## Supply

```bash
# Approve token to Pool
cast send $TOKEN "approve(address,uint256)" $POOL $AMOUNT --rpc-url $RPC --private-key $PK

# Supply (referralCode 0)
cast send $POOL "supply(address,uint256,address,uint16)" $TOKEN $AMOUNT $ON_BEHALF_OF 0 --rpc-url $RPC --private-key $PK
```

## Withdraw

```bash
# Withdraw (use type(uint256).max for full withdrawal)
cast send $POOL "withdraw(address,uint256,address)" $TOKEN $AMOUNT $RECEIVER --rpc-url $RPC --private-key $PK
```

## Borrow

Must have collateral supplied first. Collateral is auto-enabled on first supply for most assets.

```bash
# Borrow (interestRateMode: 2 = variable, referralCode 0)
cast send $POOL "borrow(address,uint256,uint256,uint16,address)" $TOKEN $AMOUNT 2 0 $ON_BEHALF_OF --rpc-url $RPC --private-key $PK
```

## Repay

```bash
# Approve debt token to Pool
cast send $TOKEN "approve(address,uint256)" $POOL $AMOUNT --rpc-url $RPC --private-key $PK

# Repay (interestRateMode: 2 = variable, use type(uint256).max for full repay)
cast send $POOL "repay(address,uint256,uint256,address)" $TOKEN $AMOUNT 2 $ON_BEHALF_OF --rpc-url $RPC --private-key $PK
```

## Read Position

```bash
# Get user account data (returns totalCollateralBase, totalDebtBase, availableBorrowsBase, currentLiquidationThreshold, ltv, healthFactor)
cast call $POOL "getUserAccountData(address)(uint256,uint256,uint256,uint256,uint256,uint256)" $USER --rpc-url $RPC

# Get aToken balance (= supplied + interest)
cast call $A_TOKEN "balanceOf(address)(uint256)" $USER --rpc-url $RPC

# Get debt balance
cast call $DEBT_TOKEN "balanceOf(address)(uint256)" $USER --rpc-url $RPC

# Get reserve aToken/debtToken addresses
cast call $POOL_DATA_PROVIDER "getReserveTokensAddresses(address)(address,address,address)" $TOKEN --rpc-url $RPC
```

## E-Mode

```bash
# Set e-mode category (0 = none, 1+ = specific category)
cast send $POOL "setUserEMode(uint8)" $CATEGORY_ID --rpc-url $RPC --private-key $PK

# Get user e-mode
cast call $POOL "getUserEMode(address)(uint256)" $USER --rpc-url $RPC
```

## Reserve Data

```bash
# Get reserve data (includes current rates)
cast call $POOL "getReserveData(address)" $TOKEN --rpc-url $RPC

# Get reserve config (LTV, liquidation threshold, etc.)
cast call $POOL_DATA_PROVIDER "getReserveConfigurationData(address)(uint256,uint256,uint256,uint256,uint256,bool,bool,bool,bool,bool)" $TOKEN --rpc-url $RPC
```

## Lend-Borrow Strategy Pattern

From the Yearn lender-borrower strategy, the common flow is:
1. Supply collateral (e.g. wstETH) → get aToken
2. Borrow against it (e.g. USDC) → get variableDebtToken
3. Deposit borrowed tokens into a yield vault (e.g. Yearn)
4. Profit = vault yield - borrow rate

Key checks:
- `balanceOfCollateral()` = aToken balance
- `balanceOfDebt()` = variableDebtToken balance
- Net APR = lender vault APR - borrow variable rate
- Monitor health factor to avoid liquidation

Reference: https://github.com/johnnyonline/yv3-aave-lender-borrower-strategy

## Gotchas
- Health factor < 1 = liquidatable. Monitor closely when borrowing.
- Interest rates are variable — borrow rate can spike with high utilization
- `getUserAccountData` returns values in base currency (USD with 8 decimals)
- `interestRateMode`: always use `2` (variable). Stable rate (1) is deprecated in V3
- Supply is auto-enabled as collateral for most assets. Use `setUserUseReserveAsCollateral` to toggle
