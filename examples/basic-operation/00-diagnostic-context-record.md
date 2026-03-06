# Diagnostic Context Record

---

## 1. Document Control

| Field | Value |
|-------|-------|
| DCR ID | DCR-NOTIF-001 |
| Date and Time | 2026-02-14 14:35 UTC |
| Severity | SEV2 |
| Incident Owner | Priya Sharma |
| Status | Frozen |
| Governance Model Version | 1.0 |
| Prompt Version | N/A |

---

## 2. Service Identification

| Field | Value |
|-------|-------|
| Service Name | notification-service (TaskFlow notification delivery) |
| Environment | production |
| SRP Reference | SRP-NOTIF-001 v1 |

---

## 3. Symptom Description

**Incident Start Time:** 2026-02-14 14:22 UTC

**Symptoms:**

- Notification delivery success rate dropped from 99.7% to 71.3% beginning at 14:22 UTC — a sustained 28-percentage-point decline
- Notification delivery latency (p99) increased from 320ms baseline to 4,200ms — a 13x increase
- Affected notifications are queued but not delivered; they are not being dropped or erroring — they are stuck in pending state

---

## 4. Scope and Impact

**Impact Scope:** subset of users

**Scope Details:**

Approximately 29% of users who triggered notification events between 14:22 UTC and the time of this DCR are experiencing delayed notifications. Notifications are not lost — they are accumulating in the pending queue. Users who triggered notifications before 14:22 UTC and users who have not triggered notifications in this window are unaffected.

---

## 5. Observable Signals

| Signal | Source | Value / Detail |
|--------|--------|---------------|
| notification-queue-depth-critical alert | Datadog (PagerDuty) | Fired at 14:35 UTC; queue depth: 127,432 pending — threshold is 100,000 |
| notification_delivery_success_rate metric | Datadog | Current: 71.3%; baseline: 99.7% |
| notification_queue_pending_count metric | Datadog | Current: 127,432; normal: < 2,000 |

---

## 6. Recent Changes

No changes in last 72h.

---

## 7. Freeze Declaration

By freezing this record, the incident owner confirms:
- The service, environment, and incident owner are correctly identified
- The symptoms are described specifically and the start time is accurate
- At least one observable signal is documented with its source
- Recent changes in the last 72 hours are documented — confirmed "no changes in last 72h"
- This record is complete and authorizes INR generation to begin

**Frozen by:** Priya Sharma
**Freeze date:** 2026-02-14
**Status:** Frozen
