# Introduction to Backtesting: Testing Ideas Before Risking Money

*By Vizanix — Beginner Level*

> This book shows you how to test a trading idea against historical data honestly, and how to avoid the traps that make a bad strategy look good on paper.

![diagram](../../assets/pnl-curve.svg)

## Table of Contents

1. What Backtesting Actually Tests
2. The Anatomy of a Backtest
3. Look-Ahead Bias: The Silent Killer
4. Overfitting: When You Fit the Noise, Not the Signal
5. Modeling Real-World Costs
6. Reading Backtest Results Honestly
7. From Backtest to Paper Trading
8. What a Good Backtest Cannot Tell You

## 1. What Backtesting Actually Tests

A backtest runs your trading strategy's rules against historical market data to see what would have happened if you'd traded that way in the past. You feed it a history of prices, it applies your buy and sell logic step by step through that history, and it produces a simulated record of trades, profits, and losses.

It's tempting to treat a backtest as a rehearsal of the future. That's not quite what it does, though. It tests whether your specific rules, applied mechanically to a specific slice of the past, would have produced a specific outcome. Whether that past pattern continues into the future is a separate question a backtest cannot answer by itself. What a good backtest gives you is a disciplined way to reject clearly bad ideas cheaply, and to build calibrated confidence, never certainty, in ideas that survive careful scrutiny.

Think of a backtest as similar to a flight simulator for a pilot. It doesn't guarantee a real flight goes perfectly, but it lets you practice, catch obvious mistakes, and build familiarity with how the aircraft, or in this case the strategy, behaves under various conditions, all without the cost of a real crash.

## 2. The Anatomy of a Backtest

Every backtest needs four ingredients. The first is historical data: the prices, and possibly volume or order book information, that your strategy will run against, cleaned according to the practices covered earlier in this library. The second is a strategy definition: the exact, mechanical rules for when to buy, sell, or hold, expressed precisely enough that a computer can execute them without ambiguity.

The third is a simulation engine: the code that walks through the historical data step by step, in the correct chronological order, applies your strategy's rules at each step, and tracks the resulting simulated positions and cash. The fourth is a set of assumptions about costs and execution, covering how much slippage to assume, what commission to charge per trade, and how quickly an order is assumed to fill after your strategy decides to trade.

The engine typically works one time step at a time: look at the data available up to and including this moment, ask the strategy whether it wants to act, simulate the resulting trade if so, then move to the next time step and repeat. This step-by-step discipline, never letting the strategy see data from the future relative to its current simulated moment, is the single most important property a correct backtest engine must have. It's exactly what the next section addresses in detail.

At the end of the run, you get a record of every simulated trade along with a time series of your simulated portfolio's value, from which you calculate performance measures covered later in this book.

## 3. Look-Ahead Bias: The Silent Killer

Look-ahead bias happens when your backtest accidentally lets the strategy use information that wouldn't have actually been available at that point in real time. It's the single most common way a backtest lies to you, and it's often invisible unless you look for it specifically.

A classic example: computing a day's average price using that day's own closing price, then using that average to decide whether to buy earlier that same day. In live trading, you wouldn't know the day's closing price until the day was already over, so a rule based on it couldn't have actually been executed intraday. But a carelessly written backtest, especially one working with whole rows of data rather than strictly time-ordered steps, can easily make this mistake without raising any error.

Another subtle version involves data revisions. Some data providers restate historical values after the fact, correcting errors or reflecting later adjustments. If your backtest uses the revised version of a data point that wouldn't have looked that way in real time, you're testing against information from the future disguised as the past.

Survivorship bias is a related trap. If your historical dataset only includes instruments that still exist today, you've silently excluded everything that went bankrupt, got delisted, or otherwise disappeared, which biases your results toward "survivors" and inflates apparent performance. Guard against these traps by explicitly asking, for every input your strategy uses, "would a trader actually have known this value at this exact moment in real time?" If the honest answer is no, that input doesn't belong in your backtest.

## 4. Overfitting: When You Fit the Noise, Not the Signal

Overfitting happens when you tune a strategy's rules so closely to a specific historical dataset that it captures the random noise in that particular data rather than any real, repeatable pattern. An overfit strategy looks spectacular in its backtest and performs poorly, often close to random, once it meets new data it wasn't tuned against.

The risk grows every time you adjust a parameter, add a rule, or try a variation and re-check the backtest result. If you test a hundred slightly different versions of a strategy against the same historical period and pick whichever one performed best, you've likely just found the version that happened to fit that period's particular noise most closely, not the version with the strongest genuine edge.

