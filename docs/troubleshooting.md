# Troubleshooting Guide: Operational Diagnostics Kit

## How to Use This Guide

When a validator returns FAIL, find the failing gate in the table below. The Remediation column describes the specific fix required. Reopen the artifact, apply the remediation, and rerun the validator in a new session.

Validators and generation are separate sessions. Don't embed fix attempts in your validation session.

---

## Diagnostic Context Record (DCR-{SERVICE}-{NNN})

This is a human-completed entry gate artifact. Fill all fields directly in the template before running the validator.

| Gate | What Failure Looks Like | Typical Cause | Remediation |
|------|------------------------|---------------|-------------|
| service_identification | Service name or system boundary absent | Scope not established before beginning investigation | Name the service, its downstream dependencies, and the scope boundary explicitly |
| symptom_description | Symptoms described as "Got an alert" without specifics | Symptom not documented at time of trigger | Record: the alert name, the threshold that fired, the exact time of trigger, and the initial user-facing impact observed |
| scope_bounded | Investigation scope undefined | All systems treated as implicitly in scope | Define which services, components, and time window are in scope for this investigation |
| evidence_documented | Observability evidence absent | Evidence not gathered at incident start | Document available signals: error rates, latency metrics, log excerpts, and alert states at time of investigation |
| recent_changes_documented | No recent changes recorded | Change context not consulted | Check the deployment log, feature flag changes, and configuration changes in the 48 hours before the incident |

---

## Investigation Record (INR-{SERVICE}-{NNN})

| Gate | What Failure Looks Like | Typical Cause | Remediation |
|------|------------------------|---------------|-------------|
| context_reference | No DCR ID referenced | INR started without a DCR | Record the DCR ID in §1; if the DCR is missing, create and freeze one before continuing |
| hypothesis_register | No hypotheses documented | Investigation conducted without tracking hypotheses | Document each hypothesis tested: what was suspected and what evidence would confirm or deny it |
| diagnostic_trail | Steps recorded without findings ("Checked logs" with no observed outcome) | Actions logged without capturing what they revealed | Record what was found for each diagnostic step: not just what was checked |
| probable_cause | Probable cause absent or stated as "unknown" | Investigation concluded without synthesizing findings | Synthesize findings into a probable cause statement; if evidence is insufficient, document what remains unknown and why further investigation is needed |
| immediate_actions | Actions taken to contain or resolve the incident not recorded | Operational actions not documented during incident | Record each action with its timestamp, the actor who performed it, and the observed outcome |
| open_questions | Open questions section absent | Investigation treated as fully resolved | List remaining unknowns that require further investigation or postmortem analysis; an empty section signals false closure |

---

## Postmortem Record (PMR-{SERVICE}-{NNN})

| Gate | What Failure Looks Like | Typical Cause | Remediation |
|------|------------------------|---------------|-------------|
| incident_reference | No INR or IR ID referenced | PMR disconnected from the investigation and incident records | Reference the frozen INR and IR IDs; the PMR synthesizes those records and must trace to them |
| timeline | Timeline less detailed than the INR or contains gaps | Timeline reconstructed from memory rather than INR | Reconstruct from INR §3 diagnostic trail; include detection, escalation, key actions, and resolution events |
| root_cause | Root cause stated at the symptom level only ("queue was full") | Systemic analysis absent | Apply 5 Whys or an equivalent technique; identify the systemic condition that allowed the failure, not just the immediate cause |
| slo_impact | SLO impact not quantified using SRP methodology | Impact described vaguely ("reliability degraded") | Calculate using SRP measurement methodology; state: the error budget consumed and which SLO window was affected |
| corrective_actions | Actions assigned to "the team" without individual owners | Individual ownership not established | Name a specific individual owner for each corrective action; include a target date and a measurable success criterion |
| lessons_learned | Lessons learned section generic ("improve monitoring") | Lessons written as aspirations rather than specific findings | Write specific lessons: what condition was not handled, what the system lacked, and what practice or change would prevent recurrence |

---

## Runbook (RB-{SERVICE}-{NNN}) [Optional]

| Gate | What Failure Looks Like | Typical Cause | Remediation |
|------|------------------------|---------------|-------------|
| trigger_conditions | Trigger stated vaguely ("high error rate") | Metric name and threshold absent | State: the exact metric name, the specific numeric threshold, and the evaluation window |
| resolution_steps | Steps not numbered, or contain conditional logic without explicit branch labels | Ambiguous procedure that relies on operator interpretation | Write numbered steps; for conditional paths, create explicit branch labels (e.g., "If condition A: go to Step 4. If condition B: go to Step 7") |
| verification | No verification step after resolution | Resolution assumed complete once steps are finished | Define how the operator confirms the issue is resolved: the specific metric or signal to check and the threshold that confirms health |
| escalation_conditions | No escalation path defined | Escalation left to operator judgment | Define: when to escalate (e.g., after N failed attempts, if condition X persists beyond Y minutes), and who to escalate to by name or role |
| evidence_grounded | Runbook procedure not traceable to INR or PMR | Procedure invented rather than derived from incident learning | Every procedure step must be traceable to an action that was taken and confirmed effective in the PMR §4 timeline |
