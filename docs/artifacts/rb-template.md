# Runbook

---

## 1. Document Control

| Field | Value |
|-------|-------|
| RB ID | RB-{SERVICE}-{NNN} |
| Version | v{N} |
| Date | {YYYY-MM-DD} |
| Status | Draft / Validated / Frozen |
| Governance Model Version | 1.0 |
| Prompt Version | {version of rb-prompt.md used} |

---

## 2. Failure Class

| Field | Value |
|-------|-------|
| Failure Class Name | {specific name — e.g., "Notification queue exhaustion due to downstream API timeout"} |
| Affected Service | {service name} |
| Source Incident | {PMR-{SERVICE}-{NNN}} |

**Description:**

{1–3 sentences describing the failure class — what fails, under what conditions, and what the observable impact is.}

---

## 3. Trigger Conditions

{Conditions indicating this runbook applies. At least two required. Each must be specific and observable — an on-call engineer can confirm or deny from monitoring without investigation.}

All of the following must be true to apply this runbook:

1. {specific observable condition — e.g., "Alert 'notification-queue-depth-critical' is firing in the production monitoring system"}
2. {specific observable condition — e.g., "notification_queue_pending_count metric exceeds 50,000 in the current measurement window"}
3. {additional condition if applicable}

---

## 4. Pre-Conditions

{Conditions that must be true before executing this runbook, or state "None — runbook applies immediately when trigger conditions are met."}

- {pre-condition — e.g., "No active deployment in the last 30 minutes (confirm in deployment log)"}
- {pre-condition}

---

## 5. Resolution Steps

{Numbered steps. Each step must specify what to do and what to observe to confirm the step succeeded. Show commands where applicable.}

1. **{Step name}**

   Action: {what to do — specific}

   Command (if applicable):
   ```
   {command or command pattern}
   ```

   Expected outcome: {what you should observe if this step succeeds}

2. **{Step name}**

   Action: {what to do — specific}

   Expected outcome: {what you should observe}

3. **{Step name}**

   Action: {what to do — specific}

   Expected outcome: {what you should observe}

4. {additional steps as needed}

---

## 6. Verification

{Confirm that the failure condition has been resolved. At least one verification step with an observable criterion.}

**Verification steps:**

1. {what to check} → Expected: {specific metric value, alert resolved, or observable state}
2. {what to check} → Expected: {specific result}

**Resolution declared when:** {the specific observable state that confirms full resolution}

---

## 7. Escalation Conditions

{Conditions under which to stop executing this runbook and escalate. At least one required with observable criterion and escalation target.}

Escalate instead of continuing if:

- {observable condition — e.g., "Queue depth has not decreased after 10 minutes of executing Step 3"}
- {observable condition — e.g., "Downstream API error rate exceeds 50% — this runbook does not address upstream provider failures"}

**Escalation target:** {who to escalate to — name, channel, or pager}

---

## 8. Evidence Grounding

{Reference at least one INR or PMR that informed this runbook. Include artifact ID and what the runbook draws from it.}

| Artifact | ID | What This Runbook Draws From It |
|----------|----|---------------------------------|
| Investigation Record | INR-{SERVICE}-{NNN} | {what diagnostic steps or findings informed this runbook} |
| Postmortem Record | PMR-{SERVICE}-{NNN} | {what root cause or corrective action findings informed this runbook} |

---

## 9. Version History

| Version | Date | What Changed | Authorized By |
|---------|------|-------------|---------------|
| v1 | {YYYY-MM-DD} | Initial version | {name} |
