# Operational Diagnostics Kit — Test Plan

This document contains the structural integrity checks and flow scenario tests for the Operational Diagnostics Kit. These tests verify that the kit is complete, internally consistent, and capable of producing valid artifacts.

---

## Structural Integrity Checks

Structural checks verify that the kit's files are present, properly named, and internally consistent. These checks do not require AI — they are verifiable by inspection.

### S-01: Four-File Completeness

**Check:** Every governed artifact type has exactly four files: spec, template, prompt, validator.

| Artifact Type | Spec | Template | Prompt | Validator |
|---------------|------|----------|--------|-----------|
| DCR | docs/specs/dcr-spec.md | docs/artifacts/dcr-template.md | *(human-authored — no prompt)* | docs/validators/dcr-validator.md |
| INR | docs/specs/inr-spec.md | docs/artifacts/inr-template.md | docs/prompts/inr-prompt.md | docs/validators/inr-validator.md |
| PMR | docs/specs/pmr-spec.md | docs/artifacts/pmr-template.md | docs/prompts/pmr-prompt.md | docs/validators/pmr-validator.md |
| RB | docs/specs/rb-spec.md | docs/artifacts/rb-template.md | docs/prompts/rb-prompt.md | docs/validators/rb-validator.md |

**Expected result:** All files present.

*Note: The DCR is human-authored and does not have a generation prompt. This is consistent with the entry gate pattern (SRER in RRK, RER in REK, KER in EEK). The spec, template, and validator constitute three of the four files; the fourth is intentionally absent for human-authored entry gates.*

---

### S-02: Hard Gate Count Alignment

**Check:** Each spec's declared hard gate count matches the validator's gate checks.

| Artifact Type | Spec Hard Gates | Validator Gates |
|---------------|----------------|----------------|
| DCR | 5 | 5 |
| INR | 6 | 6 |
| PMR | 6 | 6 |
| RB | 5 | 5 |

**Expected result:** Counts match for all four artifact types.

---

### S-03: Hard Gate Name Alignment

**Check:** Gate names in specs match gate names in validators (exact string match for JSON output field names).

| Artifact | Spec Gate Names | Validator Gate Names |
|----------|----------------|---------------------|
| DCR | service_identification, symptom_description, scope_bounded, evidence_documented, recent_changes_documented | service_identification, symptom_description, scope_bounded, evidence_documented, recent_changes_documented |
| INR | context_reference, hypothesis_register, diagnostic_trail, probable_cause, immediate_actions, open_questions | context_reference, hypothesis_register, diagnostic_trail, probable_cause, immediate_actions, open_questions |
| PMR | incident_reference, timeline, root_cause, slo_impact, corrective_actions, lessons_learned | incident_reference, timeline, root_cause, slo_impact, corrective_actions, lessons_learned |
| RB | trigger_conditions, resolution_steps, verification, escalation_conditions, evidence_grounded | trigger_conditions, resolution_steps, verification, escalation_conditions, evidence_grounded |

**Expected result:** All gate names match exactly.

---

### S-04: Prompt-to-Spec Reference Integrity

**Check:** Each generation prompt references the correct spec and template. No prompt inlines content rules.

| Prompt | References Spec | References Template | Inlines Rules? |
|--------|----------------|--------------------|----|
| inr-prompt.md | docs/specs/inr-spec.md | docs/artifacts/inr-template.md | No |
| pmr-prompt.md | docs/specs/pmr-spec.md | docs/artifacts/pmr-template.md | No |
| rb-prompt.md | docs/specs/rb-spec.md | docs/artifacts/rb-template.md | No |

**Expected result:** All prompts reference correct spec and template; no inlined rules.

---

### S-05: Validator-to-Spec Reference Integrity

**Check:** Each validator references its spec as the source of truth. Validators do not reference prompts.

| Validator | References Spec | References Prompt? |
|-----------|-----------------|-------------------|
| dcr-validator.md | docs/specs/dcr-spec.md | No |
| inr-validator.md | docs/specs/inr-spec.md | No |
| pmr-validator.md | docs/specs/pmr-spec.md | No |
| rb-validator.md | docs/specs/rb-spec.md | No |

**Expected result:** All validators reference the correct spec; none reference prompts.

---

### S-06: Template Section Alignment

**Check:** Each template's section headings match the required sections listed in the corresponding spec.

| Artifact | Spec Required Sections | Template Sections |
|----------|----------------------|-------------------|
| DCR | Document Control, Service Identification, Symptom Description, Scope and Impact, Observable Signals, Recent Changes, Freeze Declaration | §1–§7 (all present) |
| INR | Document Control, Context Reference, Hypothesis Register, Diagnostic Trail, Probable Cause, Immediate Actions, Open Questions, Freeze Declaration | §1–§8 (all present) |
| PMR | Document Control, Incident Reference, Timeline, Root Cause Analysis, SLO Impact, Corrective Actions, Lessons Learned, Cross-Kit Actions, Freeze Declaration | §1–§9 (all present) |
| RB | Document Control, Failure Class, Trigger Conditions, Pre-Conditions, Resolution Steps, Verification, Escalation Conditions, Evidence Grounding, Version History | §1–§9 (all present) |

**Expected result:** All template sections match spec required sections.

---

### S-07: Utility Prompt Identification

**Check:** Utility prompts are clearly identified as non-governed (no spec, no validator, no template). No utility prompt is referenced as a governed artifact anywhere in the kit.

