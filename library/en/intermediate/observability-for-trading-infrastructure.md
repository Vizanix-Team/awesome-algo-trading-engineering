# Observability for Trading Infrastructure

*By Vizanix — Intermediate Level*

> Design monitoring, logging, and alerting for a trading system so that when something breaks at 9:31am, you know within seconds, not from a client's phone call.

![diagram](../../assets/latency-histogram.svg)

## Table of Contents

1. Why Generic Observability Advice Falls Short for Trading
2. The Three Pillars, Adapted for Trading Systems
3. Metrics That Actually Matter
4. Structured Logging for Post-Mortem Reconstruction
5. Distributed Tracing Across the Order Lifecycle
6. Alerting Without Alert Fatigue
7. Dashboards for Different Audiences
8. Building a Culture of Observability

## 1. Why Generic Observability Advice Falls Short for Trading

Standard observability guidance from web engineering — track request latency, error rate, and throughput, alert on anomalies — is necessary but insufficient for trading infrastructure. A web service that silently drops 0.1% of requests degrades user experience. A trading system that silently drops 0.1% of fill notifications produces a wrong position, and a wrong position produces wrong risk decisions, which can produce real financial loss well before anyone notices the underlying technical fault.

The core difference is that trading systems have domain-specific correctness invariants that generic infrastructure monitoring cannot see. Your dashboards need to answer not just "is the service up" but "does our believed position match the exchange's position," "is our risk engine's view of exposure current," and "did every order we sent receive either a fill, a reject, or an explicit cancel confirmation." These are business-logic health checks layered on top of, not instead of, standard infrastructure metrics.

This distinction between infrastructure health and business correctness also changes who your observability tooling actually serves. A generic APM dashboard is built for an engineer debugging a service. A trading observability system needs to serve that same engineer during an incident, but it also needs to serve a trader deciding in real time whether to keep a strategy running, and a risk manager deciding whether current exposure is acceptable given what the monitoring is showing. Designing for all three audiences from a shared, correctly instrumented data source, rather than building each view as an afterthought bolted onto infrastructure-only tooling, is a genuinely different design exercise than standard observability engineering.

## 2. The Three Pillars, Adapted for Trading Systems

Metrics, logs, and traces are the standard three pillars, and each needs a trading-specific lens. Metrics should include not just system resource usage but domain counters: orders sent per second, fill rate, reject rate by reason code, reconciliation break count. Logs need to be structured and correlated by order ID and correlation ID so you can reconstruct the exact sequence of events for any single order across every service it touched. Traces need to follow an order from strategy decision through risk check, through the OMS, through the exchange gateway, and back — because latency or failure at any one hop has different remediation than at another, and without tracing you are stuck guessing which hop was the actual bottleneck.

A useful mental model: infrastructure observability tells you whether your machine is healthy; business observability tells you whether your trading is correct. You need both, instrumented separately, because a machine can be perfectly healthy (low CPU, low latency, no errors in the logs) while still producing subtly wrong trading outcomes due to a logic bug that never throws an exception.

The tracing layer specifically benefits from being designed around the order lifecycle as its organizing unit, rather than around individual service calls in isolation, which is the more typical default in general-purpose distributed tracing setups. A trace that spans "strategy decision through final settlement" for a single order, even though it touches many services and may span minutes or hours rather than the milliseconds typical of a web request trace, gives you a single coherent narrative object to reason about, rather than a collection of technically-linked but conceptually disjointed spans that require manual stitching together during an investigation.

## 3. Metrics That Actually Matter

Beyond the obvious (CPU, memory, network, GC pauses for managed runtimes), instrument these trading-specific metrics as first-class citizens, not afterthoughts bolted on post-incident:

Order-to-ack latency: time from your system sending an order to receiving exchange acknowledgment. A creeping increase here often precedes an exchange gateway issue by minutes, giving you lead time to react before it becomes visible in fill rates.

