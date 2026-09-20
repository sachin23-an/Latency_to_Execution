# Latency to Execution

Can an order-flow signal still make money after latency, queue position, adverse selection and fees?

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_GITHUB_USERNAME/latency-to-execution/blob/main/Latency_to_Execution.ipynb)

A short-horizon order-flow signal can predict the next price move and still lose money when you trade it. By the time the order reaches the exchange the price may have moved. A passive order gets filled mostly when the price is about to go against it. This project measures how much of the edge is left after those things, on BTCUSDT perpetual futures data from Binance.

## Questions

1. How fast does the information in order flow fade?
2. How much of the edge is left after some milliseconds of latency?
3. How much do the results depend on the queue (fill) assumption?
4. How much of a passive market maker's profit is lost to adverse selection?
5. Does a market maker that uses the signal do better than a naive one?
6. At what latency and fee does everything stop making money?

## What the notebook does

- Builds six order-book signals (OFI, trade-flow imbalance, queue imbalance, micro-price) and combines them into a prediction with a walk-forward ridge model.
- Measures how much of that edge is left when the order arrives later (taker view).
- Replays the market event by event with a Numba engine. Strategies see old data and their orders arrive late.
- Uses three queue assumptions for passive fills: conservative, proportional and optimistic.
- Compares two market makers that differ by one rule. B pulls its ask when the signal says the price will rise, and its bid when it says the price will fall.
- Measures markouts 1, 5 and 10 seconds after each fill, with bootstrap confidence intervals.
- Finds the break-even maker fee and break-even latency for each strategy and fill assumption.
- Splits results by market conditions and checks them on hold-out days.

## Checks in the notebook

- Signals are recomputed with all later data deleted and must not change.
- Every model is fitted on earlier days than the day it scores.
- Trading thresholds come from training data only.
- Nine hand-made test markets check the replay engine (queue position, trades through our price, latency, post-only, inventory limit).
- The alpha used at decision time is built from data that is old enough.
- Days are split into warm-up, development and hold-out days. The hold-out days are only used at the end.

## Data

Binance public archive, USD-M futures: `bookTicker` (best bid and ask) and `aggTrades`. This only has the top of the book, so the queue models are assumptions and the OFI is top-of-book OFI. The default dates are 2024-02-05 to 2024-02-12, 3 hours from 09:00 UTC each day.

BTCUSDT is a stand-in for studying the mechanics. Nothing here is assumed to hold for NSE, where tick size, fees and participants are different. Full NSE depth history is not free. If you have top-of-book and trade data for Nifty futures, set `DATA_MODE = "custom"` and follow the csv format in `load_custom_day`.

## How to run

1. Open `Latency_to_Execution.ipynb` in Google Colab.
2. Runtime > Run all. The first run downloads and reads the daily files (about 9 minutes for six days in my run). Later runs use the cache.
3. Put your own fee tier in `MAKER_FEE_BPS` and `TAKER_FEE_BPS` before reading any profit number.
4. Figures are saved in `figures/` and tables in `results/`. `results/results_for_readme.md` has the numbers from your run.

## Results

Paste the contents of `results/results_for_readme.md` here after a full run on real data, and add a few sentences in your own words. Do not paste numbers from a run with `DATA_MODE = "synthetic"`.

Things worth reporting: where the signal's IC peaks and how fast it fades, how much gross edge is left at 10, 100 and 500 ms, how much the fill assumption moves the profit, the markout difference between B and A with its interval, whether the hold-out days agree with the development days, and the break-even fee compared with the tick and the fees you would really pay.

## Limits

- Only the best bid and ask are visible, so true queue position is unknown. The fill model cannot be checked with public data. A real check would compare simulated fills with a few small real orders.
- Latency is a fixed number here. In real life it changes from order to order.
- One instrument and a few short sessions.
- Own fills are assumed to be known instantly, orders do not move the market, and hidden orders are ignored.

## Files

```
latency-to-execution/
  Latency_to_Execution.ipynb
  README.md
  figures/     (made by the notebook)
  results/     (made by the notebook)
```

## References

- Cont, Kukanov, Stoikov (2014). The Price Impact of Order Book Events.
- Stoikov (2018). The micro-price: a high-frequency estimator of future prices.
- Avellaneda, Stoikov (2008). High-frequency trading in a limit order book.

Research code, not investment advice.
