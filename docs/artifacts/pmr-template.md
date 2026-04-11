# Postmortem Record

---

## 1. Document Control

| Field | Value |
|-------|-------|
| PMR ID | PMR-{SERVICE}-{NNN} |
| Date | {YYYY-MM-DD} |
| Status | Draft / Validated / Frozen |
| Governance Model Version | 1.0 |
| Prompt Version | {version of pmr-prompt.md used} |
| Spec Version | {spec version} |
| Principles Version | {principles file versions} |

---

## 2. Incident Reference

| Field | Value |
|-------|-------|
| INR ID | INR-{SERVICE}-{NNN} |
| INR Status | Frozen |
| Severity | SEV1 / SEV2 / SEV3 / SEV4 |
| Incident Duration | {N minutes / N hours} |
| Service | {service name} |

---

## 3. Timeline

{Chronological sequence from first observable signal to service restoration. Each event must be timestamped with date and time. The following events must be identifiable: first signal, incident declaration (if declared), mitigation action(s), service restoration.}

| Timestamp | Event | Actor |
|-----------|-------|-------|
| {YYYY-MM-DD HH:MM UTC} | First observable signal: {description} | {monitoring system or person} |
| {YYYY-MM-DD HH:MM UTC} | {event description} | {actor} |
| {YYYY-MM-DD HH:MM UTC} | Incident declared SEV{N} | {incident commander} |
| {YYYY-MM-DD HH:MM UTC} | Mitigation: {action} | {name} |
| {YYYY-MM-DD HH:MM UTC} | Service restored: {confirmation} | {name} |

---

## 4. Root Cause Analysis

### Primary Root Cause

{The underlying condition that, if corrected, would prevent recurrence of this failure. Not the immediate trigger — the systemic condition that made the failure possible.}

### Contributing Factors

{Conditions that made the primary root cause more likely or more impactful. At least one required. "Human error" must be paired with the systemic condition that enabled the error.}

1. {contributing factor}
2. {contributing factor}

### Systemic Factors

{Assessment of organizational, process, or structural factors that contributed. May be "None identified: [brief explanation]" but must not be blank.}

---

## 5. SLO Impact

{For each SLO affected by this incident: name the SLO, quantify error budget consumed in time and percentage, and state remaining budget. If no SLOs were affected, state this explicitly with reasoning.}

| SLO Name | Budget Consumed (Time) | Budget Consumed (% of Window) | Remaining Budget | Policy Implication |
|----------|----------------------|------------------------------|-----------------|-------------------|
| {SLO name from SRP} | {N minutes} | {N%} | {N% / Exhausted} | {per SRP consumption policy} |

*If no SLOs were affected:*
No SLOs affected. Reasoning: {explain why no SLOs were affected — e.g., incident resolved within measurement window tolerance, no SLI metric exceeded threshold}.

---

## 6. Corrective Actions

{At least one required. Each action must have a named owner, specific target date, and tracking reference. Actions must trace to root cause findings.}

| Action | Owner | Target Date | Tracking Reference | Root Cause Basis |
|--------|-------|-------------|-------------------|-----------------|
| {specific action} | {named individual} | {YYYY-MM-DD} | {ticket ID} | {reference to §4 finding} |
| {specific action} | {named individual} | {YYYY-MM-DD} | {ticket ID} | {reference to §4 finding} |

---

## 7. Lessons Learned

{At least one lesson required. Lessons must be specific, actionable, and traceable to investigation or timeline findings. Not restatements of corrective actions — insights.}

1. {specific lesson — what the organization now knows or what practice should change}
2. {specific lesson}

---

## 8. Cross-Kit Actions

{For each category: state whether it applies and with what basis, or explicitly state "Not applicable" with brief reasoning.}

**EEK Escalation (code defect requiring engineering work):**
{Applicable — {description of defect, basis in §4, recommended EEK entry} / Not applicable — {reasoning}}

**PIK Pattern (recurring failure class warranting discovery re-engagement):**
{Applicable — {pattern description, basis in prior PMRs, recommended PIK intake topic} / Not applicable — {reasoning}}

**RB Recommendation (repeatable failure with known resolution):**
{Applicable — {failure class name, basis in §3 and §4, resolution steps to codify} / Not applicable — {reasoning; e.g., first occurrence, root cause not yet stable}}

---

## 9. Freeze Declaration

By freezing this record, the author confirms:
- The INR referenced is in Frozen status
- The timeline is chronologically accurate and covers from first signal to restoration
- The root cause analysis reflects the investigation findings, not speculation
- SLO impact is quantified or "not affected" with reasoning
- At least one corrective action has a named owner, specific date, and tracking reference
- At least one lesson is specific, actionable, and traceable
- Cross-kit actions are addressed

**Frozen by:** {name}
**Freeze date:** {YYYY-MM-DD}
**Status:** Frozen
