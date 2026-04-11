# Operational Diagnostics Kit — Playbook

This playbook defines the end-to-end process for the Operational Diagnostics Kit (Layer 8). It covers how to move from a declared incident to governed diagnostic artifacts: Diagnostic Context Record, Investigation Record, Postmortem Record, and optionally a Runbook.

---

## Artifact Flow

```
Trigger: SEV1/2 incident declared (or operator judgment)
         │
         ▼
Step 0: Diagnostic Context Record (DCR) — human-authored
         │ Validate → Freeze
         ▼
         [Optional: Run pattern-match-prompt.md to check for known failure signatures]
         [Optional: Run dcr-srp-consistency-check-prompt.md to validate against SRP]
         [Optional: Run hypothesis-generation-prompt.md to seed the hypothesis register]
         │
         ▼
Step 1: Investigation Record (INR) — generated
         │ [During investigation: use diagnostic-checkpoint-prompt.md if stalled]
         │ Validate → Freeze
         ▼
Step 2: Postmortem Record (PMR) — generated (after resolution)
         │ Validate → Freeze
         │
         ▼ (if PMR §8 recommends RB)
Step 3: Runbook (RB) — generated, optional
         │ Validate → Freeze
         ▼
Cross-kit outputs: PMR → RRK (next RHR), EEK (corrective actions if code defect), PIK (if recurring pattern), IEK (ER §8)
```

Steps 1–2 repeat for each incident that meets the trigger criteria. The frozen DCR for each incident is the constant reference point for that incident's INR and PMR.

---

## Trigger

ODK is triggered by:

| Trigger | Required / Optional |
|---------|-------------------|
| SEV1 incident declared | Required |
| SEV2 incident declared | Required |
| SEV3 or lower with significant learning value | Optional — operator judgment |
| Recurring failure class identified in RHR | Optional — operator judgment |

Refer to `docs/principles/diagnostic-principles.md §4` for severity definitions.

---

## Upstream Interface

**Upstream inputs (optional but recommended):**

| Source | Artifact | How Used |
|--------|----------|---------|
| RRK | Frozen SRP | Service baseline, SLO targets, known failure modes → DCR §SRP Reference and INR context |
| RRK | Past IRs | Known failure patterns → hypothesis generation |
| REK | Frozen RR §7 | Recent changes, deployment state → DCR §Recent Changes context |
| EEK | Frozen SAD | Service dependencies, architecture → INR investigation context |

None of these inputs are mandatory for DCR creation. The DCR can be authored from incident evidence alone. Upstream artifacts enhance diagnosis quality.

---

## Step 0 — Diagnostic Context Record

**Artifact:** Diagnostic Context Record (DCR)
**Type:** Human-authored entry gate
**Spec:** `docs/specs/dcr-spec.md` (5 hard gates)
**Template:** `docs/artifacts/dcr-template.md`
**Intake:** `docs/artifacts/incident-intake-template.md`
**Validator:** `docs/validators/dcr-validator.md`

### Purpose

The DCR confirms that enough context exists to begin structured diagnosis. It prevents investigation from beginning in a context vacuum — without a specific service named, specific symptoms described, and at least one observable signal documented.

### Inputs
- Incident declaration or equivalent trigger
- Completed Incident Intake Form (recommended; not required)
- Optional: Frozen SRP, Frozen RR §7

### Process

1. The incident owner completes the Incident Intake Form to gather DCR inputs.
2. Using the DCR template, the owner completes all sections: document control (severity, owner), service identification, symptom description (specific), scope and impact, observable signals (with sources), and recent changes (or explicit "no changes in last 72h").
3. The completed DCR is validated in a separate session using `docs/validators/dcr-validator.md`.
4. On PASS: the incident owner freezes the DCR. **No INR generation may begin until the DCR is Frozen.**
5. On FAIL: address blocking issues and re-validate. DCR authoring must not be blocked by information gathering — mark gaps explicitly and gather missing information before INR generation.

### Timing

Target: DCR completed within 2 hours of incident declaration.

### Freeze Points
- DCR must be Frozen before INR generation begins

---

## Step 1 — Investigation Record

**Artifact:** Investigation Record (INR)
**Type:** Generated
**Spec:** `docs/specs/inr-spec.md` (6 hard gates)
**Template:** `docs/artifacts/inr-template.md`
**Prompt:** `docs/prompts/inr-prompt.md`
**Validator:** `docs/validators/inr-validator.md`

