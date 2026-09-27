# Feature Engineering for Financial Machine Learning

*By Vizanix — Intermediate Level*

> Learn to build features from market data that actually carry predictive signal and survive the specific ways financial data tries to fool naive machine learning pipelines.

![diagram](../../assets/neural-network.svg)

## Table of Contents

1. Why Financial Feature Engineering Is Different
2. Leakage: The Silent Killer of Financial ML
3. Price-Derived Features That Actually Generalize
4. Volume and Liquidity Features
5. Cross-Sectional Features and Universe Normalization
6. Labeling: What Are You Actually Predicting
7. Feature Stability and Decay
8. Building a Feature Pipeline You Can Trust

## 1. Why Financial Feature Engineering Is Different

Most machine learning tutorials assume your data-generating process is roughly stable and your labels are unambiguous. Neither assumption holds cleanly in finance. The statistical relationship between a feature and forward returns can decay as more participants discover and trade on it, and even the definition of a label — what exactly counts as "the outcome you're predicting" — is a genuine design decision with real tradeoffs, not a given.

This means financial feature engineering is less about finding one clever feature and more about building a disciplined process: understanding exactly what information was available at the moment a feature was computed, understanding how a feature behaves across different market regimes, and being honest about the difference between a feature that's genuinely informative and one that merely looks informative due to how you happened to construct your test.

The engineering culture around this discipline matters as much as any individual technique. Teams that build financial ML features successfully tend to treat every proposed feature with the same skepticism they'd apply to a proposed change to a production trading system, requiring a clear hypothesis for why the feature should carry signal, a clean point-in-time-correct implementation, and out-of-sample validation before it earns a place in a live model. Teams that treat feature engineering as a purely exploratory, anything-goes research exercise tend to accumulate large feature sets dominated by noise-fitted artifacts that look impressive in a research notebook and add nothing, or actively subtract value, once deployed.

## 2. Leakage: The Silent Killer of Financial ML

Leakage is the single most common reason a financial ML model looks excellent in research and fails in production, and it's rarely as obvious as literally including tomorrow's price as a feature. Subtle leakage sources dominate in practice.

Look-ahead in rolling calculations is common: computing a rolling average "centered" on the current observation instead of trailing it means the calculation implicitly uses future data points. Always use trailing windows and double check your library's default behavior — some default to centered windows, which silently leaks future information into what looks like an innocent moving average feature.

Label-feature timing mismatch: if your label is "return over the next hour" but your feature uses a data point that itself was only fully known at the end of that hour (e.g., a daily-adjusted volume figure that gets revised throughout the day), you've built a feature that couldn't have existed at prediction time in live trading.

Universe survivorship in feature construction: computing a cross-sectional percentile rank feature using today's universe of tradeable instruments, when historically some of those instruments didn't exist yet or had different characteristics, silently biases your historical feature distribution.

```
# WRONG: centered window leaks future information
df['ma_leaky'] = df['price'].rolling(window=20, center=True).mean()

# CORRECT: trailing window only uses past and current data
df['ma_correct'] = df['price'].rolling(window=20).mean()
```

Build a habit of asking, for every single feature: "at the exact timestamp this row represents, could I have actually computed this value using only information available up to that instant?" If the answer requires any hesitation, the feature needs rework before it goes anywhere near a model.

Reporting-lag leakage is another common source worth naming: fundamental data (earnings, revenue figures, macroeconomic indicators) is often reported with a delay after the period it describes, and some data vendors timestamp these records by the period they cover rather than by when the data actually became public. A feature built from quarterly earnings using the report's period-end date as its availability timestamp, rather than the actual public release date weeks later, leaks the entire reporting lag directly into your backtest, letting your model "know" a quarter's results before the market itself did. Always source and use the actual public release timestamp for any data with a meaningful reporting lag, and treat any vendor field ambiguously named something like "period" or "date" with suspicion until you've confirmed exactly which timestamp it represents.

