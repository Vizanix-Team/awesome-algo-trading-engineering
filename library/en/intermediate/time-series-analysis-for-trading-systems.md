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

Most introductory time-series methods assume your data is stationary, identically distributed, and free of structural breaks. Financial data violates all three routinely. Volatility itself changes over time in clusters — calm stretches followed by turbulent ones. Correlations between assets shift with the macro regime. A statistical relationship that held for three years can break the week a company's fundamentals change or a market structure rule shifts.

None of this means statistics are useless for trading — it means you need methods that are robust to these violations, and you need discipline about testing whether your assumptions actually hold for the data you are using, rather than assuming a textbook method transfers cleanly. This chapter builds that toolkit at a practical level: what to check, how to check it, and what to do when the checks fail.

## 2. Stationarity and Why You Should Care

A stationary series has statistical properties — mean, variance, autocorrelation structure — that do not change over time. Raw asset prices are almost never stationary: a price series trending from $50 to $150 over a year has a mean that clearly depends on when you measure it. Returns (percentage or log changes) are much closer to stationary, which is why nearly every quantitative model works with returns rather than raw prices.

The Augmented Dickey-Fuller test is the standard tool for checking whether a series has a unit root (a hallmark of non-stationarity — random-walk-like behavior where shocks persist indefinitely rather than decaying). In practice:

```
from statsmodels.tsa.stattools import adfuller

result = adfuller(returns_series, autolag='AIC')
p_value = result[1]
# p_value < 0.05 suggests you can reject the null of a unit root
# i.e., the series is more likely stationary
```

Do not treat this test as a binary gate that clears your model for production. A series can pass an ADF test on one historical window and clearly fail on another, especially around regime changes. Run it on rolling windows, not just the full sample, and watch how the result evolves — if it flips frequently, your "stationary" series is stationary only in a fragile, sample-dependent sense, and any model assuming otherwise deserves extra skepticism.

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

In liquid, well-arbitraged markets, raw return autocorrelation at short lags is typically small in magnitude and unstable — which should make you suspicious of any backtest that shows a large, clean autocorrelation-based edge with no transaction cost erosion. The more common practical use of autocorrelation analysis is diagnostic: checking whether your model's *residuals* still show autocorrelation, which would indicate your model is leaving exploitable structure on the table, or conversely that your position-sizing logic is introducing unwanted serial correlation in your P&L that increases risk without adding return.

## 4. Volatility Clustering and Estimating Variance

Volatility clustering — the empirical observation that large price moves tend to be followed by more large moves, and calm periods by more calm periods — is one of the most robust patterns in financial data, far more reliable than any specific directional signal. This has direct engineering consequences: a position sizing system using a stale, long-window volatility estimate will oversize positions right as volatility is spiking and undersize them right as it's calming down, both bad.

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

The decay parameter `lam` controls the effective memory of the estimate — closer to 1 means slower adaptation, closer to 0 means faster reaction but noisier estimates. For systems that need to size positions or set risk limits in near real time, this kind of adaptive estimator is far more useful than a fixed-window historical standard deviation recomputed once a day.

## 5. Cointegration and Pairs Relationships

Two assets can each be individually non-stationary (both trending randomly) while some linear combination of them is stationary — this is cointegration, and it's the statistical basis for most pairs-trading and statistical arbitrage strategies. Intuitively: if two stocks are cointegrated, their price *spread*, appropriately weighted, tends to revert to some equilibrium level even though each individual price wanders without bound.

The Engle-Granger approach is the simplest starting point: regress one series on the other to estimate the hedge ratio, then test the residual spread for stationarity using the same kind of unit-root test discussed earlier.

```
import numpy as np
from statsmodels.tsa.stattools import adfuller

hedge_ratio = np.polyfit(price_b, price_a, 1)[0]
spread = price_a - hedge_ratio * price_b
p_value = adfuller(spread)[1]
# low p_value suggests the spread is mean-reverting -> candidate pair
```

The practical trap here is testing many candidate pairs and only reporting the ones that pass — with enough pairs tested, some will show spurious cointegration by pure chance. Correct for multiple testing, and more importantly, require an economic or structural rationale for why two assets should be related (same sector, same underlying commodity exposure, an index-and-component relationship) before you even run the statistical test, rather than mining an entire universe blindly.

## 6. Resampling and the Bar Construction Problem

Most trading systems work with "bars" — OHLCV summaries over some interval — rather than raw tick data, for tractability. The default choice, time bars (one bar per fixed clock interval), has a subtle statistical drawback: trading activity is not uniform in time, so a time bar during a quiet period represents very little information while a time bar during a burst of activity compresses a huge amount of information into the same-sized bucket.

Volume bars (a new bar every time a fixed quantity of shares trades) and dollar bars (a new bar every time a fixed dollar amount trades) address this by sampling in "activity time" rather than clock time, producing bars with more statistically uniform properties — closer to independent and identically distributed returns, which is a friendlier input for many downstream models.

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

Markets move between distinguishable regimes — trending versus range-bound, high volatility versus low, risk-on versus risk-off — and a strategy tuned for one regime often underperforms or loses money outright in another. A practical, engineering-friendly approach to regime detection does not require a full hidden Markov model implementation to be useful: simple rolling statistics (realized volatility level, trend strength via a normalized moving-average slope, average correlation across a basket of related assets) tracked over time and compared against their own historical distribution can flag "this looks like an unusual regime" reliably enough to gate strategy behavior.

The engineering pattern that works well in production is a regime classifier running as a separate, independently monitored component that publishes a regime label or a continuous regime score, which downstream strategies subscribe to and use to scale position sizes or disable themselves entirely, rather than each strategy trying to detect regime shifts independently and inconsistently.

## 8. Pitfalls: Spurious Correlation and Overfitting to Noise

With enough time series and enough lag combinations, you will always find statistically significant-looking relationships that are pure noise — this is a mathematical certainty, not a risk you can engineer away entirely, only manage. Guard against it with three habits: hold out data you genuinely never look at during model development, apply multiple-testing corrections whenever you screen many candidate signals or pairs, and demand an economic story for why a relationship should exist before trusting a purely statistical finding, especially one discovered through broad automated search rather than a specific hypothesis.

## Summary

- Work with returns, not raw prices, and verify stationarity on rolling windows rather than trusting a single full-sample test.
- Autocorrelation in returns is usually small and unstable in liquid markets; use it diagnostically on model residuals as much as for signal generation.
- Estimate volatility with an adaptive method like EWMA rather than a fixed-window historical standard deviation for anything used in real-time risk sizing.
- Cointegration testing requires an economic rationale first and multiple-testing correction second, or you will find spurious pairs.
- Volume or dollar bars generally produce statistically friendlier inputs than fixed time bars for most downstream models.
- Treat regime detection as a shared, independently monitored service that strategies subscribe to, not something each strategy reimplements.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
