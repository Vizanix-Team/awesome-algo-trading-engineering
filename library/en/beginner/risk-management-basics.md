# Risk Management Basics for New Quant Developers

*By Vizanix — Beginner Level*

> This book teaches you to think about risk before you think about profit, and gives you concrete, practical techniques for keeping a bad day from becoming a ruinous one.

![diagram](../../assets/pnl-curve.svg)

## Table of Contents

1. Why Risk Comes Before Return
2. Position Sizing: The Most Important Decision
3. Stop Losses and Their Limits
4. Diversification and Correlation
5. Leverage: A Tool That Cuts Both Ways
6. Drawdown: Living Through the Bad Stretch
7. Operational Risk: When the Bug Is the Real Danger
8. Building a Personal Risk Checklist

## 1. Why Risk Comes Before Return

New traders often ask "how much can I make," when the more useful first question is "how much can I lose, and can I survive that." This isn't pessimism; it's arithmetic. A strategy that loses 50% of its capital needs a 100% gain just to get back to where it started. Losses compound against you asymmetrically, which means protecting your capital from severe damage matters more, in the long run, than squeezing out a slightly higher average return.

Risk, in this context, means the possibility that an outcome differs from what you expected, in the unfavorable direction. Every trade carries risk: the price might move against you, the market might become illiquid right when you need to exit, or your own code might contain a bug you haven't found yet. Risk management is the discipline of identifying these possibilities in advance and deliberately limiting how much damage any single one of them can do.

You can think of risk management as the seatbelt of trading. It doesn't make you a better driver, and it doesn't get you to your destination faster. What it does is ensure that when something goes wrong, and eventually something will, the outcome is a manageable setback rather than a catastrophic, account-ending event. Every technique in this book serves that single purpose.

## 2. Position Sizing: The Most Important Decision

Position sizing means deciding how much capital to commit to any single trade, and it's arguably the single most consequential risk decision you make, more consequential than which specific strategy you run. Two traders using an identical strategy can have wildly different outcomes purely because of how much they sized each position.

A common, simple starting rule caps the amount you're willing to lose on any single trade at a small, fixed percentage of your total capital, commonly somewhere around one to two percent for a beginner. If you have a defined stop loss (covered next) that would trigger a five percent loss on the position, and you want to risk at most one percent of your total capital on this trade, you size the position so that a five percent move against you equals roughly one percent of your total account, not one percent of the position itself.

This approach means your position size shrinks automatically for trades with a wider stop loss, meaning trades where the price needs more room to move before you consider the idea proven wrong, and grows for trades with a tighter stop loss. This keeps your risk per trade roughly consistent even as the specific setup varies.

Resist the temptation to increase position size after a string of wins, a pattern sometimes called chasing, since a winning streak doesn't change the underlying probability of your next trade losing, and oversized positions after a hot streak are a common way a single subsequent loss does outsized damage.

## 3. Stop Losses and Their Limits

A stop loss, introduced briefly in the second book of this library, is a predetermined price level at which you exit a losing position, converting an open-ended, unknown potential loss into a bounded, known one. Setting a stop loss before you enter a trade, rather than deciding in the moment while already holding a losing position, matters enormously, because your judgment while watching a real loss unfold is measurably worse than your judgment beforehand, when you're not yet emotionally invested in a specific outcome.

Stop losses aren't perfect protection, though, and understanding their limits prevents a false sense of security. In a fast-moving or illiquid market, the actual price at which your stop order fills can be considerably worse than the stop price you set, an effect called slippage on the stop itself, discussed in the order book book of this library in the context of thin liquidity. A large enough sudden price gap can blow straight through your stop level without ever trading at it.

Setting a stop loss too tight, meaning too close to your entry price, causes another problem: ordinary, harmless price noise triggers it constantly, exiting you from otherwise sound trades before they have room to work. Setting it too wide defeats its purpose as meaningful protection. Calibrating stop distance against an instrument's typical volatility, rather than picking an arbitrary fixed number, produces far more sensible results, and this calibration is a place where the statistics covered later in this library becomes directly practical.

## 4. Diversification and Correlation

Diversification means spreading risk across multiple positions, strategies, or instruments rather than concentrating everything in one. The intuition is straightforward: if several independent things each have a small chance of going badly wrong, it's far less likely that all of them go wrong simultaneously than that any single one does.

The word doing the real work here is "independent." Two positions that tend to move together, called correlated positions, don't actually diversify your risk much even if they're technically different instruments. If you hold two assets that both tend to fall sharply during the same type of market stress, you've effectively concentrated risk while believing you've spread it, simply because you were looking at instrument names rather than at how the instruments actually behave together.

Correlation isn't fixed forever, either. Assets that behaved independently under normal conditions sometimes move together sharply during periods of severe market stress, when many participants sell everything at once regardless of an asset's individual characteristics. A beginner's practical takeaway is to check historical correlation between your intended positions, understand that this correlation can shift during stress exactly when you need diversification most, and avoid assuming that simply holding several different-sounding positions automatically means your risk is well spread.

