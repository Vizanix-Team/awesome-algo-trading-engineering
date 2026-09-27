# Building an Event-Driven Backtesting Engine

*By Vizanix — Intermediate Level*

> Build a backtesting engine that processes events in strict chronological order so your simulated results reflect what a live strategy could actually have achieved.

![diagram](../../assets/pnl-curve.svg)

## Table of Contents

1. Vectorized vs Event-Driven: Why the Distinction Matters
2. The Event Loop and Event Types
3. Avoiding Lookahead Bias
4. Modeling the Order Book and Fills
5. Latency Simulation
6. Transaction Costs and Slippage Models
7. Position and Portfolio Accounting
8. Validating Your Engine Against Reality

## 1. Vectorized vs Event-Driven: Why the Distinction Matters

Most people's first backtest is vectorized: load a DataFrame of daily closes, compute a signal column, shift it by one day, multiply by returns, sum it up. This works fine for a slow, low-frequency strategy with almost no path dependency, and it is fast to write. It also hides an entire category of bugs that only show up when your strategy's decisions depend on order of events within a day, on your own open orders affecting later decisions, or on realistic fill mechanics.

An event-driven backtester processes a chronologically ordered stream of events — a tick, a fill, a timer, a signal update — one at a time, and your strategy code reacts to each event exactly as it would in live trading, receiving only the information available at that instant. This is slower to build and slower to run, but it is the only architecture that lets you reuse the same strategy code in backtest and in production with minimal changes, because both environments consume the same event interface.

The intermediate-level insight worth internalizing: the value of an event-driven engine is not realism for its own sake, it is that it structurally prevents you from accidentally using future information. A vectorized backtest lets you write `df['signal'] = df['price'].rolling(20).mean()` and forget that at row `i`, you can only see prices up to `i`, not the future rows pandas happily computed for you. An event loop that only ever hands your strategy the current event, with no reference to the future, makes that mistake much harder to commit.

## 2. The Event Loop and Event Types

At minimum, define four event types: `MarketEvent` (new tick or bar arrives), `SignalEvent` (strategy decides to act), `OrderEvent` (order sent to the simulated exchange), and `FillEvent` (simulated exchange reports an execution). Route all of them through a single priority queue ordered by timestamp.

```
class EventQueue:
    def __init__(self):
        self.heap = []  # min-heap by timestamp

    def push(self, event):
        heapq.heappush(self.heap, (event.timestamp, event.seq, event))

    def pop(self):
        return heapq.heappop(self.heap)[2]
```

The `seq` tiebreaker matters: two events can share a timestamp (a tick and a timer firing in the same millisecond), and you need a deterministic, reproducible ordering, not whatever order Python's heap happens to return for equal keys. Assign a strictly increasing sequence number at event creation time and always sort by `(timestamp, seq)`.

The main loop is deliberately dumb:

```
while not queue.empty():
    event = queue.pop()
    if isinstance(event, MarketEvent):
        strategy.on_market(event)       # may push SignalEvents
    elif isinstance(event, SignalEvent):
        portfolio.on_signal(event)      # may push OrderEvents
    elif isinstance(event, OrderEvent):
        exchange_sim.on_order(event)    # may push FillEvents
    elif isinstance(event, FillEvent):
        portfolio.on_fill(event)
```

Every downstream component only reacts to what has already happened; nothing peeks ahead into the queue.

## 3. Avoiding Lookahead Bias

Lookahead bias is the single most common reason a backtest looks profitable and a live strategy does not. It creeps in through several specific mechanisms worth naming explicitly.

Bar-close bias: if you compute a signal using a daily bar's close price, and then simulate filling your order at that same day's close, you have implicitly assumed you can trade at a price you could only know after the bar closed. Fix this by delaying execution to the next bar's open, or by working at a finer time resolution where the signal-to-order latency is explicit and nonzero.

Survivorship bias: backtesting only against instruments that still exist today silently excludes everything that got delisted, went bankrupt, or was acquired, which biases your universe toward past winners. Your historical instrument universe needs to reflect what was actually tradeable at each historical date, not what is tradeable now.

Parameter fitting leakage: if you tune a strategy's parameters by testing many variations against the same historical window and picking the best one, you have not backtested a strategy, you have overfit a curve to noise. Reserve out-of-sample data you never touch during parameter selection, and treat that final test as a one-shot evaluation, not something you also iterate against.

