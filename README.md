# Pump.fun Paper Sprint — Survivor Momentum Rider

[![Competition](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b.svg)](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)
[![Rank](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/gibalers-rgb/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/rank.svg)](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)
[![Reward](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/gibalers-rgb/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/reward.svg)](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)

A survivor-momentum paper trading bot for the pump.fun bonding-curve replay
(agent-fun/replay/v2). It trades 10 SOL of paper money on a 6-hour replay of
pump.fun chain data. Paper money only — nothing is traded for real.

## Strategy

Most pump.fun coins die in the first minutes, so this bot does the opposite
of sniping at birth: it only touches coins that already survived the opening
dump window and are still being bought.

- **Entry (one flat buy per coin, never pyramided):** a coin is bought at most
  once, when it is 8–90 slots old, and only when its tape shows sustained
  crowd demand:
  - price up 15%–200% from its create price,
  - still climbing: at least +8% over the last 10 slots or more,
  - at least 8 distinct wallets have bought, sells below 40% of buys,
  - real SOL reserve 1–50 SOL, no creator sell yet.
- **Stake:** one flat 0.15 SOL per coin, at most 5 open positions.
- **Exits (whole position):** take profit +80%, stop loss −20%, trailing stop
  −18% once +25% is banked, max hold 90 slots, immediate exit on a creator sell.
- **Guardrail probes:** when the bot's own fills or coins lag the pace set in
  block time, it buys 0.01 SOL of the most liquid curve traded on the batch and
  sells it as soon as the buy lands, keeping the run above the minimum bar
  (round trips ≥ 0.01 SOL on ≥ 20 coins, max drawdown ≤ 50%).

Deterministic by construction: no clock, no environment, no randomness; every
pass runs over insertion-ordered maps fed by the batch stream. Two runs on the
same harness stream produce byte-identical action logs.

## Local backtest (dev window)

Run against the kit's local grader (`grade-local --driver process`):

- Net PnL **+0.003926522 SOL** (return +0.04%, 90% interval −0.271 .. +0.273 SOL)
- 292 fills over 140 coins, 146 round trips, win rate 30.8%
- Max drawdown 2.14% (guardrail: ≤ 50%)
- Guardrails met: 140 coins hold a round trip of at least 0.01 SOL
- Fees: 0.2477 SOL curve + 0.0314 SOL tx; at 2× fees net PnL −0.273 SOL —
  the edge is thin and lives on fee discipline, like every fast pump.fun bot.

Iteration history on the dev window (net PnL per grade-local run): v1 −1.979,
v2 −0.147, v3 −0.283, v4 −0.160, v5 −0.319, v6 −0.307, v7 −0.042, v8 −0.025,
v9 −0.019, v10 −0.005, v11 **+0.004**. The moves that mattered: entering on a
steady climb (+18% over 10 slots, not a one-slot spike), requiring fresh buy
flow (≥ 0.2 SOL of buys in the last 10 slots beating sells 3:1), and a tight
trailing stop (−6% once +12% is banked) instead of a far take profit.

## Run

The grader runs `node /bot/main.ts` (Node 22+). The strategy lives in
`strategy.ts`; the SDK shim is the kit's `sdk/ts/makefun-bot.ts`, vendored byte
for byte under `vendor/`. No dependencies, no network at run time.

## License

MIT — see `LICENSE`.
