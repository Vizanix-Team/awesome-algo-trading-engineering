# Designing an Order Management System from Scratch

*By Vizanix — Intermediate Level*

> Learn how to architect an order management system that stays correct under concurrency, network failure, and exchange weirdness, not just under a demo.

![diagram](../../assets/system-architecture.svg)

## Table of Contents

1. What an OMS Actually Owns
2. The Order State Machine
3. Modeling Orders, Fills, and Amendments
4. Concurrency: One Order, Many Writers
5. Talking to Exchanges Without Lying to Yourself
6. Persistence and Crash Recovery
7. Position and P&L Bookkeeping
8. Testing an OMS Like You Mean It

## 1. What an OMS Actually Owns

An order management system is not a REST wrapper around an exchange API. It is the single source of truth for "what did we intend to do, what did we ask the market to do, and what actually happened." Those three things drift apart constantly, and your OMS's entire job is to keep them reconciled.

Think of your OMS as owning three ledgers simultaneously. The first is intent: a strategy calls `submit_order()` and expects that call to mean something durable, even if the process crashes a millisecond later. The second is the wire protocol: what you sent to the exchange, what acknowledgments came back, and in what order. The third is truth: fills, cancels, and rejects as the exchange sees them, which you only learn about asynchronously.

A common design mistake is conflating these three. Teams build a single `Order` object, mutate it in place from multiple threads, and call it done. That works until a cancel request races an unsolicited fill, and now your object holds status `CANCELLED` while a fill sits unaccounted for in a log file somewhere. The reader who takes one lesson from this chapter should take this one: separate the record of what you tried from the record of what happened, and reconcile them explicitly rather than assuming they match.

Your OMS also owns idempotency. Every order you send to an exchange needs a client-generated identifier that survives retries. If your network call times out, you do not know if the exchange received it. Resending with the same client order ID and checking the exchange's response for "already exists" is the only sane way to avoid duplicate orders. Skip this and you will eventually double an order size during a network blip, typically on the worst possible day for it.

## 2. The Order State Machine

Model every order as a finite state machine, not a bag of booleans. A minimal but honest state set looks like this:

```
PENDING_NEW -> NEW -> PARTIALLY_FILLED -> FILLED
                 |            |
                 v            v
           PENDING_CANCEL  PENDING_CANCEL
                 |            |
                 v            v
             CANCELLED    CANCELLED (with residual fill)
                 |
                 v
              REJECTED (from PENDING_NEW only)
```

The key discipline is that transitions are one-directional and explicit. You never let application code just set `order.status = "CANCELLED"`. Instead you call `order.apply_event(CancelAckEvent(...))` and let the state machine validate whether that transition is legal from the current state. If it is not — say, a cancel ack arrives for an order already in `FILLED` — you log it as an anomaly rather than silently overwriting state. This single guardrail catches an enormous fraction of the bugs that would otherwise surface as "why does our position not match the exchange."

Pending states deserve special attention. `PENDING_NEW` means you sent the order but have not received acknowledgment. `PENDING_CANCEL` means you sent a cancel but the order could still fill before the cancel lands — a genuine race that exists on every exchange with any queuing delay. Your state machine must allow a fill event to arrive while in `PENDING_CANCEL` and handle it correctly, updating filled quantity before finalizing the cancel.

## 3. Modeling Orders, Fills, and Amendments

Keep the order and its fills as separate entities linked by identifier, never flatten a fill into a single mutable "filled quantity" field with no history. You want an append-only fill log:

```
Fill {
  order_id, exec_id, price, quantity,
  side, timestamp_exchange, timestamp_received,
  liquidity_flag  // maker/taker
}
```

`exec_id` is your deduplication key. Exchanges resend fill notifications after reconnects, and if you sum quantities blindly, you will double-count. Always check `exec_id` against what you have already applied before incrementing filled quantity.

Amendments (quantity or price changes on a live order) are trickier than most engineers expect, because different exchanges implement them differently. Some treat an amend as cancel-then-replace with a new order ID; others mutate the existing order in place and may or may not reset queue priority. Your OMS needs an exchange-specific adapter layer that normalizes these into a common internal event, rather than leaking exchange quirks into your core state machine.

## 4. Concurrency: One Order, Many Writers

