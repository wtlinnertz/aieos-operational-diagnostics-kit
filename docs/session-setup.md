# Session Setup: Operational Diagnostics Kit

Use this file to set up an AI session for each ODK artifact. Find the section for the artifact you're generating or validating. Follow the checklist before starting.

**Rule:** Generate and validate in separate sessions. Do not self-validate.

**ODK trigger:** ODK is entered when a SEV1/2 incident is declared (required) or when a lower-severity incident has high organizational learning value (operator judgment). This is a reactive operational track: it is triggered by production events, not SDLC progression.

**ODK sequence:** DCR (human-authored, under pressure) → INR (investigation, during/after incident) → PMR (postmortem, after incident is resolved) → RB (optional, for recurring failure classes).

---

## Diagnostic Context Record (DCR-{SERVICE}-{NNN})

**What you're creating:** The scope declaration for the investigation: what service is affected, what symptoms are observed, what the investigation boundary is, and what evidence is available. Written during or immediately after incident declaration.

**Note:** The DCR is human-authored under time pressure. Completeness improves after the incident is contained, but the DCR must be created at incident start to scope the investigation.

**Required Inputs (confirm before starting):**
- [ ] Declared incident (SEV1/2 or equivalent): Active or recently resolved?
- [ ] Observable symptoms: Alert name, threshold, time, and initial user impact known?
- [ ] Optional: Frozen SRP: Available? (provides SLO baseline and known failure modes)
- [ ] Optional: Frozen Release Record §7: Available? (provides recent changes context)

**Pre-Flight Gate Check (verify before completing):**
- [ ] `service_identification`: Service name and system boundary defined
- [ ] `symptom_description`: Alert name, threshold, exact trigger time, and user-facing impact documented
- [ ] `scope_bounded`: Which services, components, and time window are in scope: stated explicitly
- [ ] `evidence_documented`: Available signals (error rates, metrics, log excerpts, alert states) recorded
- [ ] `recent_changes_documented`: Deployment log, feature flag changes, and config changes checked for the 48h window before incident

**Session Setup:**
1. Use: `docs/artifacts/dcr-template.md`: fill manually, no prompt
2. Validate in a separate session: `docs/validators/dcr-validator.md`
3. After DCR is validated and frozen, proceed to INR generation

**Common Failure to Avoid:**
Observable signals section stating only "Got an alert" without specifics: record the alert name, the threshold that was breached, the exact timestamp, and the initial user-facing symptom.

---

## Investigation Record (INR-{SERVICE}-{NNN})

**What you're creating:** A structured documentation of the investigation: hypotheses, diagnostic steps, findings, probable cause, and immediate actions taken. Generated during or immediately after incident containment.

**Required Inputs (confirm before starting):**
- [ ] Frozen DCR: Frozen?
- [ ] Investigation findings: Available? (hypothesis log, diagnostic steps, observations)
- [ ] Optional: Frozen SRP: Available? (failure mode context)
- [ ] Optional: Frozen SAD (System Architecture Document from EEK): Available? (dependency context)

**Pre-Flight Gate Check (verify before generating):**
- [ ] `context_reference`: Frozen DCR ID on hand to reference in §1
- [ ] `hypothesis_register`: Each hypothesis tested is documentable (what was suspected, what evidence was gathered)
- [ ] `diagnostic_trail`: Each diagnostic step has a substantive finding (not just "checked X")
- [ ] `probable_cause`: Can state a probable cause or explicitly document why cause is not yet known
- [ ] `immediate_actions`: All containment and mitigation actions are documented with timestamps
- [ ] `open_questions`: Remaining unknowns that require postmortem analysis are identified

**Session Setup:**
1. Load: `docs/prompts/inr-prompt.md`
2. Provide: Frozen DCR content
3. Provide: Investigation notes: structured as: hypothesis, steps taken, findings per step
4. Provide: Optional frozen SRP and SAD if available
5. Provide: `docs/specs/inr-spec.md` (or confirm it is in context)
6. Validate in a separate session: `docs/validators/inr-validator.md`