| Utility Prompt | Non-Governed Label Present | Has Spec? | Has Validator? |
|----------------|--------------------------|-----------|----------------|
| hypothesis-generation-prompt.md | Yes ("utility prompt — does not produce a governed artifact") | No | No |
| diagnostic-checkpoint-prompt.md | Yes ("utility prompt — does not produce a governed artifact") | No | No |
| pattern-match-prompt.md | Yes ("utility prompt — does not produce a governed artifact") | No | No |
| dcr-srp-consistency-check-prompt.md | Yes ("utility prompt — does not produce a governed artifact") | No | No |

**Expected result:** All utility prompts are labeled non-governed; none have a spec or validator.

---

### S-08: Example Artifact Coverage

**Check:** The worked example covers all four artifact types with corresponding validator outputs showing PASS.

| Example File | Artifact Type | ID | Validator Output | Status |
|-------------|---------------|----|-----------------|--------|
| examples/basic-operation/00-diagnostic-context-record.md | DCR | DCR-NOTIF-001 | validator-outputs/dcr-validation.json | PASS |
| examples/basic-operation/01-investigation-record.md | INR | INR-NOTIF-001 | validator-outputs/inr-validation.json | PASS |
| examples/basic-operation/02-postmortem-record.md | PMR | PMR-NOTIF-001 | validator-outputs/pmr-validation.json | PASS |
| examples/basic-operation/03-runbook.md | RB | RB-NOTIF-001 | validator-outputs/rb-validation.json | PASS |

**Expected result:** All four artifact types represented; all four validator outputs present with PASS status.

---

## Flow Scenario Tests

Flow scenarios verify that the kit's artifacts, when produced in order with appropriate inputs, pass validation. These tests require AI execution.

---

### F-00: SEV2 Investigation Flow

**Scenario:** SEV2 incident declared → complete DCR → check patterns → generate INR → validate → freeze → generate PMR → validate → freeze → generate RB.

**Setup:**
- Declare a SEV2 incident for a service with a frozen SRP (e.g., notification-service with SRP-NOTIF-001)
- Complete the DCR using the template with specific symptoms, scoped impact, at least one observable signal, and explicit recent changes statement
- Optionally run pattern-match-prompt.md against prior PMRs
- Gather incident evidence: responder notes, alert records, metric data, log excerpts
- Validate DCR → freeze
- Generate INR using the INR prompt with frozen DCR + evidence
- Validate INR → freeze (within 24 hours of resolution)
- Gather post-incident evidence and allow settling period (24–48 hours)
- Generate PMR using the PMR prompt with frozen INR + SRP + post-incident evidence
- Validate PMR → freeze
- Generate RB using the RB prompt with frozen PMR + INR (if PMR §8 recommends RB)
- Validate RB → freeze

**Expected outcomes:**
- DCR: all 5 gates PASS; severity from enumerated list; specific symptoms; scope declared; at least one signal with source; recent changes documented
- INR: all 6 gates PASS; DCR confirmed Frozen; hypothesis register with at least one hypothesis; diagnostic trail with at least two timestamped steps; probable cause stated or "Undetermined" with reasoning
- PMR: all 6 gates PASS; INR confirmed Frozen; timeline covers first signal to resolution; root cause with contributing factors; SLO impact quantified; corrective actions with named owners, specific dates, tracking references; at least one lesson
- RB: all 5 gates PASS; trigger conditions specific and observable; resolution steps numbered and specific; verification with observable criterion; escalation conditions with target; evidence grounding with artifact IDs

**Key gate to verify:** INR Gate 3 (diagnostic_trail) — confirm findings are substantive, not "checked X — nothing found." INR Gate 4 (probable_cause) — confirm evidence reference is present, not just a statement.

---

### F-01: Runbook Generation Flow

**Scenario:** A second incident of the same failure class occurs → DCR identifies the pattern → pattern-match-prompt confirms prior PMR match → investigation is accelerated using the prior RB → new INR and PMR generated → RB updated to v2.

**Setup:**
- Second incident with similar trigger conditions (same alert names, similar metric values)
- Complete DCR for the new incident
- Run pattern-match-prompt.md against PMR-NOTIF-001 — expect "Strong match" result
- Apply RB-NOTIF-001 trigger condition check — trigger conditions met
- Execute RB resolution steps; investigation is accelerated
- Generate INR for the new incident (even if the RB was applied directly — the INR documents the investigation trail, even if brief)
- Generate PMR
- Assess whether the RB procedure was accurate or requires updates
- If RB requires updates: generate RB v2 using the RB prompt with new PMR as evidence

**Expected outcomes:**
- Pattern match assessment: Strong match with PMR-NOTIF-001; RB-NOTIF-001 marked as Applicable
- INR: all 6 gates PASS; notes in diagnostic trail that RB was applied; confirms which RB steps produced expected outcomes
- PMR: all 6 gates PASS; §8 Cross-Kit Actions includes RB recommendation if steps were changed or if the procedure requires updating
- RB v2 (if generated): trigger_conditions PASS; resolution_steps PASS; evidence_grounded references both the original INR/PMR and the new PMR

**Key gate to verify:** RB evidence_grounded gate — for v2, confirm both the original evidence (INR/PMR-NOTIF-001) and the new incident evidence (new PMR ID) are referenced.

---

## Notes

- All structural checks (S-01 through S-08) should be verified before running flow scenarios.
- Flow scenarios F-00 and F-01 cover the two most important operational patterns.
- Example artifacts (examples/basic-operation/) are the reference implementation; they should pass all validators without modification.
- Structural checks can be run by inspection against the file system. Flow scenarios require AI execution using the session setups in docs/how-to-use-with-ai.md.
- Additional scenarios may be added as new patterns are identified in production use (e.g., abort protocol, re-entry after correction).
