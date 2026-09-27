# State Reconciliation: Keeping Your System Honest

*By Vizanix — Intermediate Level*

> Learn to design reconciliation as a first-class, continuously running discipline rather than an end-of-day afterthought, so your system's beliefs about the world never silently diverge from reality.

![diagram](../../assets/system-architecture.svg)

## Table of Contents

1. Why State Diverges, Even in Correct Code
2. Sources of Truth and the Reconciliation Triangle
3. Designing a Reconciliation Engine
4. Break Classification and Severity
5. Real-Time vs Batch Reconciliation
6. Automated Remediation: When to Trust It
7. Reconciliation Across Multiple Venues
8. Building Reconciliation Into the Development Lifecycle

## 1. Why State Diverges, Even in Correct Code

It is tempting to think reconciliation exists to catch bugs, and it does, but that undersells its importance. Even a flawlessly implemented trading system will experience state divergence, because trading systems operate across an unreliable network boundary with an external, independently operated counterparty — the exchange. Messages get delayed, connections drop and reconnect, and the two sides of that boundary can legitimately hold different views of "current state" for brief windows, by design, not by bug.

The job of reconciliation is not to prevent divergence entirely — that is impossible given the physics of distributed systems separated by a network — but to detect divergence quickly, bound how long it persists, and provide a principled process for resolving it once detected. A system without reconciliation does not have less divergence than one with it; it simply has divergence nobody notices until a client statement doesn't match or an audit finds the gap months later.

Framing reconciliation this way also changes how you budget engineering time around it. Teams that treat reconciliation as a bug-catching afterthought tend to build it once, minimally, and move on to feature work. Teams that treat it as a permanent, load-bearing piece of infrastructure invest in it continuously, the same way they'd invest in monitoring or testing infrastructure, because its value compounds every single day the trading system runs, quietly catching the small, inevitable divergences before they compound into something a client or a regulator eventually notices on your behalf.

## 2. Sources of Truth and the Reconciliation Triangle

