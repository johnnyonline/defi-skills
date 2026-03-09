# Lender-Borrower Strategy

Borrow at a low interest rate, deposit borrowed tokens into a higher-yield vault, profit on the spread.

## Overview

```
┌──────────────┐     borrow      ┌──────────────┐     deposit     ┌──────────────┐
│   Lending    │ ──────────────► │   Borrowed   │ ──────────────► │    Yield     │
│   Protocol   │                 │    Token     │                 │    Vault     │
│  (e.g. Aave) │ ◄────────────  │              │ ◄────────────── │ (e.g. Yearn) │
└──────────────┘     repay       └──────────────┘    withdraw     └──────────────┘
       ▲
       │ supply collateral
       │
┌──────────────┐
│  Collateral  │
│   Token      │
└──────────────┘
```

Profit = vault earn rate - borrow rate (net APR must be positive).

## Parameters

Every position needs these defined upfront:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `collateralToken` | Token supplied as collateral | WETH |
| `borrowToken` | Token borrowed from lending protocol | USDC |
| `targetLTV` | Target loan-to-value ratio | 75% |
| `maxLTV` | Upper bound — delever if exceeded | 80% |
| `minLTV` | Lower bound — lever up if below | 70% |
| `minNetAPR` | Minimum acceptable spread | 1% |
| `monitorInterval` | How often to check the position | 1h |

## Opening a Position

1. **Supply collateral** to lending protocol
   - See: lending protocol skill (e.g. `aave-v3/SKILL.md` → Supply)

2. **Borrow** against collateral up to `targetLTV`
   - Calculate borrow amount: `collateralValue * targetLTV / borrowTokenPrice`
   - See: lending protocol skill (e.g. `aave-v3/SKILL.md` → Borrow)

3. **Swap** borrowed tokens if vault requires a different token
   - See: `enso/SKILL.md` → Swap

4. **Deposit** into yield vault
   - See: vault skill (e.g. `yearn-v3/SKILL.md` → Deposit)

## Monitoring

Check these on every `monitorInterval`:

### 1. Current LTV

```
currentLTV = totalDebtValue / totalCollateralValue
```

From lending protocol (e.g. Aave `getUserAccountData` returns both in base currency).

### 2. Health Factor

```
healthFactor = (collateralValue * liquidationThreshold) / debtValue
```

- `> 2.0` — safe
- `1.5 - 2.0` — monitor closely
- `1.0 - 1.5` — WARNING, consider deleveraging
- `< 1.0` — LIQUIDATABLE

### 3. Borrow Rate

Current variable borrow rate from lending protocol.
- Aave: `getReserveData` → `currentVariableBorrowRate` (ray, divide by 1e27)

### 4. Earn Rate

Current yield from the vault.
- Yearn: APR oracle (see `yearn-v3/SKILL.md` — APR section when available)

### 5. Net APR

```
netAPR = earnRate - borrowRate
```

- `> minNetAPR` — position is healthy
- `0 < netAPR < minNetAPR` — alert, consider unwinding
- `< 0` — losing money, unwind or alert immediately

## Rebalancing

### Delever (LTV too high)

Triggered when `currentLTV > maxLTV`.

1. Withdraw enough from yield vault to cover repayment
2. Swap back to borrow token if needed (enso)
3. Repay partial debt to bring LTV back to `targetLTV`

```
excessDebt = totalDebt - (totalCollateral * targetLTV)
repayAmount = excessDebt
```

### Lever Up (LTV too low)

Triggered when `currentLTV < minLTV`.

1. Borrow more to bring LTV up to `targetLTV`
2. Swap if needed (enso)
3. Deposit additional tokens into yield vault

```
additionalBorrow = (totalCollateral * targetLTV) - totalDebt
```

### Emergency Unwind

Triggered when `healthFactor < 1.2` or `netAPR < 0` for extended period.

1. Withdraw all from yield vault
2. Swap back to borrow token if needed
3. Repay all debt
4. Withdraw collateral

Always unwind in this order — repay before withdrawing collateral.

## Cron Setup

Set up a monitoring cron at `monitorInterval`:

```
Check position health:
1. Read currentLTV, healthFactor, borrowRate, earnRate
2. Calculate netAPR
3. If currentLTV > maxLTV → alert (or auto-delever if approved)
4. If currentLTV < minLTV → alert (or auto-lever if approved)
5. If healthFactor < 1.5 → alert
6. If netAPR < minNetAPR → alert
7. Log all values for tracking
```

**Important:** Never auto-execute rebalancing without explicit approval. Alert and propose the action.

## Position Tracking

Log position state on every check:

```json
{
  "timestamp": "2026-03-09T12:00:00Z",
  "collateralToken": "WETH",
  "borrowToken": "USDC",
  "collateralValue": 10000,
  "debtValue": 7500,
  "currentLTV": 0.75,
  "healthFactor": 1.85,
  "borrowRate": 0.035,
  "earnRate": 0.082,
  "netAPR": 0.047,
  "vaultShares": 7500.00,
  "status": "healthy"
}
```

## Risk Factors

- **Borrow rate spikes**: High utilization on lending protocol can spike variable rates above earn rate
- **Collateral price drops**: LTV increases as collateral loses value → potential liquidation
- **Vault yield drops**: Earn rate decreases, net APR narrows or goes negative
- **Smart contract risk**: Both lending protocol and vault have contract risk
- **Liquidation cascades**: In market crashes, liquidations can accelerate price drops
- **Oracle risk**: Stale or manipulated price feeds can cause unexpected liquidations

## Protocol References

This strategy is protocol-agnostic. Use these skills for execution:

| Function | Skill |
|----------|-------|
| Supply/borrow/repay/withdraw collateral | `aave-v3/SKILL.md` (or other lending protocol) |
| Deposit/withdraw yield vault | `yearn-v3/SKILL.md` (or other vault) |
| Token swaps | `enso/SKILL.md` |
| Token addresses | `addresses/SKILL.md` |
| Wallet balances | `enso/SKILL.md` → Wallet Balances |
