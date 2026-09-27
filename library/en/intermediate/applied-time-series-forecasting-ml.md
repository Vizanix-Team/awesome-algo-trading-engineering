# Applied Time-Series Forecasting with Machine Learning

*By Vizanix — Intermediate Level*

> Move from classical time-series statistics to practical machine learning forecasting for trading, with an honest treatment of why most naive ML approaches to price forecasting fail.

![diagram](../../assets/neural-network.svg)

## Table of Contents

1. Why Forecasting Prices Directly Is the Wrong Framing
2. Classical Baselines You Must Beat First
3. Feature Windows and Sequence Framing for ML Models
4. Tree-Based Models for Tabular Financial Features
5. Sequence Models: When They Help and When They Don't
6. Cross-Validation for Time Series (and Why K-Fold Fails)
7. Evaluation Metrics Beyond Accuracy
8. From Forecast to Trading Signal

## 1. Why Forecasting Prices Directly Is the Wrong Framing

New practitioners often frame the problem as "predict tomorrow's closing price," feed a model historical prices, and get a model that appears to forecast well — because it has effectively learned that tomorrow's price is close to today's price, which is true and also useless for trading, since acting on that "forecast" produces a strategy indistinguishable from buy-and-hold with extra steps.

Reframe the target as something with genuine decision value: a return over a specific, tradeable horizon, a probability of exceeding some threshold, or a directional classification with an associated confidence. This reframing forces you to confront directly how hard the actual problem is, rather than being lulled by a low error metric on a target that was never predictive of anything useful in the first place.

## 2. Classical Baselines You Must Beat First

Before reaching for a neural network, establish honest classical baselines, because a sophisticated model that fails to beat a naive one has told you something important: either the signal isn't there in your feature set, or your evaluation methodology has a flaw letting the naive baseline look artificially strong.

A random walk baseline (predict no change, or predict the last observed change persists) is the minimum bar. An ARIMA model, fit properly with correctly identified order parameters, is a stronger and still fully interpretable baseline for series with genuine autocorrelation structure.

```
from statsmodels.tsa.arima.model import ARIMA

model = ARIMA(returns_series, order=(1, 0, 1))
fitted = model.fit()
forecast = fitted.forecast(steps=5)
```

If your elaborate gradient-boosted or deep learning model can't beat this simple baseline out of sample, on the same data and same evaluation protocol, that's a finding, not a failure to hide — it tells you the complexity isn't earning its keep for this particular problem and dataset, and you should either simplify or go find better features rather than tuning hyperparameters indefinitely.

## 3. Feature Windows and Sequence Framing for ML Models

Most tabular ML models (gradient boosting, random forests) need you to manually engineer the "memory" of the time series into fixed-width feature columns — lagged returns, rolling statistics, and momentum indicators computed at each point in time, each becoming one column in your training matrix.

```
def build_lag_features(returns, n_lags=10):
    df = pd.DataFrame({'returns': returns})
    for lag in range(1, n_lags + 1):
        df[f'lag_{lag}'] = df['returns'].shift(lag)
    return df.dropna()
```

Sequence models (recurrent networks, temporal convolutional networks, transformers) instead consume a raw or lightly processed window of recent observations directly and learn their own internal representation of relevant history, removing some manual feature engineering burden but adding a data-hunger and interpretability cost in exchange. Neither framing is universally superior; the right choice depends heavily on how much clean historical data you actually have relative to the model's capacity, and how much you value being able to explain a specific prediction after the fact.

## 4. Tree-Based Models for Tabular Financial Features

Gradient-boosted trees (implementations like XGBoost, LightGBM, or similar) are a strong default for financial tabular data because they handle nonlinear feature interactions well, are relatively robust to unscaled or oddly distributed features, and — importantly for a domain with a real risk of overfitting to noise — support straightforward regularization through tree depth, learning rate, and minimum samples per leaf.

```
import lightgbm as lgb

train_data = lgb.Dataset(X_train, label=y_train)
params = {
    'objective': 'regression',
    'max_depth': 4,          # shallow trees to limit overfitting
    'learning_rate': 0.03,
    'num_leaves': 15,
    'min_data_in_leaf': 200, # require substantial support per split
}
model = lgb.train(params, train_data, num_boost_round=300)
```

Keep trees shallow and require a meaningful minimum sample count per leaf for financial applications specifically, because with enough boosting rounds and deep enough trees, gradient boosting will happily memorize noise in a dataset where the true signal-to-noise ratio is low, which describes most short-horizon return prediction problems. Feature importance output from these models is also a genuinely useful diagnostic tool for pruning your feature set, not just a nice-to-have visualization — features with consistently near-zero importance across multiple retraining runs are strong candidates for removal.

