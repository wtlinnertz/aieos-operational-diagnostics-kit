# Runbook — Specification

The Runbook (RB) is a codified resolution procedure for a known failure class. It is generated when a Postmortem Record identifies a repeatable failure pattern with a documented resolution. The RB is versioned and reused in future incidents of the same class to reduce mean time to resolution.

---

## Purpose

The RB serves three roles:

1. **Operational efficiency** — Converts hard-won diagnostic knowledge into a reusable procedure that reduces investigation time for known failure classes
2. **Knowledge preservation** — Persists the resolution procedure even when the engineers who handled the original incident are unavailable
3. **Training artifact** — Onboards new responders to known failure patterns without requiring a live incident

---

## Upstream Dependencies

- Frozen Postmortem Record (PMR) — must be in Frozen status before RB generation; the PMR is the evidence base for the runbook
- Optional: Frozen Investigation Record (INR) — for detailed diagnostic steps that informed the runbook
- Optional: Frozen Service Reliability Profile (SRP) — for SLO thresholds referenced in trigger conditions

---

## Required Sections

1. Document Control
2. Failure Class
3. Trigger Conditions
4. Pre-Conditions
5. Resolution Steps
6. Verification
7. Escalation Conditions
8. Evidence Grounding
9. Version History

---

## Content Rules

### Document Control
**Rules**
- RB ID must be present (format: `RB-{SERVICE}-{NNN}`)
- Version must be present (format: `v{N}` — e.g., v1, v2)
- Date must be present
- Status must be present: Draft / Validated / Frozen
- Governance Model Version must be present
- Prompt Version must be present

**Failure Examples**
- Missing RB ID
- No version field
- Status absent

### Failure Class
**Rules**
- Failure class name must be present and specific (e.g., "Notification queue exhaustion due to downstream API timeout" — not "queue problem")
- Affected service must be named
- The failure class must be grounded in a specific incident — it is not a hypothetical failure mode

**Failure Examples**
- Failure class named generically ("service degradation," "performance issue")
- Affected service absent
- Failure class described as hypothetical rather than grounded in an actual incident

### Trigger Conditions
**Rules**
- Trigger conditions must be specific and observable — an on-call engineer must be able to confirm or deny each condition from monitoring or logs without investigation
- At least two trigger conditions must be present
- Each condition must reference a specific metric name, alert name, or observable state (not vague descriptions)
- Vague triggers ("service is slow," "users complain," "something seems wrong") fail

**Failure Examples**
- Fewer than two trigger conditions
- Conditions stated vaguely ("high error rate" without threshold)
- Conditions reference monitoring concepts not observable in the named service

### Pre-Conditions
**Rules**
- Pre-conditions that must be true before executing the runbook must be stated, OR "None — runbook applies immediately when trigger conditions are met" must be stated explicitly
- Examples of pre-conditions: "Confirm no active deployment in last 30 minutes," "Verify downstream API has not changed"

**Failure Examples**
- Section blank

### Resolution Steps
**Rules**
- Steps must be numbered
- Each step must be specific and actionable — an engineer unfamiliar with the service must be able to execute it
- Commands must be shown where applicable (not "run the drain command" — show the actual command or command pattern)
- Each step must describe what to do and what to observe to confirm the step succeeded
- At least three steps must be present

**Failure Examples**
- Steps not numbered
- Fewer than three steps
- Steps stated vaguely ("restart the service," "clear the queue") without specifics
- Steps with no expected outcome

### Verification
**Rules**
- At least one verification step must be present confirming that the failure condition has been resolved
- The verification must have an observable criterion — a specific metric returning to a specific value, an alert resolving, or a manual check with an expected result
- "Monitor for a while" without a specific criterion fails

**Failure Examples**
- Verification section blank
- Verification stated as "monitor" without criterion
- No observable criterion for resolution confirmation

### Escalation Conditions
**Rules**
- Specific criteria must be defined for when to stop executing the runbook and escalate
- At least one escalation condition must be present
- Conditions must be observable — not "if it doesn't work" but "if queue depth has not decreased after 10 minutes of executing Step 3"
- Escalation target must be identified (who to escalate to, or which channel to use)

**Failure Examples**
- Section blank
- "If it doesn't work, escalate" — no observable criterion
- Escalation target not identified

### Evidence Grounding
**Rules**
- The runbook must reference at least one INR or PMR that informed it
- The reference must include the artifact ID and a brief note on what the runbook draws from that artifact
- "Based on past incidents" without an artifact ID fails

**Failure Examples**
- Section blank
- Reference to past incidents without artifact IDs
- No connection between runbook content and specific artifacts

### Version History
**Rules**
- A version history table must be present
- Each entry must include: version, date, what changed, and who authorized
- For v1 (initial version), the "what changed" entry is "Initial version"
- If no changes have been made since initial release, only the v1 row is required

**Failure Examples**
- Version history absent
- Entries missing date or authorization

---

## Format Requirements

- RB ID must follow the format `RB-{SERVICE}-{NNN}`
- Version must be `v{N}` (e.g., v1, v2, v3)
- Resolution steps must be numbered (1., 2., 3., ...)
- Evidence grounding must include artifact IDs (INR-XXX or PMR-XXX format)

---

## Completeness Rules

- All nine sections must be present and non-empty
- At least two trigger conditions, specific and observable
- At least three numbered resolution steps with actions and expected outcomes
- At least one verification step with observable criterion
- At least one escalation condition with observable criterion and escalation target
- At least one INR or PMR referenced by ID in Evidence Grounding

---

## Versioning Rules

When a runbook's resolution procedure changes (e.g., a step no longer applies, a new step is added, a command changes):
1. Do not edit a frozen RB — the frozen version remains immutable
2. Generate a new RB version (v2, v3, etc.) using the RB prompt with the updated evidence
3. Validate and freeze the new version
4. Note the trigger for the revision in Version History (e.g., "Step 3 command updated after infrastructure migration — referenced PMR-AUTH-002")

---

## Relationship Rules

- The RB is optional — only generate a RB when the PMR identifies a repeatable failure class
- A frozen RB is not amended; corrections require a new version
- The RB is consumed during active incidents, not generated during them

---

## Hard Gates

1. **trigger_conditions** — At least two specific, observable trigger conditions; each references a specific metric, alert, or observable state; no vague triggers
2. **resolution_steps** — At least three numbered steps; each step is specific and actionable with expected outcome; commands shown where applicable
3. **verification** — At least one step confirming resolution with an observable criterion (specific metric value, alert resolution, or manual check result)
4. **escalation_conditions** — At least one specific, observable escalation condition with escalation target identified
5. **evidence_grounded** — At least one INR or PMR referenced by artifact ID in Evidence Grounding with a note on what the runbook draws from it
