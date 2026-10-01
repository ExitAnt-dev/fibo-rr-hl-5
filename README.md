# ExitAnt Fibo R-R — Hyperliquid

Deploy this automated trading bot to **your own Render account** as a background
worker. The bot runs from a prebuilt Docker image; this repository contains **only
the Render Blueprint (`render.yaml`)** — no source code.

## Deploy to Render

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/ExitAnt-dev/fibo-rr-hl-5)

```
https://render.com/deploy?repo=https://github.com/ExitAnt-dev/fibo-rr-hl-5
```

1. Click the button (or open the link) and sign in to Render.
2. Render reads `render.yaml` and proposes **one background worker** with a
   **1 GB persistent disk** mounted at `/data` (settings and positions survive restarts and redeploys).
3. Enter the environment variables listed below. They are marked `sync: false`, so Render asks you for
   them on the deploy screen and stores them **only in your own Render account**.
4. Click **Apply** and wait for the first deploy to finish.
5. Open Telegram and send **`/start`** to your bot to begin setup.

## Before you deploy

- **A wallet used only by this bot**, with USDC deposited to Hyperliquid (perps balance). Do not run another bot
  or trade by hand on the same wallet: Hyperliquid keeps one position per coin, so the positions would mix.
- **Account mode: Standard.** The bot does not start on a Unified or Portfolio Margin account
  (Hyperliquid → account settings).
- **Builder fee approval.** At the first start the bot approves the ExitAnt builder fee for this wallet
  (up to 0.05% per trade, charged by Hyperliquid on each fill) and then confirms it with the ExitAnt server —
  this is the membership check. Only the wallet's public address is sent, never the key.
- The bot trades the main Hyperliquid perps (crypto) and the trade.xyz perps (US stocks, indices, gold, silver)
  that allow cross margin. It moves USDC between those two balances **inside your own account** as positions need it.
- *Optional — the cash vault* keeps part of the profits in your **spot** USDC balance of the same account.

## Environment variables

| Variable | What to enter |
|---|---|
| `HYPE_WALLET_API` | The **private key** of the trading wallet (`0x` + 64 hex characters) — not the wallet address |
| `TELEGRAM_BOT_TOKEN` | Bot token issued by [@BotFather](https://t.me/BotFather) |
| `TELEGRAM_CHAT_ID` | Your own Telegram chat ID — the bot ignores every other sender |

Nothing else to configure: the exchange, trading mode and state-file location are built into the image.

## What the bot does

Fibo R-R waits for a Bollinger Band breakout on the 5- or 15-minute chart, then enters in up to six steps
(levels) at the spacing you configure, takes profit per level at a Fibonacci ratio towards the opposite
band breakout, keeps one exchange stop-loss for the whole position at your loss limit, and resets the
ladder on an opposite breakout, after a time limit or by hand. A margin guard trims the position when used
margin exceeds your threshold. Every parameter is set in Telegram; twelve languages are supported.
On Hyperliquid a symbol runs long **or** short (one position per coin), and amounts are in USDC.

## How it works

- **Prebuilt image.** The worker pulls `docker.io/exitant/exitant-fibo-rr-hl:5` from Docker Hub. The image ships
  a compiled binary only — no Python source is included or exposed.
- **Background worker.** There is no web page; the bot runs continuously and talks to you through Telegram.
- **Your key stays yours.** The wallet key is typed into your own Render environment, signs orders inside the
  bot and is never sent anywhere.
- **Persistent state.** Settings, positions and pending orders live in a small database on the `/data` disk.

## Important

- **Keep the instance count at 1.** Two instances cause a Telegram `getUpdates` conflict.
- Render **background workers have no free plan** — choose Starter or higher.
- **The bot trades live from the first start.** Begin with a small allocation and watch one full cycle
  (breakout → entries → take-profit/stop-loss) on your account before scaling up.
- The smallest order on Hyperliquid is 10 USDC; the setup only offers sizes that reach it.
- After the deploy succeeds, send **`/start`** in Telegram to configure the bot.

---

Blueprint spec: <https://render.com/docs/blueprint-spec> · Prebuilt images: <https://render.com/docs/deploying-an-image>