Reconciliation break count and age: the number of positions or orders where your internal state disagrees with the exchange's reported state, and how long each break has been open. This should be near zero at all times during market hours; any sustained nonzero value is a correctness emergency, not a performance concern.

Market data staleness: time since the last update per instrument per feed. A feed that silently stops updating (rather than erroring loudly) is one of the most dangerous and common failure modes in trading infrastructure, because downstream strategies keep computing on data that looks valid but is frozen.

```
def record_market_data_age(instrument, last_update_ts, now):
    age_ms = (now - last_update_ts).total_seconds() * 1000
    metrics.gauge("market_data.staleness_ms", age_ms, tags={"instrument": instrument})
    if age_ms > STALENESS_THRESHOLD_MS:
        metrics.increment("market_data.stale_alert", tags={"instrument": instrument})
```

Risk limit utilization: how close each strategy or desk is to its configured limits, tracked continuously, not just checked at the moment an order is submitted. Watching this trend over the session lets a human operator intervene before a limit breach forces an automated, possibly disruptive, response.

Queue depth and processing lag across every internal message queue in your pipeline also deserves dedicated tracking, since a queue that's growing rather than draining is a leading indicator of a downstream bottleneck well before it manifests as a customer-visible symptom like delayed order acknowledgment. Alert on the rate of change of queue depth, not just its absolute level, because a queue that's small but growing steadily is a more urgent problem than one that's large but stable, and a threshold-only alert on absolute depth misses this distinction entirely, often paging you only after the backlog has already become severe enough to affect trading outcomes.

## 4. Structured Logging for Post-Mortem Reconstruction

Free-text log lines like `"Order rejected: insufficient funds"` are nearly useless for automated analysis and painful for manual post-mortems at 2am. Structure every log entry as a set of key-value fields with a consistent schema, always including a correlation ID that ties together every log line related to a single order or a single strategy decision across every service boundary.

```
{
  "timestamp": "2026-09-27T13:41:02.118Z",
  "level": "ERROR",
  "event": "order_rejected",
  "correlation_id": "ord-7f3a2c",
  "instrument": "ESZ6",
  "reason_code": "INSUFFICIENT_MARGIN",
  "strategy_id": "momentum_v3",
  "requested_qty": 50,
  "service": "exchange-gateway-cme"
}
```

This structure lets you build a query that reconstructs the complete lifecycle of any order in seconds: filter every log line across every service by `correlation_id`, sort by timestamp, and you have an exact, ordered narrative of what happened, which is exactly what you need during an incident review or a client dispute about an execution.

Retention matters too. Trading logs often need to be retained far longer than typical application logs for regulatory or dispute-resolution reasons, and the storage design should account for that from the start rather than being retrofitted after a compliance request for six-month-old data that was already purged.

Design your log schema with versioning in mind from the outset, since the fields you need to capture will grow as your system evolves, and a schema-less free-for-all where different services emit inconsistent field names for conceptually identical data (one service logs `order_id`, another logs `orderId`, a third logs `oid`) actively defeats the cross-service correlation this entire structured-logging exercise is meant to enable. A shared logging library, enforced across every service that touches the order path, with a small number of required standard fields and clear conventions for extension fields, prevents this drift far more reliably than a style guide nobody consistently follows under deadline pressure.

## 5. Distributed Tracing Across the Order Lifecycle

An order in a modern trading system typically passes through many independent services: a strategy engine, a pre-trade risk check, an OMS, a smart order router, an exchange gateway, and back through fill processing and post-trade risk. Distributed tracing instruments each hop with a span, all linked under a single trace ID, so you can visualize exactly where time is spent and where failures occur.

```
def send_order(order):
    with tracer.start_span("risk_check", trace_id=order.trace_id) as span:
        result = risk_engine.check(order)
        span.set_tag("risk_result", result.status)
    if result.approved:
        with tracer.start_span("oms_submit", trace_id=order.trace_id):
            oms.submit(order)
```