### Purpose

The INR documents the active diagnostic trail: every hypothesis considered, every diagnostic step taken, every piece of evidence gathered, and the probable cause conclusion.

### Pre-Investigation Utility Prompts (Optional)

Before beginning INR generation, consider:
- **Pattern Match** (`docs/prompts/pattern-match-prompt.md`) — Compare DCR against past PMRs/RBs for known failure signatures. If a strong match exists, the prior INR's diagnostic trail can inform the investigation.
- **DCR → SRP Consistency Check** (`docs/prompts/dcr-srp-consistency-check-prompt.md`) — Validate DCR service against frozen SRP baseline.
- **Hypothesis Generation** (`docs/prompts/hypothesis-generation-prompt.md`) — Seed the hypothesis register with structured candidate causes before investigation begins.

### Inputs
- Frozen DCR
- Incident evidence gathered during investigation: alert records, metric data, log excerpts, responder notes
- Optional: Frozen SRP, Frozen SAD

### Process

1. Gather incident evidence as the investigation proceeds (alert records, monitoring data, log excerpts, responder notes).
2. If investigation stalls, use `docs/prompts/diagnostic-checkpoint-prompt.md` to assess convergence and get a structured next-step recommendation.
3. After resolution is declared (or after sufficient evidence is gathered for an in-progress incident), collect all evidence.
4. In a new AI session, provide: frozen DCR + incident evidence + `docs/specs/inr-spec.md` + `docs/artifacts/inr-template.md`. Use the INR prompt (`docs/prompts/inr-prompt.md`).
5. Review the generated INR. Confirm that:
   - The hypothesis register reflects all hypotheses actually considered (including refuted ones)
   - The diagnostic trail is chronologically accurate and findings are substantive
   - The probable cause is grounded in evidence
6. Validate in a separate session using `docs/validators/inr-validator.md`.
7. On PASS: review owner freezes the INR. **No PMR generation may begin until the INR is Frozen.**
8. On FAIL: address blocking issues; regenerate or correct; re-validate.

### Timing

Target: INR completed within 24 hours of incident resolution.

### Freeze Points
- INR must be Frozen before PMR generation begins

---

## Step 2 — Postmortem Record

**Artifact:** Postmortem Record (PMR)
**Type:** Generated (post-resolution)
**Spec:** `docs/specs/pmr-spec.md` (6 hard gates)
**Template:** `docs/artifacts/pmr-template.md`
**Prompt:** `docs/prompts/pmr-prompt.md`
**Validator:** `docs/validators/pmr-validator.md`

### Purpose

The PMR is the organizational learning artifact. It synthesizes the investigation findings into an authoritative record: timeline, root cause, SLO impact, corrective actions, and lessons learned. It is written after the incident is resolved and the investigation is complete.

### Wait Period

Allow a brief settling period (typically 24–48 hours) after resolution before authoring the PMR. This allows post-incident review discussion to complete and initial speculation to be replaced with confirmed findings.

### Inputs
- Frozen INR
- Post-incident evidence: full event timeline, monitoring data export, post-incident review notes
- Optional: Frozen SRP (for SLO impact calculation)

### Process

1. After the settling period, collect: frozen INR + full incident evidence + post-incident review notes + monitoring data for the incident window.
2. In a new AI session, provide all inputs + `docs/specs/pmr-spec.md` + `docs/artifacts/pmr-template.md` + `docs/principles/diagnostic-principles.md`. Use the PMR prompt (`docs/prompts/pmr-prompt.md`).
3. Review the generated PMR. Confirm that:
   - Root cause analysis reflects what was actually learned, not initial speculation
   - Corrective actions have real owners, real deadlines, and real tracking references
   - SLO impact is calculated correctly from the SRP error budget
   - Lessons are specific and traceable
   - §8 Cross-Kit Actions accurately identifies escalation needs
4. Validate in a separate session using `docs/validators/pmr-validator.md`.
5. On PASS: review owner freezes the PMR.
6. On FAIL: address blocking issues; regenerate or correct; re-validate.

### Timing

Target: PMR completed within 5 business days of incident resolution.