## 5. Leverage: A Tool That Cuts Both Ways

Leverage, introduced briefly in the crypto connectivity book of this library, means controlling a position larger than the cash you've actually committed, typically by borrowing the difference. Leverage multiplies your potential gains, but it multiplies your potential losses by exactly the same factor, and it introduces a specific new danger: liquidation, where an exchange or broker automatically closes your position once losses eat too far into your posted collateral, often at an unfavorable moment and price entirely outside your control.

A useful mental exercise before using any leverage is calculating exactly what price move would trigger a full loss of your posted collateral, given the leverage ratio you're considering. With ten times leverage, a mere ten percent adverse price move can wipe out your entire committed capital on that position, a move that would be a completely ordinary, unremarkable fluctuation for many assets over even a single volatile day.

As a beginner, treat leverage as something to understand thoroughly and generally avoid, or use only in small, deliberately limited amounts, until you have real experience gauging how much an instrument typically moves and how quickly. The apparent efficiency of leverage, doing more with less capital, is exactly matched by an equally real and unforgiving efficiency at destroying that same capital when a trade moves against you.

## 6. Drawdown: Living Through the Bad Stretch

Drawdown, introduced in the backtesting book of this library, is the decline in your portfolio's value from a previous peak, and every trading approach, no matter how sound, experiences drawdowns. Understanding this in advance, and specifically thinking through how large a drawdown you can tolerate financially and psychologically before you're actually living through one, is a core piece of risk management that new traders routinely skip.

A strategy's historical maximum drawdown gives you a reference point, but remember from the backtesting book that history doesn't set a hard ceiling. A strategy can eventually experience a drawdown larger than anything in its backtest, simply because markets produce new, previously unseen situations over time. Plan around a drawdown somewhat worse than your worst historical example, not exactly equal to it.

The psychological dimension matters as much as the financial one. A drawdown that's financially survivable can still lead a trader to abandon a fundamentally sound strategy right before it recovers, purely because the emotional strain of watching sustained losses becomes unbearable. Deciding in advance, while calm, exactly what drawdown level would make you pause and reassess, versus what level is simply an expected, tolerable part of the strategy's normal behavior, prevents that decision from being made in a moment of panic.

## 7. Operational Risk: When the Bug Is the Real Danger

Market risk, the risk that prices move against you, gets most of the attention in trading discussions, but operational risk, the risk that something in your own system or process fails, causes plenty of real damage too, and it's entirely within your control to reduce.

A bug that inverts a buy and sell signal, a connectivity failure that leaves a position open when you believed your system had closed it, a mistyped order quantity that's ten times larger than intended: these are all operational failures rather than the market simply moving against a sound decision, and they're arguably more dangerous precisely because they can be far more severe and far more sudden than an ordinary bad trade.

Reduce operational risk through concrete habits: test new code thoroughly on a testnet or in paper trading, as covered in earlier books, before it ever touches real capital. Add explicit sanity checks in your order-placing code, like refusing to send any order above a hard-coded maximum size, regardless of what your strategy logic calculated, as a last line of defense against a calculation bug. Log everything your system does, so that when something does go wrong, you can reconstruct exactly what happened rather than guessing.

Treat every new piece of code you add to a live trading system with the same caution you'd want a surgeon to apply before a procedure: slow, deliberate, and checked, because the cost of a careless mistake here is measured in real money, not just wasted development time.

## 8. Building a Personal Risk Checklist

Bring the concepts in this book together into a short, concrete checklist you actually review before running any new strategy with real capital. How much am I risking on this single trade, as a percentage of my total capital. Where exactly is my stop loss, and is it calibrated to this instrument's actual typical volatility rather than picked arbitrarily. Are my positions genuinely diversified, or do they share a hidden correlation that would hurt me under stress. Am I using any leverage, and have I calculated exactly what move would wipe out my collateral. What drawdown would make me pause and reassess, and have I decided that in advance rather than in the moment. What operational safeguards, like a maximum order size check, protect me from my own bugs.

None of these questions has a universally correct answer; the right answer depends on your specific capital, temperament, and strategy. What matters is that you've actually asked and answered each one deliberately, in writing, before capital is at risk, rather than discovering the answer for the first time during a difficult moment when a real position is already going against you.

## Summary

- Protecting capital from severe damage matters more long-term than maximizing average return; losses compound asymmetrically against you.
- Position sizing, calibrated against a well-placed stop loss, is the single most consequential risk decision you make per trade.
- Diversification only works when positions are genuinely independent, and correlations can shift sharply during market stress.
- Leverage multiplies losses exactly as much as gains and introduces liquidation risk; use it sparingly as a beginner.
- Operational risk, bugs and process failures, is fully within your control to reduce through testing, sanity checks, and logging.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