The payoff is concrete: when a client asks why their order took 400 milliseconds longer than usual to acknowledge, a trace immediately shows whether that time was spent in your risk check, waiting for an internal queue, or in the network hop to the exchange, rather than requiring an engineer to manually correlate timestamps across five separate log files.

## 6. Alerting Without Alert Fatigue

An alert that fires constantly and gets ignored is worse than no alert, because it trains your on-call engineers to reflexively dismiss notifications, including the one time it matters. Tier your alerts explicitly: page-worthy (wake someone up, financial or correctness risk is active right now), urgent-but-not-paging (needs attention within the hour, post to a monitored channel), and informational (log it, review during business hours, never interrupt anyone).

Reconciliation breaks, risk limit breaches, and market data feed outages during market hours belong in the page-worthy tier without exception. A single failed retry on an otherwise-healthy connection, or a latency metric briefly crossing a soft threshold, belongs several tiers down. Review your alert-to-actual-incident ratio periodically and prune alerts that fire often but rarely correspond to real action taken — that ratio drifting upward over time is the leading indicator of an alerting system nobody trusts anymore.

Alert routing should also account for time of day and market state explicitly, not just severity. A latency degradation during active market hours on a heavily traded instrument deserves a different response urgency than the identical degradation on the same instrument overnight when nothing is trading. Building this context-awareness into your alerting logic, rather than applying uniform thresholds regardless of market state, meaningfully reduces the volume of alerts that technically meet a threshold but carry no real urgency given the actual conditions at the time they fired.

## 7. Dashboards for Different Audiences

A single dashboard cannot serve a trader, an infrastructure engineer, and a risk manager equally well, and trying to build one universal dashboard usually produces something too cluttered for any of them. Build role-specific views: a trading desk dashboard emphasizing live P&L, position, and fill rates per strategy; an infrastructure dashboard emphasizing latency percentiles, queue depths, and error rates per service; a risk dashboard emphasizing limit utilization and reconciliation status across the whole firm.

Keep the underlying metrics store unified even as the dashboards diverge — you want one source of truth queried differently for different audiences, not three separate pipelines that can silently drift out of agreement with each other, which would defeat the entire purpose of having reliable observability in the first place.

## 8. Building a Culture of Observability

Tooling alone does not produce good observability; the team's habits do. Make instrumenting new code a required part of the definition of done for any change that touches the order path, not an optional nice-to-have added later. Run regular game days where you simulate a specific failure (a stale market data feed, a delayed exchange ack, a reconciliation break) and verify your alerting and dashboards actually surface it the way you expect, before you need that surfacing during a real incident.

Every post-mortem should end with at least one concrete observability improvement, not just a process fix — if an incident took thirty minutes to diagnose because a specific piece of information was not visible anywhere, that is itself the bug to fix, independent of whatever caused the original incident.

Track time-to-detect and time-to-diagnose as explicit, tracked metrics across your incident history, separate from time-to-resolve. A shrinking time-to-detect trend tells you your alerting is improving; a stubbornly flat or growing time-to-diagnose trend, even as time-to-detect improves, tells you your team has gotten faster at knowing something is wrong without getting any faster at knowing what to do about it, which is exactly the gap that better tracing, better structured logging, and better runbooks are meant to close, and measuring it explicitly keeps the team honest about whether those investments are actually paying off.

## Summary

- Layer business-logic health checks (reconciliation breaks, feed staleness, risk utilization) on top of standard infrastructure metrics; neither alone is sufficient.
- Use structured, correlation-ID-tagged logging so any order's full lifecycle can be reconstructed with one query across every service.
- Instrument distributed traces across every hop an order takes, from strategy decision to exchange ack and back.
- Tier alerts deliberately and prune ones that fire often without producing real action, to protect the value of paging.
- Build role-specific dashboards on a single unified metrics store rather than one dashboard trying to serve everyone.
- Treat instrumentation as a required part of shipping order-path code, and let every post-mortem produce at least one observability improvement.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
