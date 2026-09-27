# Time-Series Analysis for Trading Systems

*By Vizanix — Intermediate Level*

> Build the statistical intuition and practical toolkit for treating price and volume series honestly, so your signals survive contact with real, noisy, non-stationary markets.

![diagram](../../assets/pnl-curve.svg)

## Table of Contents

1. Why Financial Time Series Break Textbook Assumptions
2. Stationarity and Why You Should Care
3. Autocorrelation and Mean Reversion
4. Volatility Clustering and Estimating Variance
5. Cointegration and Pairs Relationships
6. Resampling and the Bar Construction Problem
7. Detecting Regime Shifts
8. Pitfalls: Spurious Correlation and Overfitting to Noise

## 1. Why Financial Time Series Break Textbook Assumptions

Most introductory time-series methods assume your data is stationary, identically distributed, and free of structural breaks. Financial data violates all three routinely. Volatility itself changes over time in clusters, calm stretches followed by turbulent ones. Correlations between assets shift with the macro regime. A statistical relationship that held for three years can break the week a company's fundamentals change or a market structure rule shifts.

That doesn't make statistics useless for trading. It means you need methods that are robust to these violations, and discipline about testing whether your assumptions actually hold for the data you're using, rather than assuming a textbook method transfers cleanly. This chapter builds that toolkit at a practical level: what to check, how to check it, and what to do when the checks fail.

A useful habit to develop early is treating every statistical assumption as a testable claim about your specific dataset rather than a property you inherit for free from the method's textbook description. If a technique's derivation assumes independent and identically distributed observations, and you're applying it to a return series with known volatility clustering, you haven't automatically disqualified the technique. But you have taken on an obligation to check whether the violation is severe enough to actually distort your conclusions in a way that matters for the decision you're using the result to make. This chapter's tools exist to help you make that judgment concretely rather than by hand-waving intuition.

## 2. Stationarity and Why You Should Care

A stationary series has statistical properties, mean, variance, autocorrelation structure, that don't change over time. Raw asset prices are almost never stationary: a price series trending from $50 to $150 over a year has a mean that clearly depends on when you measure it. Returns (percentage or log changes) are much closer to stationary, which is why nearly every quantitative model works with returns rather than raw prices.

The Augmented Dickey-Fuller test is the standard tool for checking whether a series has a unit root, a hallmark of non-stationarity, random-walk-like behavior where shocks persist indefinitely rather than decaying. In practice:

```
from statsmodels.tsa.stattools import adfuller

result = adfuller(returns_series, autolag='AIC')
p_value = result[1]
# p_value < 0.05 suggests you can reject the null of a unit root
# i.e., the series is more likely stationary
```

Don't treat this test as a binary gate that clears your model for production. A series can pass an ADF test on one historical window and clearly fail on another, especially around regime changes. Run it on rolling windows, not just the full sample, and watch how the result evolves. If it flips frequently, your "stationary" series is stationary only in a fragile, sample-dependent sense, and any model assuming otherwise deserves extra skepticism.

The KPSS test complements the ADF test usefully because it tests the opposite null hypothesis, stationarity rather than a unit root, and disagreement between the two tests on the same data is itself informative. If ADF rejects a unit root (suggesting stationarity) and KPSS also fails to reject stationarity, you have reasonably strong, convergent evidence. If the two tests disagree, that usually signals a series with some intermediate behavior, like a slow-moving trend or a structural break partway through your sample, that neither test alone is well suited to characterize cleanly. You're better served visualizing the series directly and reasoning about the specific historical events that might explain the ambiguity than trusting a single automated test's verdict.

## 3. Autocorrelation and Mean Reversion

Autocorrelation measures how correlated a series is with a lagged version of itself. For returns, meaningfully nonzero autocorrelation at short lags is one of the most direct statistical footholds for a trading signal: positive autocorrelation suggests momentum (recent moves tend to continue), negative autocorrelation suggests mean reversion (recent moves tend to partially reverse).

```
def autocorrelation(series, lag):
    n = len(series)
    mean = sum(series) / n
    numerator = sum((series[t] - mean) * (series[t - lag] - mean)
                     for t in range(lag, n))
    denominator = sum((x - mean) ** 2 for x in series)
    return numerator / denominator
```