Think of reconciliation as comparing at least three independent views of the same underlying reality: your internal order/position state (what your OMS believes happened based on events it processed), the exchange's authoritative state (what the exchange's own records say, retrievable via position and order status queries), and, where applicable, a clearing or custodian record (an independent third party's view, often lagged by a settlement cycle but authoritative for legal purposes).

These three views should agree, and the interesting engineering work is entirely about what to do in the moments they don't. Two-way reconciliation (your system versus the exchange) catches processing bugs and message loss on your side. Three-way reconciliation (adding the clearing/custodian view) catches a different, often more serious class of problem: cases where your system and the exchange agree with each other but both are wrong relative to what actually settles, which can happen due to trade allocation errors, corporate actions, or clearing-level breaks that never surface in the exchange's live trading records at all.

Prioritize which reconciliation triangle to build first based on where your actual operational risk concentrates, rather than trying to build all three simultaneously from day one. A firm trading a single instrument type through a single clearing relationship might reasonably defer building full three-way reconciliation until volume justifies the investment, while a firm operating across many venues with complex allocation logic across multiple underlying accounts needs the three-way view from the start, because the allocation step itself is exactly where the kind of break that two-way reconciliation is structurally blind to tends to originate.

## 3. Designing a Reconciliation Engine

A reconciliation engine's core loop is simple to describe and easy to get wrong in the details: pull or receive each side's current state, normalize both into a common representation, diff them, and classify any differences. The normalization step deserves more attention than engineers usually give it — your internal representation might key positions by an internal instrument ID while the exchange keys by its own symbol convention, and a naive diff that doesn't map these correctly will report false breaks constantly, training your team to ignore reconciliation alerts.

```
def reconcile_positions(internal_positions, exchange_positions, symbol_map):
    breaks = []
    exchange_by_internal_id = {
        symbol_map.to_internal(p.symbol): p for p in exchange_positions
    }
    all_ids = set(internal_positions) | set(exchange_by_internal_id)
    for instrument_id in all_ids:
        internal_qty = internal_positions.get(instrument_id, 0)
        exchange_qty = exchange_by_internal_id.get(instrument_id, PositionRecord(0)).qty
        if internal_qty != exchange_qty:
            breaks.append(Break(instrument_id, internal_qty, exchange_qty))
    return breaks
```

Timing alignment matters as much as symbol mapping. Comparing your internal snapshot taken at 10:00:00.000 against an exchange snapshot that reflects state as of 10:00:00.400 will produce false breaks for any order that filled in that 400-millisecond window, even though nothing is actually wrong. A robust reconciliation engine either uses a quiescent point (a moment both sides agree no in-flight activity exists, like a defined settlement cutoff) or explicitly accounts for in-flight orders when comparing, excluding them from break classification until they settle into a final state.

Lot-size and rounding conventions deserve explicit handling too, since some instruments trade in fractional units on one venue and whole-unit lots on another, or a clearing record aggregates fills at a coarser precision than your internal system's tick-level fill log. Build your normalization layer to round or truncate consistently on both sides of a comparison using the coarser of the two precisions, rather than comparing a highly precise internal number against a rounded external one and flagging the resulting, entirely expected, small discrepancy as a genuine break.

## 4. Break Classification and Severity

Not every break deserves the same response, and treating them uniformly either causes alert fatigue (if everything pages someone) or dangerous complacency (if nothing does). Classify breaks along at least two dimensions: magnitude (a one-share difference versus a thousand-share difference) and persistence (a break that resolves itself within the next snapshot, likely due to timing, versus one that persists across multiple reconciliation cycles, which strongly suggests a genuine processing error rather than a timing artifact).

A practical severity model:

```
def classify_break(break_record, history):
    if break_record.age_cycles >= 2 and break_record.magnitude > TIMING_NOISE_THRESHOLD:
        return Severity.CRITICAL   # persistent and material -> page
    if break_record.magnitude > MATERIAL_THRESHOLD:
        return Severity.HIGH       # large but possibly transient -> urgent review
    return Severity.LOW            # small and likely timing noise -> log and track
```

Persisting breaks in a database with their full history, not just their current state, lets you distinguish "this specific instrument has had a flapping one-lot break for three days, probably a lot-size rounding issue" from "this is a new, large, and growing break that needs immediate attention." Pattern recognition on break history is often what actually catches subtle bugs, more than any single reconciliation run in isolation.

Assign every recurring break category an explicit owner and a target resolution date, tracked the same way you'd track any other engineering backlog item, rather than letting known, low-severity flapping breaks accumulate indefinitely as tolerated background noise. A break tolerated long enough eventually becomes invisible to the team, and that invisibility is exactly the condition under which a genuinely new and unrelated break, arriving with a similar signature, goes unnoticed because everyone has learned to filter out "the usual noise" without checking whether this particular instance is actually the usual noise or something new wearing a familiar disguise.

## 5. Real-Time vs Batch Reconciliation

End-of-day batch reconciliation, comparing final positions after markets close, catches the important comprehensive class of errors, but by definition it cannot catch a break early enough to prevent same-day risk decisions from being made on wrong information. Any active trading desk needs some form of intraday, near-real-time reconciliation running continuously, comparing live-updating internal state against periodically polled or streamed exchange state throughout the session.

The engineering tradeoff is polling frequency versus load: reconciling every second against every exchange gives you fast break detection but adds meaningful load to both your systems and the exchange's query infrastructure, and most exchanges rate-limit these queries. A common practical pattern polls actively traded instruments more frequently (every few seconds) and lightly traded ones less often, and additionally triggers an out-of-cycle reconciliation check immediately after any anomalous event, like a dropped and reconnected exchange session, when the probability of a real break spikes.

## 6. Automated Remediation: When to Trust It

Once you detect a break, the tempting next step is auto-correcting your internal state to match the exchange, since the exchange is usually the more authoritative source. Resist doing this unconditionally. Auto-correction is safe for small, well-understood, high-confidence categories of break — for example, a one-lot rounding difference traced to a known lot-size conversion issue you have already root-caused and fixed going forward, where you are just clearing historical residue.

It is dangerous for anything novel or large, because auto-correcting silently removes the evidence you need to diagnose the underlying bug, and worse, if your reconciliation logic itself has a bug (a bad symbol mapping, a timing misalignment), auto-correction based on that flawed comparison can actively corrupt otherwise-correct internal state. A safer default: auto-correct only pre-approved, narrowly scoped categories of break, and route everything else to a human-reviewed queue with full context (the break details, recent order and fill history for that instrument, any recent connectivity events) attached automatically so the reviewer doesn't have to reconstruct that context manually.

Even for the narrow, pre-approved auto-correction categories, log every automated correction with the same rigor you'd apply to a manually reviewed one, including a snapshot of both prior states and the specific rule that triggered the correction. This audit trail matters for two distinct reasons: it lets you retrospectively verify the auto-correction logic itself is behaving as designed over time, and it gives you the evidence trail a regulator or auditor will eventually ask for when they want to understand exactly how and why a specific historical position value came to be what it is in your permanent records.

## 7. Reconciliation Across Multiple Venues

Firms trading across multiple exchanges or brokers face a compounded version of this problem: aggregate position reconciliation needs to correctly net positions that may be held at different venues under different conventions (some venues report net position, others report separate long and short positions that need netting on your side), and a single logical position might legitimately be split across venues at any moment during an active rebalancing or migration between brokers.

The design principle that scales here is maintaining venue-level reconciliation as the primary, most granular check — reconcile against each venue independently first — and only aggregate up to a firm-wide net position view as a secondary, derived check. If you reconcile only at the aggregate level, a compensating error (long 100 at venue A wrongly recorded, short 100 at venue B wrongly recorded, netting to a correct-looking zero) can hide completely, which is exactly the kind of break that aggregate-only reconciliation is structurally blind to.

Currency and asset-class boundaries introduce a related aggregation trap: netting positions denominated in different currencies into a single reporting-currency figure before reconciling can mask a genuine break in one currency that happens to be offset by an unrelated, coincidental discrepancy in another. Reconcile within each native currency and asset class first, and only convert to a common reporting currency for presentation after the underlying, granular reconciliation has already confirmed correctness at the level where the actual positions and cash movements live.

## 8. Building Reconciliation Into the Development Lifecycle

Reconciliation should not be a system you build once and leave alone; it needs to evolve alongside every change to your order and position logic. Any change to the OMS state machine, any new order type support, any new venue integration should ship with a corresponding update to reconciliation logic and, ideally, a specific test scenario exercising it. Treat "does this feature reconcile correctly" as a required review question for any change touching the order path, in the same category as "does this handle a dropped connection," because in practice these two questions are deeply related — reconciliation is precisely the safety net that catches what your connection-handling logic misses.

## Summary

- Reconciliation cannot eliminate state divergence, which is inherent to distributed systems across a network boundary; its job is to detect and bound it.
- Compare at least your internal state, the exchange's state, and where relevant a clearing/custodian view, since two agreeing sources can both still be wrong.
- Normalize symbol conventions and align timing carefully before diffing, or you will generate false breaks that erode trust in the system.
- Classify breaks by magnitude and persistence, and keep historical records to distinguish flapping timing noise from genuine, growing errors.
- Run continuous intraday reconciliation for active trading, not just end-of-day batch checks.
- Limit auto-remediation to narrow, well-understood break categories, and route everything else to a human with full context attached automatically.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