**Common Failure to Avoid:**
Diagnostic steps recorded as actions without findings ("Checked logs", "Looked at metrics"): record what was found at each step: specific values, anomalies, or absence of expected signals.

---

## Postmortem Record (PMR-{SERVICE}-{NNN})

**What you're creating:** The organizational learning artifact: root cause analysis, SLO impact, corrective actions with named owners, and lessons learned. Generated after the incident is fully resolved.

**Note:** PMR requires the incident to be resolved and the INR to be frozen. Do not generate PMR during active incident response.

**Required Inputs (confirm before starting):**
- [ ] Frozen INR: Frozen?
- [ ] Frozen IR (from RRK): Frozen?
- [ ] Incident fully resolved: Yes?
- [ ] Optional: Frozen SRP: Available? (for SLO impact calculation using measurement methodology)

**Pre-Flight Gate Check (verify before generating):**
- [ ] `incident_reference`: Frozen INR ID and frozen IR ID on hand to reference
- [ ] `timeline`: Timeline can be reconstructed from INR §3 diagnostic trail + IR timeline
- [ ] `root_cause`: Systemic root cause (not just proximate) is understood and articulable
- [ ] `slo_impact`: SLO impact calculable using SRP measurement methodology
- [ ] `corrective_actions`: Each action has a named individual owner and target date
- [ ] `lessons_learned`: Specific, actionable lessons identified (not generic "improve monitoring")

**Session Setup:**
1. Load: `docs/prompts/pmr-prompt.md`
2. Provide: Frozen INR and frozen IR content
3. Provide: Optional frozen SRP (for SLO impact methodology)
4. Provide: `docs/specs/pmr-spec.md` (or confirm it is in context)
5. Validate in a separate session: `docs/validators/pmr-validator.md`
6. After PASS: route corrective actions to EEK (code/design changes) or RRK (SRP revision) as appropriate

**Common Failure to Avoid:**
Corrective actions assigned to "the team": each action must name a specific individual owner with a target date and a success criterion.

---

## Runbook (RB-{SERVICE}-{NNN}) [Optional]

**What you're creating:** A codified resolution procedure for a known, recurring failure class: with specific trigger conditions, numbered steps, verification, and escalation path. Generate only when the failure class is expected to recur.

**Decision rule:** Generate a Runbook only if the failure class identified in the PMR is expected to recur and would benefit from a defined, executable procedure. Do not generate a Runbook for one-off failures.

**Required Inputs (confirm before starting):**
- [ ] Frozen PMR: Frozen?
- [ ] Optional: Frozen INR: Available? (resolution steps reference)
- [ ] Optional: Frozen SRP: Available? (trigger metric names and thresholds)
- [ ] Failure class identified: Recurring? Expected to recur?

**Pre-Flight Gate Check (verify before generating):**
- [ ] `trigger_conditions`: Specific metric name, threshold value, and evaluation window are definable
- [ ] `resolution_steps`: Steps are procedurally derived from PMR §4 timeline (grounded in evidence)
- [ ] `verification`: Observable signal that confirms resolution is identifiable
- [ ] `escalation_conditions`: When to escalate and who to escalate to: defined
- [ ] `evidence_grounded`: Every resolution step is traceable to an action that worked in the PMR

**Session Setup:**
1. Load: `docs/prompts/rb-prompt.md`
2. Provide: Frozen PMR (the primary source for procedure grounding)
3. Provide: Frozen INR if available (diagnostic steps context)
4. Provide: Frozen SRP if available (metric names and thresholds for trigger conditions)
5. Provide: `docs/specs/rb-spec.md` (or confirm it is in context)
6. Validate in a separate session: `docs/validators/rb-validator.md`

**Runbook versioning:** Runbooks are versioned. When resolution procedures change based on new incident experience, increment the version and document what changed.

**Common Failure to Avoid:**
Trigger conditions stated vaguely ("high error rate"): define with precision: exact metric name, specific threshold, and evaluation window (e.g., "notification-service error rate exceeds 5% for 5 consecutive minutes on the 1-minute SLI").
