# Using OpenClaw with Solana

OpenClaw is an AI agent framework that supports autonomous Solana wallet management and trading through the `@solana-clawd/solana-wallet` plugin.

## Prerequisites

- **Node.js** 16.x or higher
- **npm** 8.x or higher
- **OpenClaw** (latest version) — see [openclaw.com](https://openclaw.com)
- Access to a Solana RPC endpoint

## 1. Install the Solana Plugin

From within an OpenClaw-enabled environment, install the Solana wallet plugin:

```bash
openclaw plugins install @solana-clawd/solana-wallet
```

This downloads and registers the plugin, compiles its TypeScript sources, and creates the required workspace directories under `~/.openclaw/workspace/`.

## 2. Configure Environment Variables

Set the following environment variables before starting your agent, or add them to `~/.openclaw/openclaw.json` for persistent configuration:

```bash
export SOLANA_RPC_URL="https://api.mainnet-beta.solana.com"
export SOLANA_WALLET_PATH="~/.openclaw/workspace/solana-wallet.json"
export SOLANA_AUTO_CREATE="true"
```

| Variable | Description | Default |
|---|---|---|
| `SOLANA_RPC_URL` | Solana RPC endpoint | `https://api.mainnet-beta.solana.com` |
| `SOLANA_WALLET_PATH` | Path to store the wallet keypair | `~/.openclaw/workspace/solana-wallet.json` |
| `SOLANA_AUTO_CREATE` | Automatically create a wallet if none exists | `false` |

## 3. Wallet Management

### Create a new wallet

```js
await tools.solana_wallet({ action: "create" })
```

### Check balances (SOL and tokens)

```js
await tools.solana_wallet({ action: "balance" })
```

### Get the wallet public key

```js
await tools.solana_wallet({ action: "address" })
```

On first run the plugin checks for an existing wallet file at `SOLANA_WALLET_PATH`.  
- If it exists, the existing keypair is reused.  
- If it does not exist and `SOLANA_AUTO_CREATE=true`, a new 64-byte keypair (compatible with the Solana CLI) is generated automatically.  
- If it does not exist and `SOLANA_AUTO_CREATE=false`, the plugin activation fails with an error.

> **Tip:** Back up your keypair file. Losing it means losing access to any funds in that wallet.

## 4. Token Swapping

Execute swaps via the Jupiter aggregator:

```js
await tools.solana_swap({
  inputToken: "USDC",
  outputToken: "SOL",
  amountUsd: 10
})
```

| Parameter | Type | Description |
|---|---|---|
| `inputToken` | `string` | Token to sell (symbol or mint address) |
| `outputToken` | `string` | Token to buy (symbol or mint address) |
| `amountUsd` | `number` | USD value to swap |

Supported token symbols include `SOL`, `USDC`, `USDT`, and any SPL token mint address.

## 5. Opportunity Scanning

Scan for trading opportunities across DexScreener and GeckoTerminal:

```js
await tools.solana_scan({ maxResults: 5 })
```

| Parameter | Type | Description |
|---|---|---|
| `chain` | `string` | Blockchain to scan (default: `"solana"`) |
| `maxResults` | `number` | Maximum number of results to return (default: `5`) |

Results are scored using momentum indicators (price change, volume, liquidity) to surface early-stage opportunities.

## 6. Autonomous Trading Monitor

The plugin ships a monitor script that runs a full momentum-based trading strategy automatically.

### Run manually

```bash
cd ~/.openclaw/workspace
node skills/solana-trader/scripts/monitor.js
```

### Schedule with cron (every 15 minutes)

```cron
*/15 * * * * cd ~/.openclaw/workspace && node skills/solana-trader/scripts/monitor.js
```

### Trading parameters

Configure risk management via environment variables:

```bash
export POSITION_SIZE_USD=10     # Maximum USD per trade
export MAX_POSITIONS=4          # Maximum concurrent open positions
export MIN_SCORE=25             # Minimum opportunity score to enter
export TAKE_PROFIT_PCT=50       # Exit at +50% gain
export STOP_LOSS_PCT=-25        # Exit at -25% loss
export TRAILING_STOP_PCT=15     # Trailing stop from peak price
```

### Exit rules

| Rule | Condition |
|---|---|
| Take profit | Position up ≥ `TAKE_PROFIT_PCT` |
| Stop loss | Position down ≤ `STOP_LOSS_PCT` |
| Trailing stop | Price retraces ≥ `TRAILING_STOP_PCT` from peak |
| Momentum death | Technical breakdown detected |

## 7. CLI Quick Reference

```bash
# Show wallet info
openclaw solana

# Interactive setup / reconfigure
openclaw configure
```

## 8. Plugin File Structure

```
~/.openclaw/workspace/
├── solana-wallet.json           # Wallet keypair (keep this secure)
└── skills/
    └── solana-trader/
        ├── SKILL.md             # Trading skill documentation
        ├── scripts/
        │   ├── monitor.js       # Autonomous trading monitor
        │   └── scan.js          # Opportunity scanner
        └── references/
            └── strategy.md     # Full strategy documentation
```

## Safety & Disclaimers

> ⚠️ **Trading cryptocurrencies involves substantial risk of loss.**
>
> - Start with small position sizes ($5–$20) to validate behaviour before scaling up.
> - Never risk more than you can afford to lose.
> - Keep your private key file (`solana-wallet.json`) secure and backed up offline.
> - This software is provided "as is" without warranty of any kind.

## Further Reading

- [OpenClaw Documentation](https://openclaw.com)
- [openclaw-solana-plugins on GitHub](https://github.com/solana-clawd/openclaw-solana-plugins)
- [Solana Documentation](https://docs.solana.com)
- [Jupiter Aggregator Documentation](https://docs.jup.ag)