In production, an order's state can be touched by at least three independent actors: the strategy thread requesting a cancel, the market data / drop-copy thread applying a fill, and a reconciliation job comparing against an exchange position snapshot. If these three write to the same in-memory object without discipline, you get torn reads and lost updates.

The cleanest pattern is a single-writer actor model per order (or per order-book-of-orders keyed by instrument), where all mutations go through one serialized event queue. Every event — submit, cancel request, fill, reject, amend ack — gets pushed onto that queue and applied sequentially by one thread that owns the order's memory. Readers get a consistent snapshot; writers never race.

```
class OrderActor:
    def __init__(self, order_id):
        self.state = OrderStateMachine()
        self.queue = SingleConsumerQueue()

    def enqueue(self, event):
        self.queue.push(event)   # thread-safe push

    def run(self):
        while True:
            event = self.queue.pop()   # single consumer
            self.state.apply(event)
            self.publish_state_change()
```

This costs you some throughput versus a naive shared-memory approach, but it buys correctness you cannot get any other way without extremely careful lock design, and lock-based order state machines are notoriously easy to get subtly wrong.

## 5. Talking to Exchanges Without Lying to Yourself

Your OMS should never trust its own optimistic state as ground truth for risk decisions. Build an explicit "confirmed vs assumed" distinction. When you send a new order, you can optimistically decrement available buying power immediately (so a fast strategy loop does not overspend before the ack arrives), but you must track that decrement as provisional and reverse it cleanly if the order is rejected.

Sequence numbers matter more than most people assume before their first outage. If an exchange feed sends events with a monotonic sequence number, track the last sequence you processed and detect gaps immediately. A gap means you missed a fill or a cancel ack, and continuing to trade on stale state is how positions silently diverge from reality over a trading day. Better to pause new order flow on that instrument and force a snapshot resync than to keep trading blind.

## 6. Persistence and Crash Recovery

Every state transition needs to hit durable storage before you consider it "applied," or you will lose orders on crash. Write-ahead logging is the right mental model: append the event to a durable log first, then update in-memory state, then acknowledge upstream. On restart, replay the log to rebuild state before accepting any new orders.

Recovery also means reconnecting to exchanges and reconciling. On startup, request current open orders and positions from every connected exchange and diff them against your recovered local state. Any mismatch — an order you think is live that the exchange says is filled, or vice versa — goes into a manual review queue rather than being silently auto-corrected, at least until your reconciliation logic has proven itself over months of production use.

## 7. Position and P&L Bookkeeping

Positions should be derived, not stored as an independently mutated field. Compute position as a fold over your fill log: sum signed quantity by instrument. This makes your position auditable — you can always answer "why do we think we are long 500 shares" by pointing at the exact fills that produced that number, rather than trusting a counter that might have drifted due to a missed decrement somewhere.

Realized P&L needs a consistent cost-basis method (FIFO is the simplest to reason about and audit) applied uniformly. Unrealized P&L requires a mark price, and you should timestamp which mark you used, because "what was our P&L at 14:32:07" is a question you will eventually need to answer precisely during a dispute or a post-mortem.

## 8. Testing an OMS Like You Mean It

Unit tests on the state machine are necessary but nowhere near sufficient. Build a deterministic exchange simulator that can inject reordered acknowledgments, duplicate fill notifications, dropped connections mid-order, and partial fills followed by late rejects. Replay these scenarios against your real OMS code, not a mock, and assert that final state always matches an independently computed expected state.

Property-based testing pays off here: generate random sequences of valid exchange events (respecting the state machine's legal transitions) and assert invariants hold after every event — position never goes negative on quantity that was never bought, filled quantity never exceeds order quantity, no fill is ever double-counted. These invariant checks catch classes of bugs that scenario-specific tests miss entirely.

## Summary

- Separate intent, wire protocol, and confirmed truth into distinct ledgers; never conflate them into one mutable object.
- Model orders as explicit state machines with validated transitions, including pending states for the race between cancel and fill.
- Use `exec_id` deduplication on fills and a single-writer actor model per order to avoid concurrency bugs.
- Treat sequence-number gaps in exchange feeds as a hard stop signal, not something to silently paper over.
- Persist every state transition via write-ahead logging before acknowledging it upstream, and reconcile against exchange state on every restart.
- Derive positions and P&L from an append-only fill log so they stay auditable rather than trusted blindly.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