In liquid, well-arbitraged markets, raw return autocorrelation at short lags is typically small in magnitude and unstable, which should make you suspicious of any backtest that shows a large, clean autocorrelation-based edge with no transaction cost erosion.

Microstructure effects can also produce autocorrelation that has nothing to do with genuine predictability. Bid-ask bounce, prices oscillating between the bid and ask as trades alternate between buyer- and seller-initiated, introduces artificial negative autocorrelation into a raw trade-price series that vanishes once you look at mid-price rather than last-trade-price returns. Before concluding a mean-reversion signal is real and tradeable, verify it survives this substitution. A signal that only shows up in trade-price returns and disappears in mid-price returns is very likely an artifact of market mechanics rather than a genuine, exploitable pattern in the underlying valuation process. The more common practical use of autocorrelation analysis is diagnostic: checking whether your model's residuals still show autocorrelation, which would indicate your model is leaving exploitable structure on the table, or conversely that your position-sizing logic is introducing unwanted serial correlation in your P&L that increases risk without adding return.

## 4. Volatility Clustering and Estimating Variance

Volatility clustering, the empirical observation that large price moves tend to be followed by more large moves, and calm periods by more calm periods, is one of the most robust patterns in financial data, far more reliable than any specific directional signal. This has direct engineering consequences: a position sizing system using a stale, long-window volatility estimate will oversize positions right as volatility is spiking and undersize them right as it's calming down. Both bad.

A simple, robust way to estimate current volatility is an exponentially weighted moving average of squared returns, which naturally weights recent observations more heavily without requiring a hard window cutoff:

```
def ewma_variance(returns, lam=0.94):
    var = returns[0] ** 2
    variances = [var]
    for r in returns[1:]:
        var = lam * var + (1 - lam) * r ** 2
        variances.append(var)
    return variances
```

The decay parameter `lam` controls the effective memory of the estimate. Closer to 1 means slower adaptation, closer to 0 means faster reaction but noisier estimates. For systems that need to size positions or set risk limits in near real time, this kind of adaptive estimator is far more useful than a fixed-window historical standard deviation recomputed once a day.

GARCH-family models extend this idea by explicitly modeling how today's variance depends on both yesterday's variance and yesterday's squared return, capturing the clustering dynamic with a small number of estimated parameters rather than a fixed decay constant chosen by hand. The added complexity earns its place primarily when you need forward-looking variance forecasts over a specific horizon, useful for options-adjacent risk work, rather than just a smoothed estimate of current variance, where the simpler EWMA approach is usually adequate and considerably easier to implement, monitor, and explain to a risk committee unfamiliar with the details of a fitted GARCH specification.

## 5. Cointegration and Pairs Relationships

Two assets can each be individually non-stationary (both trending randomly) while some linear combination of them is stationary. This is cointegration, and it's the statistical basis for most pairs-trading and statistical arbitrage strategies. Intuitively: if two stocks are cointegrated, their price spread, appropriately weighted, tends to revert to some equilibrium level even though each individual price wanders without bound.

The Engle-Granger approach is the simplest starting point: regress one series on the other to estimate the hedge ratio, then test the residual spread for stationarity using the same kind of unit-root test discussed earlier.

```
import numpy as np
from statsmodels.tsa.stattools import adfuller

hedge_ratio = np.polyfit(price_b, price_a, 1)[0]
spread = price_a - hedge_ratio * price_b
p_value = adfuller(spread)[1]
# low p_value suggests the spread is mean-reverting -> candidate pair
```

The practical trap here is testing many candidate pairs and only reporting the ones that pass. With enough pairs tested, some will show spurious cointegration by pure chance. Correct for multiple testing, and more importantly, require an economic or structural rationale for why two assets should be related (same sector, same underlying commodity exposure, an index-and-component relationship) before you even run the statistical test, rather than mining an entire universe blindly.