## 4. Modeling the Order Book and Fills

The fidelity of your fill simulation determines how trustworthy your results are. A naive backtester fills every order instantly at the last traded price, regardless of size — this overstates performance for any strategy that trades enough size to move the market, or that relies on passive limit orders capturing spread.

A better approach simulates against synthetic or historical order book depth: for a market order of size Q, walk the book levels and compute a volume-weighted average fill price across however many levels of depth Q consumes, rather than a single fixed price. For limit orders, only fill when the simulated market price crosses your limit, and be honest about queue position — a passive limit order sitting behind other orders at the same price level does not fill just because the price touched your level once; it needs the volume ahead of you in the queue to trade through first.

```
def simulate_market_fill(book_levels, quantity):
    remaining = quantity
    cost = 0.0
    for price, size in book_levels:
        take = min(remaining, size)
        cost += take * price
        remaining -= take
        if remaining <= 0:
            break
    if remaining > 0:
        raise InsufficientLiquidity(quantity)
    return cost / quantity  # VWAP fill price
```

## 5. Latency Simulation

Live trading has a real, nonzero delay between deciding to trade and the exchange receiving the order, and another delay before you learn the outcome. A backtest that ignores this implicitly assumes you can act on information instantaneously, which especially inflates the apparent performance of any strategy reacting to fast-moving signals.

Model latency as explicit event delays: when your strategy emits an `OrderEvent`, do not process it against the current market state — schedule its arrival at the simulated exchange some milliseconds (or more, depending on your infrastructure) later, using the same event queue. This single change often meaningfully reduces the apparent edge of short-horizon strategies, which is valuable information, not an inconvenience.

## 6. Transaction Costs and Slippage Models

Every fill should incur a modeled cost: exchange fees or rebates depending on maker/taker status, and a slippage component reflecting the fact that your own order size affects price. A simple but honest slippage model scales cost with the square root of order size relative to typical volume, reflecting the commonly observed pattern that impact grows sublinearly with size rather than linearly.

```
def slippage_bps(order_qty, avg_daily_volume, impact_coefficient):
    participation = order_qty / avg_daily_volume
    return impact_coefficient * (participation ** 0.5) * 10000
```

Calibrate `impact_coefficient` conservatively rather than optimistically — an under-modeled cost is far more dangerous to a strategy's live viability than an over-modeled one, because it lets a marginal strategy look profitable in backtest when it is not.

## 7. Position and Portfolio Accounting

Route every fill through a single portfolio object that maintains cash, positions, and mark-to-market value consistently. Compute portfolio equity at every market event, not just at trade events, so your equity curve reflects unrealized P&L moving with the market between trades, not a staircase that only changes when you trade.

Track realized and unrealized P&L separately using a defined cost-basis method, and keep a full trade log alongside the equity curve. When a backtest result looks surprising, the trade log is what lets you trace the specific decisions that produced it, rather than staring at an aggregate Sharpe ratio and guessing.

## 8. Validating Your Engine Against Reality

Before trusting any strategy result from your engine, validate the engine itself. Run a strategy that should be trivially unprofitable after costs (e.g., pure noise trading) and confirm it loses money at roughly the rate your cost model implies. Run a known buy-and-hold benchmark through the full pipeline and confirm it matches a simple manual calculation. Feed the engine a single synthetic order book scenario with a hand-computed expected fill price and assert your simulator matches exactly.

Finally, whenever feasible, paper-trade the strategy live for a period and compare live fills against what your backtester would have simulated for the same market conditions. Persistent, systematic divergence here is the clearest signal that your backtest has a fidelity gap worth closing before risking real capital.

## Summary

- Event-driven architecture structurally reduces lookahead bias by only exposing your strategy to past and present events.
- Use a single timestamp-and-sequence-ordered priority queue as the backbone of the engine.
- Simulate fills against order book depth and respect queue position for passive limit orders, not just last-price fills.
- Model latency explicitly as event-arrival delay, not as an afterthought.
- Use a slippage model with sublinear cost scaling, calibrated conservatively.
- Validate the engine itself with known-answer scenarios before trusting any strategy result it produces.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