### Freeze Points
- PMR must be Frozen before RB generation begins (if RB is warranted)
- Frozen PMR is the cross-kit output to RRK (for next RHR) and optionally EEK and PIK

---

## Step 3 — Runbook (Optional)

**Artifact:** Runbook (RB)
**Type:** Generated, optional, versioned
**Spec:** `docs/specs/rb-spec.md` (5 hard gates)
**Template:** `docs/artifacts/rb-template.md`
**Prompt:** `docs/prompts/rb-prompt.md`
**Validator:** `docs/validators/rb-validator.md`

### When to Generate

Generate a Runbook when the PMR §8 Cross-Kit Actions recommends it — i.e., when the PMR identifies a repeatable failure class with a known resolution procedure.

Do not generate a Runbook for:
- Novel failures where the root cause is not yet fully understood
- One-time failures unlikely to recur
- Failures where the resolution was a one-time fix (e.g., "cleared the stuck migration") rather than a repeatable procedure

### Inputs
- Frozen PMR
- Optional: Frozen INR (for detailed diagnostic steps)
- Optional: Frozen SRP (for SLO thresholds in trigger conditions)

### Process

1. In a new AI session, provide: frozen PMR + frozen INR + `docs/specs/rb-spec.md` + `docs/artifacts/rb-template.md`. Use the RB prompt (`docs/prompts/rb-prompt.md`).
2. Review the generated RB. Confirm that:
   - Trigger conditions are observable from monitoring without investigation
   - Resolution steps are specific enough to execute under pressure
   - Verification steps have observable criteria
   - Escalation conditions define when to stop
   - Evidence grounding references the correct INR and PMR IDs
3. Validate in a separate session using `docs/validators/rb-validator.md`.
4. On PASS: review owner freezes the RB.
5. On FAIL: address blocking issues; regenerate or correct; re-validate.

### Timing

Target: RB completed within 10 business days of PMR freeze.

### Versioning

When the resolution procedure changes, issue a new RB version (v2, v3, etc.) rather than editing the frozen RB. Refer to `docs/specs/rb-spec.md §Versioning Rules` for the full protocol.

---

## Downstream Outputs

**Downstream:** Multiple cross-kit consumers

| Destination | Artifact | What to Provide |
|-------------|----------|----------------|
| RRK | PMR | Reference PMR ID in next RHR §Incident Summary. PMR corrective actions may trigger SRP revision — apply RRK SRP Revision Protocol. |
| EEK | PMR §Corrective Actions | When PMR §8 identifies a code defect (EEK Escalation), create an escalation record per RRK Trigger 1 pattern. Provide PMR ID and defect description to EEK team. |
| PIK | PMR recurring pattern | When PMR §8 identifies a recurring pattern (3+ occurrences), provide PMR ID and pattern description to PIK team as discovery intake context. |
| IEK | ER §8 | Update the project Engagement Record §8 with all frozen ODK artifact IDs. PMR data supplements portfolio synthesis. |

---

## Freeze Points Summary

| Artifact | When Frozen | What It Gates |
|----------|------------|---------------|
| DCR | Before INR generation | INR generation |
| INR | Within 24h of resolution | PMR generation |
| PMR | Within 5 business days of resolution | RB generation (if warranted); cross-kit outputs |
| RB | Within 10 business days of PMR freeze | Runbook is ready for operational use |

---

## Session Discipline

Follow these rules in every AI session:

| Rule | Why |
|------|-----|
| One artifact per session | Prevents context contamination across artifacts |
| Separate generation and validation sessions | Prevents self-validation bias |
| Include full frozen upstream artifacts | Do not summarize; the AI needs complete context |
| Include spec and template for generation sessions | The AI generates against the spec, not from memory |
| Include spec and validator for validation sessions | The validator judges against the spec, not against the generated content |

Do not ask the AI that generated an artifact to also validate it in the same session.

---

## Re-Entry Protocol

When a frozen artifact must be corrected:

1. Identify the artifact to be changed (DCR, INR, PMR, or RB).
2. Identify all downstream artifacts that reference it or depend on it.
3. Assess the impact: does the change affect the PMR root cause? Does it change SLO impact calculations or corrective action grounding?
4. Issue a new version of the artifact (e.g., INR-SERVICE-001v2, PMR-SERVICE-001v2).
5. Re-validate the new version.
6. Re-validate all affected downstream artifacts.
7. Document the change in the new version's document control section.

