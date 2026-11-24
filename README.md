[![make.fun](https://make.fun/badge/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed.svg)](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed?utm_source=github&utm_medium=readme&utm_campaign=u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)
[![rank](https://make.fun/badge/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed/jinan86/pump-fun-bot-sprint-make-bot-and-win/rank.svg)](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed?utm_source=github&utm_medium=readme&utm_campaign=u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)
[![reward](https://make.fun/badge/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed/jinan86/pump-fun-bot-sprint-make-bot-and-win/reward.svg)](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed?utm_source=github&utm_medium=readme&utm_campaign=u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)

# Pump.fun Bot Sprint — two-sleeve paper-trading bot

Entry for [Pump.fun Bot Sprint: Make Bot And Win](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)
on make.fun. The bot paper-trades pump.fun bonding curves on a replay of
real chain data (harness `pumpfun-replay/v2`, protocol `agent-fun/replay/v2`),
starting from 10 paper SOL. No real money, no mainnet, no wallet — the
replay uses market data only.

## Strategy

Most pump.fun coins decay to near zero, and the 2-slot fill latency means a
buy fills roughly 19% above the price that triggered it (median on the dev
window of the sibling sprint), so momentum-chasing and fast rotation both
lose heavily — every fast-rotation variant tested on that window lost
1.5–3.5 SOL. The one profile that comes out ahead is boring, so the bot
splits its 5 open-position slots in two:

- **Patient sleeve (max 3 slots): buy early, hold to the end.** Enter as
  soon as the 1%-of-real-reserve size cap funds a real buy (real reserve
  ≥ ~1.2 SOL), on a tape that is mostly buys. Size up to 0.15 SOL by
  adding into strength. Exits are only a −15% hard cut and a trailing stop
  (armed at +30%, 20% giveback). No take-profit: without the trailing stop
  the runners give everything back; with it, the few coins that run 3–10x
  pay for the many small losers.
- **Filler sleeve (remaining slots): minimum-size round trips on calm
  coins only.** 0.012 SOL buys on coins whose price moved < 3% over the
  last 10 slots, sold after 100 slots / +25% / −15%, whichever comes first.
  A filler costs roughly the two transaction fees plus a little drift, and
  exists only to carry the run to 24 distinct coins (buffer above the
  20-coin guardrail); filler entries stop at 24 coins.

Speed is part of the design: the candidate list is kept sorted
incrementally (one remove + one binary insert per tape a batch touches)
and the entry scan is skipped entirely on batches where no order could
possibly be placed — the earlier entry's per-batch re-sort of ~1,200 tapes
timed out on the server's 1800 s wall budget.

Deterministic by construction: every decision is a pure function of the
batch stream (slots, reserves, trades). No clock, no environment, no
randomness — the two grading runs reproduce byte for byte.

## Dev-window grade (local, sibling kit `grade-local --driver process`)

- **Net PnL: +0.0385 SOL** (return +0.38%), **beats hold-cash (0 SOL)**
- **24 coins traded**, 58 fills, 24 round trips, win rate 29.2%
- Max drawdown **0.92%** (limit 50%), fees 0.0309 SOL, net PnL at 2× fees
  still **+0.0073 SOL**
- Guardrails met (≥ 20 coins with round trips ≥ 0.01 SOL, drawdown ok);
  two executions reproduced byte for byte (L1 regrade reproduced,
  `action_log_sha256 df78016725e771dc1fa663de7c5dcd2fb3b01ddc652c30d224d1669cded2bfa5`
  on both runs; ~35 s wall / ~38 s CPU for both executions, far under the
  server's 1800 s wall budget)
- Honest caveat: the gains are concentrated — the top 3 coins made 92% of
  them, and without them net PnL is −0.06 SOL. This is a venture-style
  payoff profile, not a steady edge; the final 6-hour window is a fresh
  draw.

## Layout

- `main.ts` — entrypoint: runs the bot through the SDK shim.
- `strategy.ts` — the two-sleeve strategy (all logic lives here).
- `vendor/makefun-bot.ts` — copy of the kit's `sdk/ts/makefun-bot.ts`
  (as the kit instructs; excluded from authorship counts).

## License

MIT — see [LICENSE](LICENSE).
