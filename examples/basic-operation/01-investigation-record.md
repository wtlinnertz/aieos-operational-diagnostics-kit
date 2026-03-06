# Investigation Record

---

## 1. Document Control

| Field | Value |
|-------|-------|
| INR ID | INR-NOTIF-001 |
| Date and Time | 2026-02-14 15:22 UTC |
| Status | Frozen |
| Governance Model Version | 1.0 |
| Prompt Version | 1.0 |

---

## 2. Context Reference

| Field | Value |
|-------|-------|
| DCR ID | DCR-NOTIF-001 |
| DCR Status | Frozen |
| Service | notification-service (TaskFlow notification delivery) |
| Environment | production |

---

## 3. Hypothesis Register

| # | Hypothesis | Evidence Required | Evidence Found | Status |
|---|-----------|------------------|---------------|--------|
| H-1 | Queue consumer goroutine is leaking — workers are not being cleaned up on cancellation, causing progressive reduction in active worker capacity | Worker goroutine count trending up; goroutine stack traces showing stuck workers; worker capacity metric declining | Goroutine count increased from baseline 48 to 1,847 over 53 minutes. pprof goroutine dump at 15:05 UTC showed 1,799 goroutines stuck at `notification.(*Consumer).processWithTimeout` awaiting context cancellation that never fires. Worker capacity at 6% of normal. | Confirmed |
| H-2 | Downstream email/SMS provider is throttling or failing — causing queue to back up with retries | Provider API error rate > 0%; provider status page; retry queue depth | SendGrid API returned HTTP 200 for all sampled test notifications at 14:47 UTC. SMS provider API health check healthy. Provider error rate: 0%. | Refuted |
| H-3 | Database connection pool exhaustion is blocking queue queries — consumers cannot fetch pending notifications | DB connection pool utilization metric; connection wait time | DB connection pool at 31% utilization (normal range). No connection wait time recorded. | Refuted |

---

## 4. Diagnostic Trail

| Timestamp | Action | Finding |
|-----------|--------|---------|
| 2026-02-14 14:35 UTC | Priya Sharma paged by PagerDuty alert notification-queue-depth-critical; confirmed alert and began investigation | Queue depth confirmed at 127,432; delivery success rate confirmed at 71.3% in Datadog |
| 2026-02-14 14:40 UTC | Queried notification-service logs in Datadog for errors in the 14:22–14:35 UTC window | No ERROR-level log entries. Found WARN entries: "context canceled: processWithTimeout exceeded 30s" — 847 occurrences between 14:22 and 14:35 UTC |
| 2026-02-14 14:47 UTC | Sent test notification to internal test account; monitored delivery | Test notification queued and delivered in 4,340ms (baseline 320ms); confirmed H-2 refuted — provider side is healthy, delay is in the queue processing |
| 2026-02-14 14:52 UTC | Checked database connection pool metrics in Datadog | Pool utilization at 31% (normal range 20–40%). Connection wait time: 0ms. H-3 refuted. |
| 2026-02-14 14:58 UTC | Checked notification-worker-count metric in Datadog (tracks active goroutine workers) | Worker goroutine count: 1,847 at 14:58 UTC. Expected baseline: ~48. Count has been rising steadily since 14:22 UTC at rate of ~35/minute |
| 2026-02-14 15:05 UTC | Triggered pprof goroutine dump via notification-service /debug/pprof/goroutine endpoint | 1,799 goroutines in state WAITING at `notification.(*Consumer).processWithTimeout` — specifically at the `select` waiting for ctx.Done() which is never signaled because context cancel is not being propagated correctly on timeout |
| 2026-02-14 15:12 UTC | Reviewed notification-service commit history to identify when processWithTimeout was last modified | processWithTimeout was refactored in v1.2.1 (deployed 2026-01-28) to add per-notification timeout. The refactor did not correctly propagate context cancellation from the parent context |
| 2026-02-14 15:17 UTC | Rolled back notification-service to v1.2.0 using `kubectl set image deployment/notification-service notification-service=taskflow/notification-service:v1.2.0` | Deployment rolled back; pods cycling to v1.2.0 |
| 2026-02-14 15:22 UTC | Monitored post-rollback metrics | Worker goroutine count declining from 1,847 toward baseline. Queue depth beginning to decline. Delivery success rate recovering — 89% at 15:22 UTC |

---

## 5. Probable Cause

**Probable Cause:**

Queue consumer goroutine leak introduced in v1.2.1. The `processWithTimeout` function added in v1.2.1 used a local timeout context but did not correctly propagate cancellation to child contexts when the per-notification timeout expired. This caused goroutines to remain stuck in `WAITING` state indefinitely, progressively reducing active worker capacity until the service could no longer keep pace with the inbound notification rate.

**Evidence Basis:**

- Diagnostic step at 15:05 UTC: pprof goroutine dump confirmed 1,799 goroutines stuck at the `processWithTimeout` `ctx.Done()` select
- Diagnostic step at 15:12 UTC: code review confirmed the refactor in v1.2.1 did not propagate context cancellation
- Diagnostic step at 14:58 UTC: goroutine count trend matched the symptom onset at 14:22 UTC exactly

---

## 6. Immediate Actions

**Actions Taken:**

| Time | Action | Taken By |
|------|--------|---------|
| 2026-02-14 15:17 UTC | Rolled back notification-service to v1.2.0 using `kubectl set image deployment/notification-service notification-service=taskflow/notification-service:v1.2.0` | Priya Sharma |
| 2026-02-14 15:22 UTC | Monitored metrics post-rollback | Priya Sharma |
| 2026-02-14 15:30 UTC | Confirmed queue draining and delivery recovery | Priya Sharma |

**Restoration Confirmed:**

At 15:30 UTC: notification_delivery_success_rate: 99.6% (within SLO target of 99.5%); notification_queue_pending_count: 847 (declining toward normal); worker goroutine count: 49 (baseline 48). Alert notification-queue-depth-critical cleared at 15:28 UTC. Incident declared resolved.

---

## 7. Open Questions

1. The goroutine leak was introduced in v1.2.1 deployed on 2026-01-28 — why did the incident not manifest until 2026-02-14 (17 days later)? The leak may require a specific combination of high notification volume and concurrent timeout events. The exact triggering condition is not established.
2. Are there other callsites in notification-service that use the same pattern as `processWithTimeout`? The code review was limited to the processWithTimeout function itself.

---

## 8. Freeze Declaration

By freezing this record, the author confirms:
- The DCR referenced is in Frozen status
- The hypothesis register reflects all hypotheses considered, including refuted ones
- The diagnostic trail is chronologically accurate and findings are substantive
- The probable cause is grounded in evidence
- Immediate actions and restoration status are accurately documented
- This record is complete and authorizes PMR generation to begin

**Frozen by:** Priya Sharma
**Freeze date:** 2026-02-14
**Status:** Frozen
