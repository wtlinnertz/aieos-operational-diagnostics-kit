# Diagnostic Context Record — Specification

The Diagnostic Context Record (DCR) is the entry gate for the Operational Diagnostics Kit. It must be completed before the Investigation Record can be generated. It validates that enough context exists to begin structured diagnosis: the service is identified, symptoms are described specifically, scope is bounded, at least one observable signal is documented, and recent changes are accounted for.

This is a **boundary contract** and an **entry gate**, not a generated artifact. The DCR is human-authored. It is validated against this spec before INR generation begins.

---

## What This Artifact Is Not

- **Not an incident ticket or alert.** The DCR is a structured context record that gates diagnostic investigation — not a monitoring alert or helpdesk ticket. It requires deliberate human authoring to confirm sufficient context exists.
- **Not an investigation record.** The DCR confirms context exists; the INR documents the investigation itself. Hypotheses, diagnostic steps, and conclusions belong in the INR.
- **Not a postmortem.** The DCR captures the initial state at investigation start. Organizational learning and root cause analysis belong in the PMR.

---

## Purpose

The DCR serves two roles:

1. **Entry gate** — Confirms that an incident has enough documented context to support structured diagnosis; prevents investigation from beginning in a context vacuum
2. **Baseline record** — Freezes the initial incident state so the INR has an authoritative starting point and investigation drift is traceable

---

## Upstream Dependencies

- Incident declaration or equivalent trigger (SEV1/2 mandatory; operator judgment for lower severity)
- Optional: Frozen Service Reliability Profile (SRP) from RRK — for SLO baseline and known failure modes
- Optional: Frozen Release Record §7 from REK — for recent changes and deployment state

---

## Required Sections

1. Document Control
2. Service Identification
3. Symptom Description
4. Scope and Impact
5. Observable Signals
6. Recent Changes
7. Freeze Declaration

---

## Content Rules

### Document Control
**Rules**
- DCR ID must be present (format: `DCR-{SERVICE}-{NNN}`)
- Date and time must be present
- Incident severity must be stated using the enumerated list: SEV1 | SEV2 | SEV3 | SEV4
- Incident owner must be named (individual, not team)
- Status must be present: Draft / Validated / Frozen

**Failure Examples**
- Missing DCR ID
- Severity stated as "Critical" or "P1" — non-enumerated value
- "Platform team" as incident owner — not a named individual

### Service Identification
**Rules**
- Service name must be present (specific service or component name)
- Environment must be declared (production | staging | development | multi-environment)
- SRP reference should be noted if a frozen SRP exists for the service (ID or "No SRP")

**Failure Examples**
- "Backend services" — not a specific service name
- Environment absent or "unknown"

### Symptom Description
**Rules**
- Symptoms must be described specifically — not generically ("it's slow," "users complain," "something is wrong")
- At least one specific symptom with a measurable or observable characteristic must be present
- Incident start time must be present with date and time (not date only)

**Failure Examples**
- "Service is degraded" — too generic
- "Notifications are slow" without any specificity (e.g., how slow, for which users, which operation)
- Start time present as date only (no time)

### Scope and Impact
**Rules**
- Impact scope must be declared from one of: all users | subset of users (described) | internal only | unknown
- If "subset of users," the subset must be described (not just "some users")
- If "unknown," that must be explicitly stated — not left blank

**Failure Examples**
- Scope field blank
- "Many users" without defining the subset
- Impact scope not declared

### Observable Signals
**Rules**
- At least one observable signal must be documented: alert name, metric name and value, user report with specifics, or engineer observation with specifics
- Signals must be specific enough to be actionable — "monitoring alert" alone fails
- Signal source must be identified (monitoring system, user report channel, engineer name)

**Failure Examples**
- "Got an alert" — no alert name
- "Users reported issues" — no specifics
- Observable signals section blank

### Recent Changes
**Rules**
- Changes in the last 72 hours must be documented (deployments, configuration changes, dependency changes, infrastructure changes), OR
- The statement "No changes in last 72h" must be present explicitly
- "Unknown" is not acceptable — if change history is unknown, that must be documented as a gap with a resolution action

**Failure Examples**
- Recent changes section blank
- "Check with the team" — not a documented state
- "No changes" without the explicit "in last 72h" qualifier

---

## Format Requirements

- DCR ID must follow the format `DCR-{SERVICE}-{NNN}` where SERVICE is a short identifier for the service (e.g., NOTIF, PAYMENTS, AUTH)
- Severity must be from the enumerated list: SEV1 | SEV2 | SEV3 | SEV4
- Environment must be from the enumerated list: production | staging | development | multi-environment
- Incident owner must be a person's name, not a team name or role title
- Incident start time must include date and time

---

## Completeness Rules

- All seven sections must be present and non-empty
- Service must be named specifically
- At least one observable signal documented with source
- Recent changes documented or "no changes in last 72h" explicitly stated

---

## Relationship Rules

- The DCR gates INR generation — no INR may be generated until the DCR is frozen and validated
- The DCR does not replace the INR — the INR follows after the entry gate passes
- A DCR is per-incident — a new incident requires a new DCR

---

## Hard Gates

1. **service_identification** — Service named; environment declared (from enumerated list); incident owner named (individual, not team)
2. **symptom_description** — At least one specific symptom described with measurable/observable characteristic; incident start time present with date and time
3. **scope_bounded** — Impact scope declared from enumerated list (all users / subset described / internal only / unknown); if subset, subset described
4. **evidence_documented** — At least one observable signal documented with source (alert name, metric with value, user report with specifics, or engineer observation with specifics)
5. **recent_changes_documented** — Changes in last 72h documented, OR "No changes in last 72h" stated explicitly; "unknown" without resolution action fails
