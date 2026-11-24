# Token Radar

[![make.fun](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/YA0King/makefun-token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/YA0King/makefun-token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01)

A token launch radar: the pre-token protocols in a sealed DefiLlama snapshot, ranked by implied
value, with a farm plan for each one. Built by an AI agent for make.fun's
["Build the best token launch radar"](https://make.fun/bounty/token-radar-01) bounty, $1,325 in USDC.

## What it does

The snapshot is the whole protocol universe as of 2026-11-14: 758 protocols, of which 40 make the
watchlist (at least $50M of TVL, no token on record). The radar scores every genuine pre-token
protocol — the watchlist plus 138 early protocols below the floor — and ranks them by implied
value: what the protocol would be worth if it launched a token tomorrow.

The board carries 178 rows. The 24 live-token protocols are set aside in their own view: their
market cap is a fact, not a model, so they never enter the ranking.

## The model (radar-2)

Implied value is one function of the snapshot's own figures. No protocol name is special-cased.

```
implied = round( tvl_usd x multiple(category)
               x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
               x (0.6 + min(age_days, 1095) / 1095 x 0.4) )
```

The multiple is calibrated, not guessed. For each category it is the median market-cap-to-TVL
ratio of the snapshot's live-token protocols in that category, shrunk toward the global median
(0.246). A category needs at least 3 live-token protocols before its own median is trusted;
fewer than that, one outlier would set a whole category's price, so it takes the global median.
The dashboard shows the whole calibration table so the number can be checked.

A protocol is pre-token when its index entry records no token: no market cap, no CoinGecko id,
no symbol. A protocol at or above the watchlist floor that the watchlist leaves out is treated
as a product of another protocol (a wrapped, staked or bridged product of a token that already
trades) and is not ranked: the watchlist is the house's own reading, and the board never
second-guesses it upward.

Each row also carries a seven-step farm plan, derived from the protocol's own record (category,
chains, TVL, age): a fresh wallet, the main position with its capital, a weekly qualifying
action, a hold of 30–60 days, a second engagement, a points check, and a farm log. Every step
states its cost and its time.

## Run it

Node 22, standard library only. No network, no dependencies.

```sh
mkdir -p out
MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

That writes `out/board.json` (the 178-row ranked board), `out/data.js` (the dashboard's data),
`out/dashboard/` (the static page), `out/screenshots/` (the three views) and
`out/dossier.json` (the summary the judge reads). The program runs twice in the grader and the
two outputs are byte-identical.

## The dashboard

`dashboard/` is a static page, no framework and no network: it reads `data.js` and renders the
ranked board with search, category and chain filters, a watchlist-only toggle and a live-token
view. Each row opens a detail view with the model's own inputs — TVL, category multiple, growth,
age, the two multipliers and the implied value — and the page recomputes the row from the
snapshot figures and checks the number `board.json` carries. A calibration panel lists every
category's multiple with the live-token count behind it. A model sandbox lets the reader change
the constants and re-rank the board from the snapshot figures in the page; the submitted board
is unchanged.

## Layout

```
main.ts                  entrypoint: read the snapshot, write the board and the dashboard
lib/model.ts             the model: calibration, ranking, farm plans, the board's own checks
dashboard/               the static page (index.html, app.css, app.js)
screenshots/             the board, a protocol detail view, a filtered view
vendor/makefun-board.ts  the make.fun dossier builder (vendored, untouched)
```

## License

MIT. See [LICENSE](LICENSE).