---

## Abort Protocol

If the DCR cannot be completed because the incident context is too sparse:

1. Do not proceed with INR generation.
2. Document the gaps in the Incident Intake Form and the DCR draft.
3. Gather the missing information before authoring the final DCR.

If the incident is resolved before the INR can be generated (very fast resolution):

1. Complete the DCR from available evidence, even if sparse.
2. Generate the INR from DCR + available evidence + responder recollection. Note in the INR document control that the investigation was informal due to rapid resolution.
3. The PMR is still required — fast resolution does not exempt the incident from postmortem analysis if it meets severity thresholds.

---

## Amendment Procedure

A frozen artifact may be corrected in place without re-validation when **all** of the following criteria are met:

1. The correction does not affect any field evaluated by a hard gate.
2. The correction does not change scope, decisions, owners, or technical findings.
3. The correction does not affect any field referenced by a downstream artifact.

**Procedure:** Make the correction and add an Amendment Log entry to the artifact's Document Control section: date, what changed, materiality criterion cited, and who authorized the change. No re-validation required.

**If there is any ambiguity**, treat the change as material and issue a new artifact version.

---

## Escalation Paths

ODK artifacts may trigger cross-kit escalation:

### To EEK (Code Defect)
When a PMR §4 root cause is a code defect in an EEK-governed system:
1. Create an escalation record: PMR ID, defect description, affected system.
2. Send to the EEK team. They will create a Kit Entry Record and handle the fix.
3. Reference the escalation in the ER §8 notes.

### To PIK (Recurring Pattern)
When PMR §8 identifies a recurring pattern (3+ occurrences of the same root cause class):
1. Create an escalation record: PMR ID(s), pattern description, affected service.
2. Send to PIK team as discovery intake context.
3. Reference the escalation in the ER §8 notes.

A human must authorize both escalation decisions.

---

## Principle File Revision

When `docs/principles/diagnostic-principles.md` changes, use the change categories defined in `aieos-governance-foundation/docs/principle-file-standard.md`:

| Change Category | Version Bump | Re-Entry Impact |
|----------------|-------------|-----------------|
| **Minor** (clarification only) | `v_.x → v_.x+1` | No re-entry required; already-frozen artifacts remain valid |
| **Significant** (new requirement or tightened constraint) | `v1.x → v1.x+1` | Review artifacts generated after the change against updated principles; already-frozen artifacts are grandfathered |
| **Breaking** (removal or loosening) | `vN.x → vN+1.0` | Requires service owner authorization and documented business justification; re-entry may be warranted |

Every change to a principle file must bump the version field.

---

## Deprecation and Sunset

When a service is decommissioned or a diagnostic engagement is cancelled, ODK artifacts transition to terminal lifecycle states. See `aieos-governance-foundation/docs/deprecation-protocol.md`.

| Situation | Action |
|-----------|--------|
| Service decommissioned | `Deprecated` on all frozen ODK artifacts for that service |
| Service migrated to new system | `Deprecated` on old artifacts; new service starts a new artifact series |
| Diagnostic engagement cancelled after one or more artifacts are Frozen | `Deprecated` on frozen artifacts; `Abandoned` on non-frozen artifacts |
| Engagement cancelled before any artifact is Frozen | `Abandoned` on all in-progress artifacts |

Frozen PMRs are especially important to retain — their organizational learning value persists even after the service is decommissioned.

---

## Maintaining the Engagement Record

The Engagement Record (ER) is a project-level artifact that lives in the consuming project at `docs/engagement/er-{initiative}.md`. The ER spec and format are defined in `aieos-governance-foundation/docs/engagement-record-spec.md`.

**ODK maintains the Layer 8 section of the ER.**

### What to Update During Diagnostic Operations

| Trigger | ER update |
|---------|-----------|
| DCR frozen | Add DCR ID to §8 artifact table |
| INR frozen | Add INR ID to §8 artifact table |
| PMR frozen | Add PMR ID to §8 artifact table |
| RB frozen | Add RB ID to §8 artifact table (N/A if no runbook produced) |

### On Service Decommission

When issuing a Deprecation Notice: update §1 Status and §7 Initiative Outcome as appropriate.