A practical defense is splitting your historical data into separate periods: one for developing and tuning your strategy (in-sample data), and one you don't touch until you're finished tuning, used only once at the very end to check performance (out-of-sample data). If a strategy performs dramatically worse on out-of-sample data than on the data you tuned it against, that gap is a strong warning sign of overfitting.

A related discipline is preferring simple strategies with few parameters over complex ones with many. Every additional parameter you tune gives overfitting another way to sneak in. A strategy you can describe in one or two sentences, with one or two parameters, is far easier to reason about honestly than one with a dozen interacting thresholds.

## 5. Modeling Real-World Costs

A backtest that ignores trading costs is measuring a strategy that doesn't exist. Every real trade crosses some version of the bid-ask spread covered earlier in this library, and many markets also charge an explicit commission or fee per trade.

Slippage, the gap between the price your strategy expected and the price it would actually receive, deserves careful, honest modeling. A simple but reasonable starting approach assumes you always pay slightly worse than the last known price, roughly matching the spread, rather than assuming you always get the exact closing price you see in your bar data, which no real market order ever guarantees you.

Strategies that trade frequently are especially sensitive to cost assumptions, since costs compound with every trade. A strategy showing a healthy profit before costs can easily turn unprofitable once realistic costs are subtracted, particularly if it trades often on a modest average profit per trade. Always run your backtest with cost assumptions included from the start, rather than as an afterthought applied only once you already like the results, since that ordering subtly biases you toward trusting rosier, cost-free numbers first.

## 6. Reading Backtest Results Honestly

A backtest typically reports a handful of summary numbers: total return, the overall percentage gain or loss over the tested period; maximum drawdown, the largest peak-to-trough decline your simulated portfolio experienced along the way; and some measure of risk-adjusted return, which weighs your profit against how much your portfolio value bounced around to get there.

Look at the full equity curve, the chart of simulated portfolio value over time, not just the final summary numbers. A strategy that ends with an attractive total return might have gotten there through one lucky period concentrated in a small slice of your test, with the rest of the time essentially flat or losing. That tells a very different story than a smooth, steadily rising curve.

Pay close attention to drawdown and how long recovery from it took. A strategy with an excellent average return but occasional brutal drawdowns might be mathematically profitable over a long enough horizon while being practically unbearable to actually hold through, since real capital and real nerves have limits that a spreadsheet doesn't.

Finally, check how many independent trades your backtest actually contains. A handful of trades cannot reliably demonstrate anything statistically, no matter how good they look, in the same way flipping a coin three times and getting three heads tells you nothing reliable about whether the coin is fair.

## 7. From Backtest to Paper Trading

A promising backtest earns you the next step, not immediate live trading with real money. Paper trading runs your strategy against live, real-time market data, generating simulated trades exactly as it would in reality, but without committing actual capital.

Paper trading catches a different category of problems than backtesting does: bugs in how your system connects to live data feeds, timing issues that only appear when data arrives continuously rather than being read instantly from a file, and the psychological experience of watching a strategy operate in real time, which is genuinely different from reviewing a finished historical result.

Run a strategy in paper trading for long enough to see a reasonable number of trade opportunities, not just a day or two, before considering real capital. Compare its paper trading behavior directly against what your backtest predicted for the same live period, since a large, unexplained gap between the two is a sign something in your backtest or your live implementation doesn't match reality.

## 8. What a Good Backtest Cannot Tell You

Even a rigorous, honest backtest, free of look-ahead bias, carefully checked for overfitting, and modeling costs realistically, cannot promise future performance. Markets change: participant behavior shifts, liquidity conditions evolve, and a pattern that was genuinely real and exploitable for years can fade once enough other participants notice and trade against it.

A backtest also can't fully capture your own behavior as a human operating the system: whether you'll actually follow the strategy's signals during a stressful drawdown, or override it based on a news headline that wasn't part of your tested rules. Building in disciplined rules for exactly when you will and won't intervene manually is part of preparing a strategy for real deployment, and no backtest number substitutes for that preparation.

Treat a good backtest as a rigorous first filter, a way to reject weak ideas cheaply and build calibrated, appropriately humble confidence in the ones that survive, rather than as a promise of future results.

## Summary

- A backtest tests whether specific rules, applied mechanically to historical data, would have produced a given outcome; it doesn't guarantee the future.
- A correct backtest engine processes data strictly in chronological order, never letting the strategy see future information.
- Overfitting turns a strategy into a noise-fitter; guard against it with out-of-sample testing and simple, low-parameter designs.
- Realistic slippage and commission assumptions belong in the backtest from the start, not bolted on afterward.
- A promising backtest justifies paper trading next, not immediate live capital, and no backtest guarantees future results.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
