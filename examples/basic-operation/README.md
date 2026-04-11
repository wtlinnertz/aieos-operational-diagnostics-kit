# Worked Example: TaskFlow Notification Service Queue Exhaustion Incident

This directory contains a complete worked example of an ODK engagement for a SEV2 incident affecting the TaskFlow notification service. The incident was a notification queue exhaustion event caused by goroutine leak during context cancellation.

---

## Incident Summary

**Service:** TaskFlow Notification Service
**Severity:** SEV2
**Duration:** 47 minutes
**Root Cause:** Queue consumer goroutine leak during HTTP context cancellation, causing progressive queue growth until workers were unable to keep pace with inbound rate.

---

## Artifact Sequence

| File | Artifact | ID | Status |
|------|----------|----|--------|
| `00-diagnostic-context-record.md` | Diagnostic Context Record | DCR-NOTIF-001 | Frozen |
| `01-investigation-record.md` | Investigation Record | INR-NOTIF-001 | Frozen |
| `02-postmortem-record.md` | Postmortem Record | PMR-NOTIF-001 | Frozen |
| `03-runbook.md` | Runbook | RB-NOTIF-001 | Frozen |

---

## Validator Outputs

| File | Artifact | Result |
|------|----------|--------|
| `validator-outputs/dcr-validation.json` | DCR-NOTIF-001 | PASS |
| `validator-outputs/inr-validation.json` | INR-NOTIF-001 | PASS |
| `validator-outputs/pmr-validation.json` | PMR-NOTIF-001 | PASS |
| `validator-outputs/rb-validation.json` | RB-NOTIF-001 | PASS |

---

## Context

This example continues the TaskFlow notification service scenario used across AIEOS kits:

- **PIK**: DPRD defined the notification delivery feature
- **EEK**: Implementation produced the notification-service v1.2.3
- **REK**: v1.2.3 released with 25% canary exposure
- **RRK**: SRP-NOTIF-001 defined SLOs; IR-NOTIF-001 captured the incident
- **ODK**: DCR-NOTIF-001 through RB-NOTIF-001 document the structured diagnostic engagement

The RRK IR-NOTIF-001 and ODK PMR-NOTIF-001 are complementary — the IR is the lightweight operational record; the PMR is the organizational learning artifact.

---

## Key Design Decisions Illustrated

1. **DCR is sparse but valid** — The DCR was completed under time pressure with limited information. It passes all 5 hard gates with specific symptoms, scoped impact, one observable signal, and explicit "no changes in last 72h" confirmation.

2. **INR shows two refuted hypotheses** — Not every hypothesis in a real investigation is confirmed. The INR documents the full investigation trail including dead ends.

3. **PMR §8 recommends RB generation** — The failure class is repeatable (queue exhaustion from goroutine leak is a known failure mode), so the PMR authorizes a runbook.

4. **RB is executable, not hypothetical** — Every resolution step traces to documented actions in INR-NOTIF-001. Commands are shown with actual service names.
