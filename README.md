# ⚡ FIRSTBLOOD — Jito Bundle Backrunner for pump.fun V3

Solana bot that detects new pump.fun V3 token launches via WebSocket log subscriptions, validates the dev buy, and backruns with a Jito bundle to land in the same block.

## How it works

1. **Subscribe** — connects to Helius WebSocket, subscribes to `logsSubscribe` for the pump.fun V3 program
2. **Detect** — classifies log messages to find `BondingCurveV3` / `BondingCurveJupiter` creates
3. **Parse** — extracts mint address, SOL amount, reserves from the trade event data
4. **Filter** — checks minimum dev buy, bonding curve state, position limits, cooldown
5. **Buy** — builds a BuyV2 transaction with ATA creation + Jito tip
6. **Bundle** — sends the signed transaction as a Jito bundle via the Block Engine API
7. **Monitor** — tracks positions, exits on take-profit / stop-loss / timeout
8. **Sell** — builds and sends a SellV2 transaction as a Jito bundle

## Setup

```bash
# Clone
git clone https://github.com/zostaff/firstblood.git
cd firstblood

# Install
npm install

# Configure
cp .env.example .env
# Edit .env with your keys
```

### Required environment variables

| Variable | Description | Default |
|---|---|---|
| `RPC_WSS` | Helius WebSocket endpoint | — |
| `RPC_URL` | Helius HTTP endpoint | — |
| `HELIUS_API_KEY` | Helius API key | — |
| `WALLET_PRIVATE_KEY` | Base58 private key | — |
| `BACKRUN_BUY_SOL` | SOL amount per buy | `0.1` |
| `BACKRUN_JITO_TIP` | Jito tip per bundle | `0.001` |
| `BACKRUN_TAKE_PROFIT` | Take profit multiplier | `2.0` |
| `BACKRUN_STOP_LOSS` | Stop loss multiplier | `0.5` |
| `BACKRUN_TIMEOUT_SEC` | Position timeout | `60` |
| `BACKRUN_MIN_DEV_BUY` | Minimum dev buy in SOL | `0.5` |
| `BACKRUN_MAX_POSITIONS` | Max concurrent positions | `3` |
| `BACKRUN_COOLDOWN_SEC` | Cooldown between buys | `5` |

## Usage

```bash
# Run live
npm start

# Dry run (no real transactions)
npm run dry
# or
node bot.js --dry-run
```

## Architecture

```
bot.js              — main entry, orchestration
lib/
  config.js         — env → config object
  constants.js      — program IDs, tip accounts, discriminators
  logger.js         — colored console output
  listener.js       — WebSocket logsSubscribe
  parser.js         — log classification + trade event parsing
  pda.js            — PDA derivation (bonding curve, ATA)
  bonding.js        — bonding curve state + math
  tx-builder.js     — buy/sell transaction construction
  jito.js           — Jito Block Engine bundle submission
  positions.js      — position tracking + TP/SL/timeout
```

## Risk Warning

This is experimental software for educational purposes. Trading memecoins is extremely risky. You can lose your entire investment. Use at your own risk.
