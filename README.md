# 🔶 Specular — OpenClaw Skill

Give your AI agent access to DeFi credit. Borrow USDC, earn yield, manage credit scores — all through natural conversation.

## What it does

- **Deploy agents** — spin up onchain identities that can borrow
- **Borrow USDC** — against multi-asset collateral (WETH, wstETH, WBTC, etc.)
- **Build credit** — every repayment improves your agent's score (0–1000)
- **Earn yield** — deposit USDC to earn from agent borrowers (~8% APY)
- **Monitor positions** — check scores, rates, health factors, loan status

## Install

### Via ClawHub

```bash
clawhub install specular
```

### Manual

```bash
# Clone into your OpenClaw skills directory
cd ~/.openclaw/skills
git clone https://github.com/thegrand-canyon/specular-openclaw-skill specular
```

## Setup

1. Get an API key at [specular.financial](https://specular.financial)
2. Add to your OpenClaw environment:

```bash
# In your openclaw.json or .env
SPECULAR_API_KEY=your-api-key-here
```

## Usage

Just talk to your agent:

> "Create a new Specular agent called trading-bot-1"

> "Check my agent's credit score"

> "Borrow 10K USDC using my WETH as collateral"

> "Repay my active loan"

> "How much yield am I earning?"

> "Deposit 5000 USDC into the Specular pool"

## How Specular Works

Specular is credit infrastructure for AI agents on Base.

**For agents (borrowers):**
- Register an onchain identity → start with score 500
- Borrow USDC against collateral → rate depends on your score
- Repay on time → score improves → rates drop, collateral requirements shrink
- Score formula: `S = 500 + Repayment + Volume − Defaults − Leverage`

**For users (lenders):**
- Deposit USDC into the pool → receive spUSDC shares
- Shares appreciate as agents pay interest
- No claiming or compounding — yield accrues automatically

## Links

- [Specular Dashboard](https://specular.financial/app)
- [GitHub](https://github.com/thegrand-canyon/specular)
- [Whitepaper](https://specular.financial/specular-whitepaper.pdf)

## License

MIT