## 3. Price-Derived Features That Actually Generalize

Raw price levels make poor features because they're non-stationary and scale-dependent across instruments trading at wildly different price points. Transform toward stationary, scale-invariant representations: returns over multiple horizons, normalized price position within a recent range, and volatility-adjusted (rather than raw) price changes.

```
def build_price_features(prices, windows=(5, 20, 60)):
    features = {}
    for w in windows:
        ret = prices.pct_change(w)
        vol = prices.pct_change().rolling(w).std()
        features[f'return_{w}d'] = ret
        features[f'return_{w}d_vol_adj'] = ret / (vol * (w ** 0.5) + 1e-9)
        rolling_min = prices.rolling(w).min()
        rolling_max = prices.rolling(w).max()
        features[f'range_position_{w}d'] = (
            (prices - rolling_min) / (rolling_max - rolling_min + 1e-9)
        )
    return features
```

Volatility-adjusted returns matter because a 2% move means something very different for a historically calm instrument than for a historically volatile one — dividing by realized volatility puts different instruments and different time periods on a more comparable footing, which generally helps a model generalize across both.

Range-position features (where does the current price sit within its recent trading range) capture a different kind of information than raw returns — they encode something closer to "is this near a recent extreme" without depending on the specific price scale, which tends to be more stable across different instruments than a scale-dependent raw price feature.

Multi-timescale versions of the same base feature, computed across several window lengths simultaneously (5-day, 20-day, 60-day momentum, for instance) are more valuable together than any single window in isolation, because they let a model learn interactions between short-term and long-term regimes that a single fixed window can't express — a short-term momentum reading that agrees with the long-term trend often carries different implications than one that contradicts it, and providing both windows as separate features lets a sufficiently flexible model learn that distinction directly rather than requiring you to hand-engineer an interaction term explicitly.

## 4. Volume and Liquidity Features

Volume features need the same non-stationarity treatment as prices — raw daily volume trends over time as an instrument's popularity changes, so use relative volume (current volume divided by a trailing average) rather than raw volume levels.

```
def volume_features(volume, window=20):
    avg_volume = volume.rolling(window).mean()
    return {
        'relative_volume': volume / (avg_volume + 1e-9),
        'volume_trend': avg_volume.pct_change(window),
    }
```

Liquidity-related features — bid-ask spread level and its recent trend, order book depth at the touch, estimated market impact for a standard trade size — carry genuine signal for many strategies, particularly ones concerned with execution cost or short-horizon reversal effects, since liquidity conditions and subsequent price behavior are often related. These features require access to intraday or tick-level data to construct properly; approximating them from daily OHLCV data alone (e.g., using the daily high-low range as a spread proxy) is a reasonable fallback but a meaningfully noisier one, and you should treat results built on such proxies with appropriate skepticism.

Volume profile shape features, capturing not just how much volume traded but when during the day it concentrated, can also carry information distinct from a simple aggregate volume count. An instrument whose volume has recently shifted to concentrate more heavily near the close, relative to its own historical pattern, may be signaling something about changing participant composition worth capturing as a feature in its own right, separate from whatever aggregate relative-volume feature you're already computing, since the aggregate figure can stay flat even while the underlying intraday distribution shifts meaningfully.

## 5. Cross-Sectional Features and Universe Normalization

Many financial ML models operate cross-sectionally — ranking or scoring many instruments against each other at each point in time, rather than treating each instrument's time series in isolation. This requires normalizing features across the universe at each timestamp, typically via rank transformation or z-scoring within that day's cross-section, so a feature's scale differences across instruments don't dominate the signal.

```
def cross_sectional_rank(feature_df):
    # feature_df: rows = timestamps, columns = instruments
    return feature_df.rank(axis=1, pct=True)
```

