---
name: specular
description: Interact with Specular Protocol — DeFi credit infrastructure for AI agents. Deploy agents, borrow USDC, earn yield, check credit scores, and manage positions. Use when the user mentions Specular, agent credit, DeFi borrowing for agents, or lending USDC to agents.
metadata:
  {
    "openclaw":
      {
        "emoji": "🔶",
        "requires": { "env": ["SPECULAR_API_KEY"] },
        "primaryEnv": "SPECULAR_API_KEY",
      },
  }
---

# Specular Protocol Skill

You can interact with Specular, a DeFi credit infrastructure protocol for AI agents on Base. Specular lets AI agents borrow USDC against onchain collateral, building credit history to unlock better rates over time. Users can also deposit USDC to earn yield from agent borrowers.

**Base URL:** `https://api.specular.financial/v1`
**Docs:** https://specular.financial/app (Docs tab)
**GitHub:** https://github.com/thegrand-canyon/specular

## Authentication

All requests require the header:

```
Authorization: Bearer $SPECULAR_API_KEY
```

If the user hasn't set up their API key, direct them to https://specular.financial to create an account and generate one.

## What You Can Do

### 1. Agent Management

**Create a new agent:**

```bash
curl -X POST https://api.specular.financial/v1/agents \
  -H "Authorization: Bearer $SPECULAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "my-agent"}'
```

Returns: `{ id, address, score: 500, wallet }` — every new agent starts with a score of 500.

**Get agent details:**

```bash
curl https://api.specular.financial/v1/agents/{agent_id} \
  -H "Authorization: Bearer $SPECULAR_API_KEY"
```

Returns: `{ id, name, address, score, wallet, activeLoans, totalBorrowed, totalRepaid, repaymentRate }`

**List all agents:**

```bash
curl https://api.specular.financial/v1/agents \
  -H "Authorization: Bearer $SPECULAR_API_KEY"
```

### 2. Borrowing (for Agents)

**Get current terms for an agent:**

```bash
curl https://api.specular.financial/v1/agents/{agent_id}/terms \
  -H "Authorization: Bearer $SPECULAR_API_KEY"
```

Returns: `{ rate, collateralRatio, score }` — the rate and collateral requirements based on the agent's current credit score.

**Borrow USDC:**

```bash
curl -X POST https://api.specular.financial/v1/agents/{agent_id}/borrow \
  -H "Authorization: Bearer $SPECULAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "principal": "10000",
    "collateral": [
      {"asset": "WETH", "amount": "5.0"},
      {"asset": "wstETH", "amount": "2.0"}
    ]
  }'
```

Returns: `{ loanId, rate, collateralRatio, collateralValue, txHash }`

**Important collateral rules:**
- You cannot use USDC as collateral (you're borrowing USDC)
- Prefer premium collateral: WBTC > WETH/wstETH > LINK/AAVE > other tokens
- Each asset has a haircut (safety discount): e.g. WETH at 20% means $100 of WETH = $80 collateral value
- Multi-asset collateral baskets are supported — mix assets in one borrow

**Repay a loan:**

```bash
curl -X POST https://api.specular.financial/v1/agents/{agent_id}/repay \
  -H "Authorization: Bearer $SPECULAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"loanId": "loan-abc123"}'
```

Returns: `{ txHash, newScore, scoreChange }` — repaying on time improves the agent's credit score.

### 3. Earning Yield (for Users)

**Check pool stats:**

```bash
curl https://api.specular.financial/v1/pool \
  -H "Authorization: Bearer $SPECULAR_API_KEY"
```

Returns: `{ totalDeposits, totalBorrowed, utilization, apy, sharePrice }`

**Deposit USDC to earn yield:**

```bash
curl -X POST https://api.specular.financial/v1/pool/deposit \
  -H "Authorization: Bearer $SPECULAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"amount": "10000"}'
```

Returns: `{ shares, apy, txHash }` — you receive spUSDC shares that appreciate automatically.

**Check your position:**

```bash
curl https://api.specular.financial/v1/pool/position \
  -H "Authorization: Bearer $SPECULAR_API_KEY"
```

Returns: `{ deposited, earned, shares, currentApy, dailyYield }`

**Withdraw:**

```bash
curl -X POST https://api.specular.financial/v1/pool/withdraw \
  -H "Authorization: Bearer $SPECULAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"shares": "5000"}'
```

### 4. Credit Score

Specular uses an onchain credit scoring system (0–1000):
- **Score formula:** S(a) = clamp(500 + R + V − D − L, 0, 1000)
  - R = Repayment history (up to +400)
  - V = Loan volume (up to +200, maxes at 10 loans)
  - D = Default penalty (up to −300)
  - L = Leverage penalty (up to −100)
- **Higher score = lower rates + less collateral required**
- New agents start at 500. A perfect agent can reach 1000.
- Score improves with each successful repayment.

## Behavior Guidelines

When the user asks about Specular:

- **"Check my agent" / "How's my agent doing"** → Call GET /agents/{id}, show score, rate, active loans, wallet value
- **"Deploy an agent" / "Create agent"** → Call POST /agents with a name
- **"Borrow" / "Get a loan"** → First check terms (GET /terms), then execute borrow (POST /borrow)
- **"Repay"** → Call POST /repay with the loan ID
- **"How much am I earning" / "Check yield"** → Call GET /pool/position
- **"Deposit"** → Call POST /pool/deposit
- **"What are my rates"** → Call GET /agents/{id}/terms and explain the rate + collateral ratio
- **"Improve my score"** → Explain the scoring formula and suggest borrowing + repaying on time

Always confirm with the user before executing transactions (borrow, repay, deposit, withdraw). Show them the details and get approval first.

When showing agent data, format it cleanly:
```
🔶 Agent: Atlas-7
   Score: 920 (92nd percentile)
   Rate: 2.1% APR
   Collateral: 100%
   Active Loans: 2 ($25K outstanding)
   Wallet: $87K across 7 assets
```

When showing yield data:
```
💰 Your Position
   Deposited: $10,000 USDC
   Earned: $42.30
   APY: 8.4%
   Daily: +$2.30/day
```

## Network

Specular is deployed on **Base** (Ethereum L2). All transactions settle on Base. The protocol uses USDC as the primary lending asset.

## Contracts (Reference)

- **SpecularIdentity** — Agent registration and credit scoring
- **SpecularPool** — ERC-4626 lending vault (spUSDC)
- **SpecularLending** — Loan lifecycle management
- **SpecularCollateral** — Multi-asset collateral baskets and liquidation

## Links

- Website: https://specular.financial
- Dashboard: https://specular.financial/app
- GitHub: https://github.com/thegrand-canyon/specular
- Whitepaper: https://specular.financial/specular-whitepaper.pdf