Even a genuinely cointegrated pair with a solid economic rationale is not a static, permanent relationship. The hedge ratio itself can drift over time as the underlying businesses or exposures evolve, and a pairs strategy using a hedge ratio estimated once at the start and never revisited will accumulate a growing, unhedged directional exposure as that ratio drifts away from its estimated value. Reestimate the hedge ratio on a rolling basis, and monitor the spread's mean-reversion speed over time as an early warning indicator. A spread that is reverting noticeably more slowly than its historical norm is a sign the underlying relationship may be weakening before it fully breaks down, giving you a chance to reduce exposure ahead of an outright cointegration failure rather than discovering it only after the strategy has already lost money on a broken relationship.

## 6. Resampling and the Bar Construction Problem

Most trading systems work with "bars," OHLCV summaries over some interval, rather than raw tick data, for tractability. The default choice, time bars (one bar per fixed clock interval), has a subtle statistical drawback: trading activity is not uniform in time, so a time bar during a quiet period represents very little information while a time bar during a burst of activity compresses a huge amount of information into the same-sized bucket.

Volume bars (a new bar every time a fixed quantity of shares trades) and dollar bars (a new bar every time a fixed dollar amount trades) address this by sampling in "activity time" rather than clock time, producing bars with more statistically uniform properties, closer to independent and identically distributed returns, which is a friendlier input for many downstream models.

```
def build_volume_bars(ticks, volume_threshold):
    bars = []
    bucket = []
    bucket_volume = 0
    for tick in ticks:
        bucket.append(tick)
        bucket_volume += tick.volume
        if bucket_volume >= volume_threshold:
            bars.append(summarize_bar(bucket))
            bucket, bucket_volume = [], 0
    return bars
```

Switching from time bars to volume or dollar bars is a low-effort change that measurably improves the statistical properties of downstream signal calculations for many strategies, and it deserves a place in an intermediate engineer's default toolkit rather than being treated as an exotic technique.

## 7. Detecting Regime Shifts

Markets move between distinguishable regimes, trending versus range-bound, high volatility versus low, risk-on versus risk-off, and a strategy tuned for one regime often underperforms or loses money outright in another. A practical, engineering-friendly approach to regime detection doesn't require a full hidden Markov model implementation to be useful. Simple rolling statistics (realized volatility level, trend strength via a normalized moving-average slope, average correlation across a basket of related assets) tracked over time and compared against their own historical distribution can flag "this looks like an unusual regime" reliably enough to gate strategy behavior.

The engineering pattern that works well in production is a regime classifier running as a separate, independently monitored component that publishes a regime label or a continuous regime score, which downstream strategies subscribe to and use to scale position sizes or disable themselves entirely, rather than each strategy trying to detect regime shifts independently and inconsistently.

## 8. Pitfalls: Spurious Correlation and Overfitting to Noise

With enough time series and enough lag combinations, you will always find statistically significant-looking relationships that are pure noise. This is a mathematical certainty, not a risk you can engineer away entirely, only manage. Guard against it with three habits: hold out data you genuinely never look at during model development, apply multiple-testing corrections whenever you screen many candidate signals or pairs, and demand an economic story for why a relationship should exist before trusting a purely statistical finding, especially one discovered through broad automated search rather than a specific hypothesis.

A fourth habit, less commonly discussed but equally valuable, is deliberately testing your discovery process against known-null data. Generate synthetic random series with no genuine embedded relationship, run your full signal-discovery pipeline against them exactly as you would against real data, and see how often the pipeline reports a "significant" finding anyway. If your process regularly finds apparent signal in pure noise at a rate higher than your nominal significance threshold implies it should, the pipeline itself has a methodological flaw worth fixing before you trust any of its findings on real data, however statistically significant those findings appear to be in isolation.

## Summary

- Work with returns, not raw prices, and verify stationarity on rolling windows rather than trusting a single full-sample test.
- Autocorrelation in returns is usually small and unstable in liquid markets; use it diagnostically on model residuals as much as for signal generation.
- Estimate volatility with an adaptive method like EWMA rather than a fixed-window historical standard deviation for anything used in real-time risk sizing.
- Cointegration testing requires an economic rationale first and multiple-testing correction second, or you will find spurious pairs.
- Volume or dollar bars generally produce statistically friendlier inputs than fixed time bars for most downstream models.
- Treat regime detection as a shared, independently monitored service that strategies subscribe to, not something each strategy reimplements.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
