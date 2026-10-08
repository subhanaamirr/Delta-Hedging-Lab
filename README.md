# Delta-Hedging Lab

How well does Black-Scholes delta hedging work once you leave the textbook? I built a hedging simulator to test the theory piece by piece (discrete rebalancing, the wrong volatility, transaction costs), then ran the same strategy on three years of real BTC volatility and data from Deribit.

## Key results

| Question | Result |
|---|---|
| Does hedging error shrink like 1/√N? | Fitted slope **−0.487** (theory −0.5); Derman-Kamal formula within ~3% |
| Does the gamma P&L identity hold path by path? | Correlation **0.984** between simulated P&L and Σ ½ΓS²(σᵢ²Δt − (ΔS/S)²) |
| Optimal hedge frequency with 10bp costs (3-month option) | **~100 rebalances**; hedging more often costs more in fees than it removes in noise |
| Does Leland's cost-adjusted vol cover rebalancing costs? | Yes: mean P&L **0.002** after excluding the one-off set-up and unwind fees |
| BTC: is implied vol above realised? | Yes in **71%** of 153 trades; average premium **6.1 vol points** |
| BTC: weekly short 30-day ATM call, hedged daily | Average P&L **+0.75% of spot** per trade, worst trade **−4.7%** |
| BTC: does the edge survive the bid-ask spread? | Break-even at **6.6 vol points** below mid |

A quick sanity check on the BTC number: a 30-day ATM option has vega ≈ 0.4·√(30/365) ≈ 0.115% of spot per vol point. 6.1 points × 0.115 ≈ 0.70%, close to the 0.75% the backtest produced.

## What's in the notebook

1. **Black-Scholes core:** pricer, Greeks and implied vol, checked against put-call parity, finite differences and Monte Carlo before anything else runs.
2. **Hedging frequency:** hedging error vs number of rebalances, compared with Derman & Kamal.
3. **Volatility mismatch:** selling at 20% implied when realised comes in at 10–30%.
4. **Path dependence:** two paths with the same realised vol and very different P&L, because gamma is concentrated near the strike close to expiry.
5. **Hedging at implied vs realised vol:** realised locks in the final P&L, implied gives a smoother mark-to-market (Ahmad & Wilmott, 2005).
6. **Transaction costs:** the cost-optimal hedge frequency and Leland's (1985) adjusted volatility.
7. **BTC backtest:** every week, sell a 30-day ATM call priced at Deribit's DVOL index and delta-hedge it daily on BTC-PERPETUAL with 5bp per trade.

All charts and outputs are saved in the notebook, so you can read it on GitHub without running anything.

## Limitations

- **DVOL is not the ATM vol.** Like the VIX, it is built from options across many strikes and includes the skew, so it usually sits above at-the-money implied vol. Part of the measured premium is that gap, not edge.
- **The spread is modelled, not quoted.** The bid-ask test assumes a flat give-up in vol points rather than using historical Deribit quotes.
- **Overlapping trades.** Trades last 30 days but start every 7, so about four are open at once. The running-sum chart is not a portfolio return, and the trades are not independent observations.
- **No margin or funding model,** so there is no capital base and no Sharpe ratio.
- **The sample has no 2020- or 2022-style crash,** so the tail risk of short vol is probably understated.

## Run it

```bash
pip install numpy scipy pandas matplotlib requests jupyter
jupyter notebook delta_hedging_lab.ipynb
```

The simulations take about a minute. The first run downloads daily DVOL and BTC-PERPETUAL prices from Deribit's free public API (no account needed) and saves them to `data/`.

## References

- Derman, E. & Kamal, M. (1999). *When You Cannot Hedge Continuously: The Corrections to Black-Scholes.* Risk.
- Leland, H. (1985). *Option Pricing and Replication with Transactions Costs.* Journal of Finance.
- Ahmad, R. & Wilmott, P. (2005). *Which Free Lunch Would You Like Today, Sir?* Wilmott Magazine.
