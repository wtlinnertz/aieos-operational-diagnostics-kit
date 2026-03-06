# Diagnostic Context Record — Validator

This validator evaluates a completed Diagnostic Context Record (DCR) against `docs/specs/dcr-spec.md`. It is used in a separate AI session from the one that authored the DCR.

**Validator role:** Judge pass/fail. Do not suggest improvements, redesign content, or offer alternatives. Evaluate only what is explicitly present.

---

## Inputs Required

To run this validator, provide:
1. The completed DCR (full document)
2. `docs/specs/dcr-spec.md` (the spec — use this as the authoritative rules)

Do not use any other document as the source of truth for pass/fail criteria.

---

## Evaluation Procedure

Evaluate each hard gate in order. For each gate, apply the rules exactly as stated in the spec. Do not infer intent. Do not give partial credit. Ambiguity in the artifact is a failure condition — if you cannot determine whether a gate passes from what is explicitly present, the gate fails.

---

## Hard Gate Checks

### Gate 1: service_identification

Check:
- Service name is present and specific — "backend services" or "the API" is not a specific service name
- Environment is from the enumerated list: production | staging | development | multi-environment
  - "prod" or "live" without explicitly matching the enumerated values fails
- Incident owner is a named individual — a team name, role title, or "on-call" fails

**Pass:** Specific service name; environment from enumerated list; named individual as incident owner.
**Fail:** Generic service description; environment absent or not from enumerated list; team name or role as incident owner.

---

### Gate 2: symptom_description

Check:
- At least one symptom is described with a measurable or observable characteristic — "service is slow" or "users complain" without a specific metric, count, or observable state fails
- Incident start time is present with both date and time — date only fails; "around 2pm" without a date fails

**Pass:** At least one specific symptom with measurable/observable characteristic; start time with date and time.
**Fail:** Only generic symptom descriptions; start time absent or date only.

---

### Gate 3: scope_bounded

Check:
- Impact scope is declared — field must not be blank
- Scope is from the enumerated list: all users | subset of users | internal only | unknown
  - "some users" or "many users" not from the enumerated list fails
- If scope is "subset of users," the subset must be described specifically — "most users" or "affected users" fails

**Pass:** Scope from enumerated list; if subset, subset described specifically.
**Fail:** Scope field blank; scope not from enumerated list; subset declared but not described.

---

### Gate 4: evidence_documented

Check:
- At least one observable signal is documented with its source
- Signal must be specific — "monitoring alert" without a name fails; "users reported issues" without specifics fails
- Signal source must be identified — signal listed without a source (monitoring system, user report channel, or engineer name) fails

**Pass:** At least one observable signal with source; signal is specific enough to be actionable.
**Fail:** No signals documented; signal present but no source; signal too generic to be actionable.

---

### Gate 5: recent_changes_documented

Check:
- Recent changes in the last 72 hours are documented, OR
- The explicit statement "No changes in last 72h" is present
- "Unknown" without a documented resolution action (who will check and by when) fails
- A blank section or "TBD" fails

**Pass:** Changes documented (even if none — "No changes in last 72h" present); or unknown with documented resolution action.
**Fail:** Section blank; "TBD"; "check with the team" without a resolution action; "no changes" without the 72h qualifier.

---

## Output Format

Produce a JSON result in exactly this format:

```json
{
  "status": "PASS | FAIL",
  "summary": "<one sentence verdict>",
  "hard_gates": {
    "service_identification": "PASS | FAIL",
    "symptom_description": "PASS | FAIL",
    "scope_bounded": "PASS | FAIL",
    "evidence_documented": "PASS | FAIL",
    "recent_changes_documented": "PASS | FAIL"
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
- Do not redesign the symptom description or scope assessment
- Do not evaluate writing quality beyond spec requirements
- Do not infer severity from impact description if the severity field uses a non-enumerated value
- Evaluate only what is explicitly present in the document
