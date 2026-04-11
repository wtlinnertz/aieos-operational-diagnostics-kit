# Runbook

---

## 1. Document Control

| Field | Value |
|-------|-------|
| RB ID | RB-NOTIF-001 |
| Version | v1 |
| Date | 2026-02-21 |
| Status | Frozen |
| Governance Model Version | 1.0 |
| Prompt Version | 1.0 |

---

## 2. Failure Class

| Field | Value |
|-------|-------|
| Failure Class Name | Notification queue exhaustion due to goroutine leak in queue consumer |
| Affected Service | notification-service (TaskFlow notification delivery) |
| Source Incident | PMR-NOTIF-001 |

**Description:**

The notification-service queue consumer leaks goroutines when per-notification processing times out but the timeout context's cancellation is not propagated correctly. Over time, goroutine count grows to the point where active worker capacity is insufficient to process the inbound notification rate, causing queue depth to grow and delivery success rate to drop. Notifications are queued but not delivered; they are not lost.

---

## 3. Trigger Conditions

All of the following must be true to apply this runbook:

1. Alert `notification-queue-depth-critical` is firing in Datadog (queue depth > 100,000)
2. `notification_queue_pending_count` metric in Datadog exceeds 50,000 and is trending upward (not declining)
3. `notification_delivery_success_rate` metric in Datadog is below 95% (not just a transient dip)

---

## 4. Pre-Conditions

- Confirm no active deployment of notification-service in the last 30 minutes (check deployment log or ask the on-call engineer who was paged)
- Confirm downstream providers are healthy: check SendGrid status at status.sendgrid.com and Twilio status at status.twilio.com — if either is degraded, this runbook does not apply (the failure class is different)

---

## 5. Resolution Steps

1. **Check goroutine count**

   Action: Open the notification-service Datadog dashboard and check the `notification_worker_goroutine_count` metric for the last 2 hours.

   Expected outcome: Goroutine count is trending upward from a baseline of approximately 48. A count above 200 with an upward trend confirms the goroutine leak.

   If goroutine count is NOT elevated: This runbook does not apply. Proceed to escalation (see §7).

2. **Take a pprof goroutine dump**

   Action: SSH to any running notification-service pod and run:
   ```
   kubectl exec -n production $(kubectl get pods -n production -l app=notification-service -o jsonpath='{.items[0].metadata.name}') -- curl -s http://localhost:6060/debug/pprof/goroutine?debug=1 | head -100
   ```

   Expected outcome: Output shows many goroutines in `WAITING` state at `notification.(*Consumer).processWithTimeout`. If the count of goroutines at this stack frame is > 50, goroutine leak is confirmed.

3. **Check the current notification-service version**

   Action: Run the following to check the deployed version:
   ```
   kubectl get deployment/notification-service -n production -o jsonpath='{.spec.template.spec.containers[0].image}'
   ```

   Expected outcome: If version is v1.2.1 or any version between v1.2.1 and the fixed version (v1.2.2+), rollback is the correct mitigation. If version is v1.2.2 or later, the root cause is different — this runbook does not apply.

4. **Initiate rollback to v1.2.0**

   Action: Roll back to the last known-good version:
   ```
   kubectl set image deployment/notification-service -n production notification-service=taskflow/notification-service:v1.2.0
   ```

   Expected outcome: kubectl confirms `deployment.apps/notification-service image updated`. Pods begin cycling within 30 seconds.

5. **Monitor rollback progress**

   Action: Watch pod rollout:
   ```
   kubectl rollout status deployment/notification-service -n production
   ```

   Expected outcome: `deployment "notification-service" successfully rolled out` within 2–3 minutes. Goroutine count in Datadog begins declining within 5 minutes of rollout completing.

---

## 6. Verification

**Verification steps:**

1. Check `notification_delivery_success_rate` in Datadog → Expected: rising toward 99.5% or above within 10 minutes of rollback completion
2. Check `notification_worker_goroutine_count` in Datadog → Expected: declining from peak and trending toward baseline (~48) within 10 minutes of rollback
3. Check `notification_queue_pending_count` in Datadog → Expected: declining within 5 minutes of rollback (the queue should drain as goroutines are restored)
4. Confirm alert `notification-queue-depth-critical` clears → Expected: alert resolves within 15 minutes of rollback

**Resolution declared when:** `notification_delivery_success_rate` is at or above 99.5% for 5 consecutive minutes, queue depth is below 5,000, and the goroutine count is below 100.

---

## 7. Escalation Conditions

Escalate instead of continuing if:

- Goroutine count is NOT elevated at Step 1 (below 100 with no upward trend) — this runbook does not apply; a different failure class is causing the queue exhaustion
- Downstream provider (SendGrid or Twilio) status pages show an active incident — this runbook does not apply; the failure class is downstream provider degradation, not goroutine leak
- Queue depth has not declined within 15 minutes of rollback completing — rollback may not have resolved the issue; a different version or a different root cause may be involved
- `notification_delivery_success_rate` has not recovered above 90% within 20 minutes of rollback — escalate to investigate whether there is a secondary failure

**Escalation target:** Notification service on-call (PagerDuty escalation policy: `notification-service-p2`) or Marcus Chen directly if unavailable.

---

## 8. Evidence Grounding

| Artifact | ID | What This Runbook Draws From It |
|----------|----|---------------------------------|
| Investigation Record | INR-NOTIF-001 | Trigger conditions (§3 signals observed at 14:35 UTC); diagnostic steps 1–2 (goroutine count check and pprof dump at 14:58 and 15:05 UTC); resolution step (rollback at 15:17 UTC); verification criteria (recovery observed at 15:22–15:30 UTC) |
| Postmortem Record | PMR-NOTIF-001 | Root cause (processWithTimeout goroutine leak in v1.2.1); pre-conditions (downstream provider check from §4 Contributing Factor analysis); known-good version (v1.2.0 identified as rollback target) |

---

## 9. Version History

| Version | Date | What Changed | Authorized By |
|---------|------|-------------|---------------|
| v1 | 2026-02-21 | Initial version | Priya Sharma |
