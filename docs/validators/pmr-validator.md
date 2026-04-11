# Postmortem Record — Validator

This validator evaluates a completed Postmortem Record (PMR) against `docs/specs/pmr-spec.md`. It is used in a separate AI session from the one that generated the PMR.

**Validator role:** Judge pass/fail. Do not suggest improvements, redesign content, or offer alternatives. Evaluate only what is explicitly present.

---

## Inputs Required

To run this validator, provide:
1. The completed PMR (full document)
2. `docs/specs/pmr-spec.md` (the spec — use this as the authoritative rules)

Do not use any other document as the source of truth for pass/fail criteria.

---

## Evaluation Procedure

Evaluate each hard gate in order. For each gate, apply the rules exactly as stated in the spec. Do not infer intent. Do not give partial credit. Ambiguity in the artifact is a failure condition — if you cannot determine whether a gate passes from what is explicitly present, the gate fails.

---

## Hard Gate Checks

### Gate 1: incident_reference

Check:
- INR ID is present (must be an artifact ID in INR-{SERVICE}-{NNN} format or clearly identifiable as such)
- INR status is confirmed as Frozen — any other status fails
- Severity is from the enumerated list: SEV1 | SEV2 | SEV3 | SEV4
  - "Critical," "P1," "High," or other non-enumerated values fail
- Incident duration is stated in clock time with a number — "brief," "extended," "several hours" without a number fail

**Pass:** INR ID present; INR confirmed Frozen; severity from enumerated list; duration in clock time with number.
**Fail:** INR ID absent; INR not confirmed Frozen; severity non-enumerated; duration without number.

---

### Gate 2: timeline

Check:
- Events are listed in chronological order (earliest timestamp first)
- Each event has a timestamp with both date and time — date only fails
- The following events are identifiable (labeled or clearly discernible from description): first observable signal, at least one mitigation action, service restoration
- The timeline covers from the first signal to service restoration — jumping from symptom to resolution with no intermediate events fails

**Pass:** Chronological order; timestamps with date and time; first signal, mitigation, and restoration identifiable; continuous coverage.
**Fail:** Events out of order; timestamps absent or date-only; first signal, mitigation, or restoration not identifiable; timeline jumps from symptom to resolution.

---

### Gate 3: root_cause

Check:
- Primary root cause is stated — the underlying condition, not just the immediate trigger
- At least one contributing factor is present
- Contributing factors do not consist of "human error" alone — if human error is the only factor cited, it must be paired with the systemic condition that enabled the error
- Systemic factors are assessed — the assessment may be "None identified: [explanation]" but must not be blank

**Pass:** Primary root cause stated; at least one contributing factor beyond pure human error; systemic factors assessed (not blank).
**Fail:** Root cause absent; only "human error" with no systemic analysis; no contributing factors; systemic factors blank.

---

### Gate 4: slo_impact

Check:
- Affected SLOs are identified by name (not by generic description — "availability SLO" must be a named SLO from the SRP)
- For each affected SLO:
  - Error budget consumed is stated in both time (minutes or equivalent) and percentage of the measurement window budget
  - Remaining budget post-incident is stated
- If no SLOs were affected: this must be explicitly stated with reasoning — blank, "N/A," or "not applicable" without reasoning fails

**Pass:** Affected SLOs named with consumption in time and percentage and remaining budget stated; OR "no SLOs affected" with reasoning.
**Fail:** Section blank; SLOs not named; budget consumption in percentage only; remaining budget absent; "N/A" without reasoning; unaffected conclusion without reasoning.

---

### Gate 5: corrective_actions

Check:
- At least one corrective action is present
- Each action has:
  - Named owner — individual name, not a team or "TBD"
  - Target date — specific (YYYY-MM-DD or equivalent) — "next sprint," "soon," "Q2" fail
  - Tracking reference — ticket ID or task system reference — "TBD" or "to be created" fail
- Actions must be grounded in root cause findings — actions with no connection to the §4 root cause analysis fail

**Pass:** At least one action with named individual owner, specific date, and valid tracking reference; action grounded in §4.
**Fail:** No corrective actions; all actions assigned to teams; dates not specific; tracking references "TBD"; actions not traceable to root cause.

---

### Gate 6: lessons_learned

Check:
- At least one lesson is stated
- Lessons must be specific — generic lessons ("improve monitoring," "communicate better," "be more careful") fail
- Lessons must trace to findings in the investigation or timeline — lessons that cannot be connected to this specific incident's findings fail
- Lessons must not be restatements of corrective actions

**Pass:** At least one specific lesson traceable to investigation findings; not a restatement of a corrective action.
**Fail:** Section blank; only generic lessons; lessons not traceable to this incident; lessons are restatements of corrective actions.

---

## Output Format

Produce a JSON result in exactly this format:

```json
{
  "status": "PASS | FAIL",
  "summary": "<one sentence verdict>",
  "hard_gates": {
    "incident_reference": "PASS | FAIL",
    "timeline": "PASS | FAIL",
    "root_cause": "PASS | FAIL",
    "slo_impact": "PASS | FAIL",
    "corrective_actions": "PASS | FAIL",
    "lessons_learned": "PASS | FAIL"
  },
  "blocking_issues": [
    {
      "gate": "<gate_name>",
      "description": "<what specifically failed>",
      "location": "<section or field where the failure is>"
    }
  ],
  "warnings": [
    {
      "description": "<non-blocking observation>",
      "location": "<section>"
    }
  ],
  "completeness_score": "<0-100>"
}
```

**Interpretation rules:**
- Any gate failure → `"status": "FAIL"`
- `blocking_issues` lists exactly the failures — no additional content
- `warnings` are non-blocking; they do not affect status
- `completeness_score` is advisory; it does not override gate results
- If all gates pass, `blocking_issues` is an empty array

---

## Validator Constraints

- Do not suggest how to fix failures
- Do not redesign the root cause analysis, timeline, or lessons
- Do not evaluate writing quality beyond spec requirements
- Do not infer severity from impact description if the severity field uses a non-enumerated value
- Evaluate only what is explicitly present in the document
