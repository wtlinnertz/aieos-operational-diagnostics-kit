# Postmortem Record — Specification

The Postmortem Record (PMR) is the post-resolution root cause analysis artifact. It is authored after the incident is resolved and the investigation is complete. It synthesizes the investigation findings into an authoritative organizational learning record: timeline, root cause, SLO impact, corrective actions, and lessons learned.

---

## Purpose

The PMR serves three roles:

1. **Organizational learning** — Documents what was learned from the incident and what the organization commits to do differently
2. **Cross-kit output** — PMR corrective actions may become EEK work items; PMR recurring patterns may inform PIK discovery; PMR is referenced in the next RRK RHR
3. **Runbook trigger** — If the PMR identifies a repeatable failure class with a known resolution, it authorizes RB generation

---

## Upstream Dependencies

- Frozen Investigation Record (INR) — must be in Frozen status before PMR generation
- Incident evidence: full timeline of events, monitoring data, responder notes
- Optional: Frozen Service Reliability Profile (SRP) — for SLO impact calculation

---

## Required Sections

1. Document Control
2. Incident Reference
3. Timeline
4. Root Cause Analysis
5. SLO Impact
6. Corrective Actions
7. Lessons Learned
8. Cross-Kit Actions
9. Freeze Declaration

---

## Content Rules

### Document Control
**Rules**
- PMR ID must be present (format: `PMR-{SERVICE}-{NNN}`)
- Date of authoring must be present
- Status must be present: Draft / Validated / Frozen
- Governance Model Version must be present
- Prompt Version must be present

**Failure Examples**
- Missing PMR ID
- Status absent or "TBD"

### Incident Reference
**Rules**
- INR ID must be present and confirmed as Frozen
- Severity must be stated using the enumerated list: SEV1 | SEV2 | SEV3 | SEV4
- Incident duration must be stated in clock time (minutes or hours with a number); "brief," "extended," or "several hours" without a number fail

**Failure Examples**
- INR ID absent
- INR status listed as "Validated" rather than "Frozen"
- Duration stated as "several hours" — no number
- Severity stated as "Critical" — non-enumerated value

### Timeline
**Rules**
- Events must be listed in chronological order
- Each event must have a timestamp with date and time
- The timeline must cover from the first observable signal to service restoration
- The following event types must be identifiable in the timeline: first signal, incident declaration (if declared), at least one mitigation action, service restoration
- Events may be labeled by type or the label may be discernible from the description — but they must be identifiable

**Failure Examples**
- Events not in chronological order
- Timestamps absent or date-only
- Timeline jumps from symptom to resolution with no intermediate events
- First signal and service restoration not identifiable

### Root Cause Analysis
**Rules**
- Primary root cause must be stated — the underlying condition that, if corrected, would prevent recurrence
- At least one contributing factor must be present — a condition that made the primary root cause more likely or more impactful
- Contributing factors must not be "human error" alone — a contributing factor that explains only human error must also identify what systemic condition enabled the error
- Systemic factors must be assessed — the assessment may conclude "none identified" but must not be blank

**Failure Examples**
- Root cause section blank
- "Human error" as the only finding with no systemic analysis
- Systemic factors section blank
- Contributing factors absent

### SLO Impact
**Rules**
- Affected SLOs must be identified with their SLO names
- For each affected SLO: error budget consumed must be quantified in both minutes (or equivalent time unit) and percentage of the measurement window budget; remaining budget post-incident must be stated
- If no SLOs were affected: this must be explicitly stated with reasoning — it is not sufficient to leave the section blank or write "N/A"

**Failure Examples**
- Section blank
- Affected SLOs listed without consumption amounts
- Budget consumption in percentage only, without time equivalent
- "N/A" without explanation of why no SLOs were affected
- SLO impact stated without identifying specific SLO names

### Corrective Actions
**Rules**
- At least one corrective action must be present
- Each corrective action must have:
  - Named owner (individual, not team)
  - Target date (YYYY-MM-DD or equivalent — "next sprint" fails)
  - Tracking reference (ticket ID or task system reference — "TBD" fails)
- The corrective action must address a specific finding from the root cause analysis — general improvement actions not grounded in this incident's findings fail

**Failure Examples**
- No corrective actions
- All actions assigned to "the team" — no named owner
- Target dates stated as "next sprint" or "soon"
- Tracking references listed as "TBD"
- Actions not grounded in root cause findings

### Lessons Learned
**Rules**
- At least one lesson must be stated
- Lessons must be specific and actionable — they describe what the organization now knows that it didn't know before, or what practice should change
- "Improve monitoring" is not a lesson; "add queue depth alert at 80% capacity to provide 20-minute lead time before exhaustion" is a lesson
- Lessons must trace to findings in the investigation or timeline

**Failure Examples**
- Section blank
- Generic lessons ("communicate better," "improve monitoring") without specifics
- Lessons that are restatements of corrective actions, not insights

### Cross-Kit Actions
**Rules**
- This section must be present
- For each cross-kit action type (EEK escalation, PIK pattern, RB recommendation): state whether it applies and with what basis, or explicitly state "Not applicable"
- "Not applicable" must include brief reasoning; blank fails

**Failure Examples**
- Section blank
- Cross-kit action types not addressed
- "Not applicable" without any reasoning

---

## Format Requirements

- PMR ID must follow the format `PMR-{SERVICE}-{NNN}`
- Severity must be from the enumerated list: SEV1 | SEV2 | SEV3 | SEV4
- INR status must be explicitly confirmed as "Frozen"
- Timestamps in the timeline must include date and time
- Corrective action target dates must be specific (YYYY-MM-DD or equivalent)

---

## Completeness Rules

- All nine sections must be present and non-empty
- At least one corrective action with owner, date, and tracking reference
- At least one lesson learned with specific, traceable content
- SLO impact quantified or "not affected" with reasoning

---

## Relationship Rules

- The PMR gates RB generation — if a RB is warranted, the PMR must be frozen first
- The PMR is the organizational learning record — it is written after resolution, not during investigation
- The PMR references the INR as its upstream dependency but is not a restatement of the INR

---

## Hard Gates

1. **incident_reference** — INR ID present and confirmed Frozen; severity from enumerated list; duration in clock time with number
2. **timeline** — Chronological sequence from first signal to resolution; events timestamped with date and time; first signal, mitigation, and restoration identifiable
3. **root_cause** — Primary root cause stated; at least one contributing factor (not "human error" alone); systemic factors assessed (may be "none identified" with explanation, not blank)
4. **slo_impact** — SLO impact quantified per affected SLO (error budget consumed in time and percentage, remaining budget stated), OR "no SLOs affected" stated explicitly with reasoning; section not blank
5. **corrective_actions** — At least one corrective action with named owner (individual), specific target date, and valid tracking reference; action grounded in root cause findings
6. **lessons_learned** — At least one specific, traceable lesson stated; not blank; not a restatement of corrective actions; not generic
