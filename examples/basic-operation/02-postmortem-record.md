# Postmortem Record

---

## 1. Document Control

| Field | Value |
|-------|-------|
| PMR ID | PMR-NOTIF-001 |
| Date | 2026-02-17 |
| Status | Frozen |
| Governance Model Version | 1.0 |
| Prompt Version | 1.0 |

---

## 2. Incident Reference

| Field | Value |
|-------|-------|
| INR ID | INR-NOTIF-001 |
| INR Status | Frozen |
| Severity | SEV2 |
| Incident Duration | 68 minutes (14:22 UTC declaration-equivalent to 15:30 UTC confirmed restoration) |
| Service | notification-service (TaskFlow notification delivery) |

---

## 3. Timeline

| Timestamp | Event | Actor |
|-----------|-------|-------|
| 2026-02-14 14:22 UTC | First observable signal: notification delivery success rate begins declining (99.7% → declining) | Datadog monitoring |
| 2026-02-14 14:35 UTC | Alert notification-queue-depth-critical fired (queue depth: 127,432 — threshold: 100,000); Priya Sharma paged via PagerDuty | PagerDuty / Priya Sharma |
| 2026-02-14 14:35 UTC | Incident declared SEV2; investigation begun | Priya Sharma |
| 2026-02-14 14:40 UTC | Logs checked; 847 WARN entries "context canceled: processWithTimeout exceeded 30s" found between 14:22–14:35 UTC | Priya Sharma |
| 2026-02-14 14:47 UTC | Provider test: SendGrid and SMS provider confirmed healthy; H-2 (downstream provider) refuted | Priya Sharma |
| 2026-02-14 14:52 UTC | DB connection pool checked; pool at 31% utilization; H-3 (DB exhaustion) refuted | Priya Sharma |
| 2026-02-14 14:58 UTC | Worker goroutine count confirmed at 1,847 (baseline 48); goroutine leak identified as probable cause | Priya Sharma |
| 2026-02-14 15:05 UTC | pprof goroutine dump taken; 1,799 goroutines stuck at processWithTimeout ctx.Done() | Priya Sharma |
| 2026-02-14 15:12 UTC | Code review of processWithTimeout; v1.2.1 refactor identified as source of goroutine leak | Priya Sharma |
| 2026-02-14 15:17 UTC | Mitigation: rollback to notification-service v1.2.0 initiated via kubectl | Priya Sharma |
| 2026-02-14 15:22 UTC | Post-rollback monitoring: goroutine count declining; queue draining; delivery rate recovering | Priya Sharma |
| 2026-02-14 15:28 UTC | notification-queue-depth-critical alert cleared | Datadog |
| 2026-02-14 15:30 UTC | Service restored: delivery success rate 99.6%; queue depth 847 and declining; incident declared resolved | Priya Sharma |

---

## 4. Root Cause Analysis

### Primary Root Cause

The `processWithTimeout` function introduced in notification-service v1.2.1 (deployed 2026-01-28) created per-notification timeout contexts but did not correctly propagate cancellation to child contexts when the timeout expired. When a per-notification timeout fired, the goroutine handling that notification entered `WAITING` state at `ctx.Done()` — a state it never exited because the parent context's cancellation was not propagated to the local context. Each timeout event leaked one goroutine permanently. Over 17 days, goroutine count grew slowly; on 2026-02-14, high notification volume combined with elevated per-notification processing time caused the rate of goroutine leak to exceed the service's ability to sustain delivery throughput.

### Contributing Factors

1. **No goroutine count monitoring alert**: The notification-service had no alert on worker goroutine count. The goroutine leak was invisible — detectable only by checking the worker-count metric manually. Had a goroutine count alert existed, the slow-growing leak would have been detected days before impact.

2. **No pre-deploy goroutine leak test**: The v1.2.1 release did not include a long-running integration test that would have detected goroutine accumulation. The bug was not triggered by unit tests because the leak only manifests under timeout conditions with sustained load.

3. **Context propagation pattern not documented**: The processWithTimeout refactor used a non-idiomatic context propagation pattern. No code review guideline or architectural standard documented the correct pattern for nested context cancellation in this service.

### Systemic Factors

The goroutine leak pattern (creating child contexts without propagating cancellation from parent contexts) is a common Go concurrency mistake. The organization does not have a shared library or linter rule that catches this pattern. While the immediate cause is a specific code bug, the systemic factor is the absence of a standard pattern for timeout-with-cancellation in Go that is enforced or easily discoverable.

---

## 5. SLO Impact

SRP-NOTIF-001 v1 defines:
- **Notification Delivery SLO**: 99.5% success rate, 30-day rolling window (43,200 minutes)
- **Error budget at incident start**: ~80% remaining (estimate based on prior RHR period data)