The critical discipline here: perform this cross-sectional normalization using only the instruments that were actually part of your tradeable universe *at that historical date*, not your current universe applied retroactively. Getting this wrong is one of the most common and hardest-to-detect leakage sources in cross-sectional financial ML, because the resulting bug produces smoothly plausible-looking features that pass every simple sanity check.

## 6. Labeling: What Are You Actually Predicting

The label you choose shapes everything downstream, and there's rarely one obviously correct choice. Fixed-horizon labels (return over exactly the next N minutes or days) are simple but can mislabel a move that would have triggered a stop-loss or take-profit well before the horizon ends, treating a strategy that would have exited early as if it held the full period.

An alternative is a triple-barrier style label: define an upper barrier (take-profit level), a lower barrier (stop-loss level), and a time barrier (maximum holding period), and label each observation by whichever barrier gets touched first. This produces labels that more directly reflect how a real strategy following stop-loss and take-profit discipline would actually experience the outcome.

```
def triple_barrier_label(prices, entry_idx, upper_pct, lower_pct, max_holding):
    entry_price = prices[entry_idx]
    upper = entry_price * (1 + upper_pct)
    lower = entry_price * (1 - lower_pct)
    for i in range(entry_idx + 1, min(entry_idx + max_holding + 1, len(prices))):
        if prices[i] >= upper:
            return 1   # upper barrier hit first
        if prices[i] <= lower:
            return -1  # lower barrier hit first
    return 0  # time barrier hit, neither triggered
```

Whichever labeling scheme you choose, be explicit and consistent about it across your research pipeline, since comparing model performance across experiments that silently used different labeling logic produces conclusions that aren't actually comparable.

## 7. Feature Stability and Decay

A feature that predicted returns well two years ago may predict them poorly today, either because market structure changed or because enough participants started trading on the same signal, competing away its edge. Track feature importance and predictive power over rolling windows, not just as a single number computed once over your full historical sample, and treat a feature whose importance is trending down as a warning sign worth investigating before it becomes a live problem.

Building this monitoring into your pipeline as an ongoing process, rather than a one-time research exercise, is what separates a research artifact from a production-grade feature set. A feature dashboard tracking rolling information coefficient (the correlation between a feature and forward returns, recomputed over a trailing window) per feature over time gives you an early warning system for exactly this kind of decay.

Distinguish feature decay from feature regime-dependence carefully, since they call for different responses. A feature that's decaying monotonically, with information coefficient trending steadily toward zero over an extended period, is likely being arbitraged away and should eventually be retired. A feature whose information coefficient oscillates between clearly positive and clearly negative depending on identifiable market conditions (say, positive in trending regimes and negative in range-bound ones) is not decaying at all — it's regime-dependent, and the right response is to condition the feature's use on a regime signal rather than discarding it, since it still carries real, usable information within the regimes where it performs well.

## 8. Building a Feature Pipeline You Can Trust

Structure your feature pipeline so the exact same code path generates features in both backtesting and live production, computed from a point-in-time correct data store — meaning a query for "what did this feature look like as of date D" always returns what was actually computable using data available up to D, including correctly excluding any data revisions that happened after the fact. Maintaining this point-in-time discipline in your underlying data storage is unglamorous work, but it's the single highest-leverage investment you can make to prevent leakage systematically rather than catching it feature by feature through manual review.

## Summary

- Leakage is usually subtle — centered rolling windows, mistimed labels, and retroactively applied universes are the most common and hardest-to-spot sources.
- Transform raw prices and volumes into stationary, scale-comparable features: returns, volatility-adjusted returns, relative volume, range position.
- Normalize cross-sectional features using the tradeable universe as it existed at each historical date, never today's universe applied backward.
- Choose your labeling scheme deliberately; a triple-barrier approach often reflects real trading behavior better than a fixed-horizon return label.
- Monitor feature predictive power over rolling windows to catch decay before it silently degrades live performance.
- Maintain a point-in-time correct data store so backtest and live pipelines compute features identically from the same disciplined source.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
