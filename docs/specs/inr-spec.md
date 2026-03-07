# Investigation Record — Specification

The Investigation Record (INR) is the active diagnostic trail artifact produced during or immediately after a structured investigation. It documents the hypotheses considered, evidence gathered, diagnostic steps taken, and the probable cause conclusion. It is the organizational record of how the team figured out what was wrong.

---

## What This Artifact Is Not

- **Not a postmortem.** The INR documents how the team investigated — the hypotheses, diagnostic steps, and evidence gathered. Analysis of systemic causes, organizational learning, and corrective action plans belong in the PMR.
- **Not a resolution record.** The INR documents the investigation trail through probable cause; resolution execution and post-resolution verification belong in the PMR.
- **Not a substitute for evidence.** The INR must reference concrete evidence (log excerpts, metric values, system outputs) for each conclusion. Assertions without evidence references are not acceptable.

---

## Purpose

The INR serves three roles:

1. **Diagnostic trail** — Preserves the full investigation for postmortem analysis; prevents loss of context between investigation and RCA
2. **PMR input** — Provides the PMR with the authoritative probable cause and evidence base for root cause analysis
3. **Runbook seed** — Documents the diagnostic steps that, if repeatable, become a runbook

---

## Upstream Dependencies

- Frozen Diagnostic Context Record (DCR) — must be in Frozen status before INR generation
- Incident evidence gathered during investigation: alert records, metric data, log excerpts, responder notes
- Optional: Frozen Service Reliability Profile (SRP) — for SLO context and known failure modes
- Optional: Frozen System Architecture Document (SAD) from EEK — for service dependency context

---

## Required Sections

1. Document Control
2. Context Reference
3. Hypothesis Register
4. Diagnostic Trail
5. Probable Cause
6. Immediate Actions
7. Open Questions
8. Freeze Declaration

---

## Content Rules

### Document Control
**Rules**
- INR ID must be present (format: `INR-{SERVICE}-{NNN}`)
- Date and time of authoring must be present
- Status must be present: Draft / Validated / Frozen
- Governance Model Version must be present
- Prompt Version must be present

**Failure Examples**
- Missing INR ID
- Date only (no time) for authoring timestamp

### Context Reference
**Rules**
- DCR ID must be present
- DCR status must be confirmed as Frozen (not Draft or Validated)
- Service and environment from the DCR must be restated

**Failure Examples**
- DCR ID missing or "see incident ticket"
- DCR status listed as "Validated" rather than "Frozen"

### Hypothesis Register
**Rules**
- At least one hypothesis must be present
- Each hypothesis must include:
  - Hypothesis statement (the proposed cause)
  - Evidence required to confirm or refute
  - Evidence found (what was actually observed), or "Not yet gathered"
  - Status: Active | Confirmed | Refuted | Superseded
- A hypothesis row that has only a statement and no evidence fields fails this gate

**Failure Examples**
- No hypotheses present
- Hypotheses listed without evidence fields
- All hypotheses in "Active" status with no investigation against them

### Diagnostic Trail
**Rules**
- Diagnostic steps must be listed in chronological order
- Each step must have a timestamp with date and time
- Each step must have both an action taken and a finding from that action
- "Checked logs" without specifying what was found is not a valid finding — the finding must be substantive ("Checked logs — found no errors matching the symptom window" or "Checked logs — found 847 delivery timeout errors between 14:22 and 14:45 UTC")
- At least two diagnostic steps must be present

**Failure Examples**
- Steps not in chronological order
- Steps missing timestamps
- Steps with action but no finding
- Fewer than two steps
- Findings stated as "nothing found" without specifying what was checked

### Probable Cause
**Rules**
- Probable cause must be stated explicitly, OR "Undetermined" must be stated with reasoning for why it could not be determined
- If a probable cause is stated, it must trace to at least one piece of evidence in the Diagnostic Trail
- "Unknown" without reasoning fails; "Undetermined — queue exhaustion confirmed but root cause of queue growth not identified due to log retention gap" passes

**Failure Examples**
- Probable cause section blank
- "Unknown" with no explanation
- Probable cause stated without any evidence reference

### Immediate Actions
**Rules**
- Actions taken to restore service must be documented (not what should have been done — what was done)
- Restoration must be confirmed (how it was verified — metric returning to baseline, alert resolved, manual verification with result), OR current status stated explicitly if service is not yet restored
- If service was not yet restored at time of INR authoring, that must be stated explicitly

**Failure Examples**
- Immediate actions section blank
- "Restarted the service" without confirmation of restoration
- Restoration claimed without confirmation method

### Open Questions
**Rules**
- Open questions remaining after the investigation must be captured, OR "None" must be stated explicitly
- Open questions are questions that remain unanswered and relevant to understanding the full cause — not general improvement suggestions
- If open questions exist, each must be stated specifically (not "understand the system better")

**Failure Examples**
- Section blank
- "None" when the Diagnostic Trail shows unresolved hypotheses
- Open questions stated as improvement suggestions rather than investigation gaps

---

## Format Requirements

- INR ID must follow the format `INR-{SERVICE}-{NNN}`
- DCR status must be confirmed explicitly as "Frozen"
- Timestamps for diagnostic steps must include date and time
- Hypothesis status must be from the enumerated list: Active | Confirmed | Refuted | Superseded

---

## Completeness Rules

- All eight sections must be present and non-empty
- At least one hypothesis with evidence fields
- At least two diagnostic steps with timestamps, actions, and findings
- Probable cause stated or "Undetermined" with reasoning

---

## Relationship Rules

- The INR gates PMR generation — no PMR may be generated until the INR is frozen
- The INR is an investigation artifact — it reflects what was known and done during investigation, not retrospective analysis
- The INR informs the PMR's root cause analysis; it is not the RCA itself

---

## Hard Gates

1. **context_reference** — DCR ID present and confirmed Frozen; service and environment restated
2. **hypothesis_register** — At least one hypothesis with hypothesis statement, evidence required, evidence found (or "not yet gathered"), and status from enumerated list
3. **diagnostic_trail** — At least two diagnostic steps in chronological order; each step has timestamp with date and time, action taken, and substantive finding
4. **probable_cause** — Probable cause stated with evidence reference, OR "Undetermined" stated with reasoning; section not blank
5. **immediate_actions** — Actions taken to restore service documented; restoration confirmed with verification method, OR current non-restored status explicitly stated
6. **open_questions** — Open questions captured with specific statements, OR "None" stated explicitly; section not blank
