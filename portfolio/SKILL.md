# Portfolio Management Skill

Protocol-agnostic portfolio tracking, accounting, and reporting. Orchestrates protocol-specific skills — never calls RPC directly.

## Architecture

```
portfolio/
  SKILL.md
  portfolio.json       # live state — positions, params, tx log
  snapshots/           # daily value snapshots (YYYY-MM-DD.json)
  scripts/
    report.sh          # full portfolio report (calls protocol skill scripts)
    log-tx.sh          # append transaction to tx log
    snapshot.sh        # daily snapshot of portfolio value
```

Each protocol skill (aave-v3, yearn-v3, etc.) owns its own `scripts/` directory. This skill calls those scripts — it never duplicates protocol logic.

## Portfolio State (`portfolio.json`)

```json
{
  "wallet": "0x...",
  "positions": [
    {
      "id": "lender-borrower-1",
      "strategy": "lender-borrower",
      "status": "active",
      "openedAt": "2025-03-09T...",
      "entries": [
        {
          "protocol": "aave-v3",
          "type": "collateral",
          "asset": "wstETH",
          "assetAddress": "0x...",
          "entryAmount": "0.0085",
          "entryPriceUSD": 2350
        },
        {
          "protocol": "aave-v3",
          "type": "debt",
          "asset": "USDC",
          "assetAddress": "0x...",
          "entryAmount": "12.0"
        },
        {
          "protocol": "yearn-v3",
          "type": "deposit",
          "vault": "yvUSDS-1",
          "vaultAddress": "0x...",
          "shares": "11.003",
          "entryAmount": "11.998"
        }
      ],
      "params": {
        "targetLTV": 0.60,
        "maxLTV": 0.68,
        "minLTV": 0.52
      }
    }
  ],
  "txLog": [
    {
      "timestamp": "2025-03-09T...",
      "txHash": "0x...",
      "action": "supply",
      "protocol": "aave-v3",
      "asset": "wstETH",
      "amount": "0.0085",
      "positionId": "lender-borrower-1"
    }
  ]
}
```

## Reporting

### Per Position
- Current value (USD + ETH)
- Health metrics (LTV, health factor for lending positions)
- Expected APR vs actual APR
- Unrealized PnL (current value - entry value)

### Portfolio Level
- Total value (USD + ETH)
- Weighted average APR
- Net PnL since inception

### Snapshots
Daily JSON snapshots for historical tracking:
```json
{
  "date": "2025-03-10",
  "totalValueUSD": 20.50,
  "totalValueETH": 0.0087,
  "positions": { "lender-borrower-1": { "valueUSD": 20.50 } },
  "netPnlUSD": -0.30
}
```

## Usage Pattern

The agent's job is orchestration only:
1. Read `portfolio.json` for current state
2. Call protocol skill scripts for live data
3. Aggregate, compare, report
4. Log any transactions via `log-tx.sh`

**Scripts handle the complexity. The agent decides which scripts to run and in which order.**

## Dependencies
- `aave-v3/scripts/*` — lending position data
- `yearn-v3/scripts/*` — vault balances, APRs
- `addresses/SKILL.md` — token/contract addresses
- `foundry/SKILL.md` — cast CLI reference
- `lender-borrower-strategy/SKILL.md` — strategy logic and formulas