| SLO Name | Budget Consumed (Time) | Budget Consumed (% of Window) | Remaining Budget | Policy Implication |
|----------|----------------------|------------------------------|-----------------|-------------------|
| Notification Delivery Success Rate | 68 minutes | 0.16% of 43,200-minute window | ~79.84% remaining | Within SRP §3 slow-burn threshold; no error budget alert triggered |

Note: the 29% delivery degradation affected only the fraction of notifications that were queued during the 68-minute window. The actual SLO consumption is the aggregate success rate below 99.5% over the window, not the full 68 minutes at 71.3%. The above calculation uses the incident window duration as a conservative proxy; actual consumption may be lower depending on the exact minute-by-minute success rate data.

---

## 6. Corrective Actions

| Action | Owner | Target Date | Tracking Reference | Root Cause Basis |
|--------|-------|-------------|-------------------|-----------------|
| Fix processWithTimeout context cancellation in notification-service; issue v1.2.2 with correct context propagation; add unit test for timeout-with-cancellation behavior | Marcus Chen | 2026-02-21 | ENG-4471 | §4 Primary Root Cause |
| Add goroutine count alert to notification-service Datadog dashboard: alert when worker goroutine count exceeds 100 (2x expected baseline of 48) for more than 5 minutes | Priya Sharma | 2026-02-19 | ENG-4472 | §4 Contributing Factor 1 |
| Add goroutine leak detection to notification-service integration test suite: long-running test that verifies goroutine count does not grow under sustained timeout conditions | Marcus Chen | 2026-02-28 | ENG-4473 | §4 Contributing Factor 2 |
| Document Go context propagation standard for timeout-with-cancellation pattern in the engineering wiki; circulate to backend engineering channel | Priya Sharma | 2026-03-07 | ENG-4474 | §4 Contributing Factor 3 |

---

## 7. Lessons Learned

1. **Goroutine leak detection requires active monitoring, not just alerting on symptoms**: The delivery success rate alert fired when the service was severely degraded. A goroutine count alert would have fired days earlier when the count first exceeded baseline — providing time to investigate before impact. Silent accumulation of leaked resources is the characteristic failure mode of goroutine leaks; monitoring the resource directly, not just the symptom, is required.

2. **A 17-day latency between bug introduction and incident does not mean the bug is not in production**: The goroutine leak was present from 2026-01-28 but did not cause visible impact until specific traffic conditions on 2026-02-14. Slow-growing resource leaks are particularly dangerous because they pass smoke testing and short-term monitoring. Integration tests must run long enough to expose accumulation patterns.

3. **The pprof goroutine dump was the fastest path to root cause**: The 8-minute time from "probable cause suspected" (14:58 UTC) to "root cause confirmed" (15:05 UTC) was enabled by the pprof endpoint being accessible in production. This capability should be documented in runbooks and onboarding materials as a first-line tool for Go service investigations, not an advanced technique.

---

## 8. Cross-Kit Actions

**EEK Escalation (code defect requiring engineering work):**
Applicable — the root cause is a code defect in notification-service, a system governed by the Engineering Execution Kit. Corrective action ENG-4471 (fix processWithTimeout and issue v1.2.2) requires engineering work. Escalation record: Trigger 1 (SEV2 code defect), PMR-NOTIF-001, defect: incorrect context cancellation propagation in processWithTimeout function, affected system: notification-service. Send to EEK team with PMR-NOTIF-001 as context.

**PIK Pattern (recurring failure class warranting discovery re-engagement):**
Not applicable — this is the first occurrence of this specific failure class. Monitor for recurrence; if the same root cause class (resource leak not detected by existing monitoring) appears in three consecutive RHR periods, escalate to PIK.

**RB Recommendation (repeatable failure with known resolution):**
Applicable — queue exhaustion due to goroutine leak is a recognizable failure class with a clear diagnostic procedure (check goroutine count, take pprof dump, check processWithTimeout) and a known immediate mitigation (rollback). Generate RB-NOTIF-001 for "Notification queue exhaustion due to goroutine leak." Resolution steps trace to INR-NOTIF-001 §6.

---

## 9. Freeze Declaration

By freezing this record, the author confirms:
- The INR referenced is in Frozen status
- The timeline is chronologically accurate and covers from first signal to restoration
- The root cause analysis reflects investigation findings, not speculation
- SLO impact is quantified
- At least one corrective action has a named owner, specific date, and tracking reference
- At least one lesson is specific, actionable, and traceable
- Cross-kit actions are addressed

**Frozen by:** Priya Sharma
**Freeze date:** 2026-02-17
**Status:** Frozen
