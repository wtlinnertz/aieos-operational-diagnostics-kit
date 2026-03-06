# Diagnostic Principles

**Version:** 1.0

These principles define the organizational policy for incident investigation and postmortem analysis. They govern how this organization approaches structured diagnosis of production failures. All ODK artifact generation sessions must include this file.

---

## §1 Investigation Principles

**1.1 Structure before action.** Before taking diagnostic actions, establish a hypothesis. Unstructured "try things" investigation wastes time and can obscure evidence. Every diagnostic step should be preceded by a hypothesis it is testing.

**1.2 Evidence over assumption.** All claims about the cause of an incident must be grounded in observable evidence — metric data, log entries, alert records, or responder observations. Assumptions are hypotheses, not findings.

**1.3 Capture the trail.** The investigation trail — every hypothesis considered, every diagnostic step taken, every piece of evidence gathered — is as valuable as the conclusion. The trail enables postmortem analysis and future runbook creation.

**1.4 State uncertainty explicitly.** If the probable cause cannot be determined with confidence, say so explicitly. "Undetermined" with reasoning is more valuable than a confident claim without evidence.

**1.5 Preserve evidence.** Do not delete logs, screenshots, or alert records during or after an incident. Evidence that seems irrelevant during investigation may be critical during postmortem analysis.

---

## §2 Postmortem Principles

**2.1 Blameless by default.** Postmortem analysis focuses on systemic factors, not individual fault. When a person made an error, the analysis asks: "What conditions made this error possible?" not "Who made this error?"

**2.2 RCA is not over when service is restored.** Service restoration confirms the immediate fix worked. Root cause analysis requires understanding why the failure occurred, which requires evidence gathering and analysis after resolution — not during.

**2.3 Corrective actions must be actionable.** A corrective action with no named owner and no target date is not a corrective action — it is a wish. Every action must have an owner, a deadline, and a tracking reference.

**2.4 Lessons must be specific.** "Improve monitoring" is not a lesson. "Add a metric to track queue depth at 90% capacity, not 100%, to enable earlier detection of queue exhaustion" is a lesson.

**2.5 SLO impact is not optional.** Every postmortem must quantify the SLO impact of the incident, even if that impact was zero. If no SLOs were affected, document why and what conditions prevented impact.

---

## §3 Runbook Principles

**3.1 Runbooks document what worked.** A runbook is not a design document for how things should work — it documents the actual steps that resolved an actual incident. The INR and PMR are the source; the RB distills their findings.

**3.2 Runbooks must be executable.** Every step must be specific enough for an engineer unfamiliar with the service to execute it under pressure. Vague steps ("check the database") are not acceptable. Specific steps ("query SELECT count(*) FROM notification_queue WHERE status='pending' and check against SRP threshold") are.

**3.3 Runbooks expire if unvalidated.** A runbook that has not been executed or reviewed in more than 6 months may not reflect current system state. Add a Last Validated date and establish a review cadence.

**3.4 Escalation conditions are required.** Every runbook must define the conditions under which the responder should stop executing the runbook and escalate. A runbook without escalation conditions can delay appropriate escalation.

**3.5 Evidence grounding is required.** A runbook must reference the INR or PMR that informed it. This preserves traceability between the documented procedure and the real incidents it was derived from.

---

## §4 Severity Definitions

All ODK artifacts use the following severity levels. These must match the severity classifications used in RRK Incident Records.

| Severity | Criteria |
|----------|---------|
| SEV1 | Complete service outage affecting all users, or data loss or corruption affecting any users |
| SEV2 | Significant degradation affecting a material subset of users, or SLO breach in progress |
| SEV3 | Partial degradation or performance issue with workaround available; no SLO breach |
| SEV4 | Minor issue affecting a small number of users or internal users only; no production impact |

ODK engagement is required for SEV1 and SEV2. Operator judgment applies for SEV3 and SEV4.

---

## §5 Timing Standards

| Artifact | Target Completion |
|----------|------------------|
| DCR | Within 2 hours of incident declaration |
| INR | Within 24 hours of incident resolution |
| PMR | Within 5 business days of incident resolution |
| RB (if warranted) | Within 10 business days of PMR freeze |

These are targets, not hard gates. Circumstances may require deviation; document deviations in the artifact.