## 5. Sequence Models: When They Help and When They Don't

Recurrent and attention-based sequence models can, in principle, capture more complex temporal dependencies than a fixed lag-feature representation. In practice, for the amount and signal-to-noise ratio of most financial time series (compared to, say, the volume of text or image data these architectures were originally developed against), they frequently underperform well-regularized tree-based models on tabular financial features, because they need substantially more data to reliably learn useful patterns without overfitting to the specific noise realizations in your training window.

Where sequence models genuinely tend to earn their added complexity is on problems with richer, higher-frequency input (full order book snapshots over short windows, tick-level microstructure sequences) where the temporal structure itself is complex and voluminous enough to reward a model that learns its own representation, rather than problems using coarse daily or hourly bars with a modest number of engineered features, where the simpler tabular approach is usually both more robust and much easier to validate and debug.

```
# Illustrative sketch, not literal library code
model = Sequential([
    LSTM(units=32, input_shape=(window_length, n_features), return_sequences=False),
    Dropout(0.3),          # meaningful dropout given limited effective sample size
    Dense(1)
])
```

Whichever architecture you choose, budget real time for a rigorous comparison against the tree-based baseline on identical data splits before committing production infrastructure to the more complex approach.

## 6. Cross-Validation for Time Series (and Why K-Fold Fails)

Standard K-fold cross-validation shuffles data randomly into folds, which for time series means training on future data and validating on past data within some folds — a direct and often invisible form of lookahead leakage that inflates validation performance in a way that will not survive contact with live, forward-only production data.

Use a time-respecting validation scheme instead: walk-forward validation, where you train on an expanding or rolling historical window and validate strictly on the period immediately following it, then roll the window forward and repeat.

```
def walk_forward_splits(n_samples, train_size, test_size, step):
    start = 0
    splits = []
    while start + train_size + test_size <= n_samples:
        train_idx = range(start, start + train_size)
        test_idx = range(start + train_size, start + train_size + test_size)
        splits.append((train_idx, test_idx))
        start += step
    return splits
```

Also insert a purge gap between the end of a training window and the start of its corresponding test window whenever your labels have forward-looking horizons (a label using the next 5 days of returns needs those 5 days excluded or gapped from the adjacent training data), or you'll leak information about the test period's outcomes into features that were technically computed from timestamps just before it.

## 7. Evaluation Metrics Beyond Accuracy

Standard ML metrics like accuracy or R-squared can be actively misleading for financial forecasting, because they weight every prediction equally, while in trading, a small number of large, correct directional calls at the right moments matter far more than a high hit rate on small, unimportant moves. Evaluate forecasts using metrics that better reflect trading value: rank correlation (Spearman's) between predicted and realized returns, since many trading strategies only need reliable relative ordering, not precise magnitude; hit rate specifically on larger predicted moves; and, most directly, the outcome of running predictions through a simplified backtest that turns forecasts into positions and reports realized P&L after estimated transaction costs.

```
def information_coefficient(predictions, realized_returns):
    return pd.Series(predictions).corr(pd.Series(realized_returns), method='spearman')
```

A model with mediocre raw accuracy but a consistently positive, stable information coefficient across many out-of-sample periods is frequently far more valuable in practice than a model with impressive point-accuracy that turns out to be concentrated in periods or instruments that don't translate into an executable trading edge.

## 8. From Forecast to Trading Signal

A forecast is not a trading signal until it passes through a deliberate translation layer: sizing (how much capital to commit given the forecast's confidence and typical error magnitude), thresholding (should a weak, near-zero forecast actually trigger a trade at all, given transaction costs), and risk overlay (does executing on this forecast respect existing position and exposure limits). Treat this translation layer as a first-class, separately tested component, not an afterthought bolted onto the model's raw output — many otherwise-promising models fail in live trading not because the forecast itself was wrong, but because the translation from forecast to position was naive, oversized, or ignored transaction costs that erased the entire modeled edge.

## Summary

- Frame the prediction target around tradeable returns or thresholds, never raw future prices, or you'll build a model that "predicts" nothing useful.
- Beat honest classical baselines (random walk, ARIMA) before trusting any more complex model's apparent edge.
- Prefer well-regularized, shallow gradient-boosted trees as a strong default for tabular financial features over sequence models, unless working with rich high-frequency input data.
- Always validate with walk-forward splits and purge gaps, never standard K-fold, to avoid lookahead leakage.
- Evaluate using rank correlation and backtested, cost-adjusted P&L, not raw accuracy or R-squared alone.
- Treat the translation from forecast to actual position size as a separately engineered, separately tested layer, not an afterthought.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
