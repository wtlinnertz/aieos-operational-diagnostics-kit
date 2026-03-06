# Investigation Record — Validator

This validator evaluates a completed Investigation Record (INR) against `docs/specs/inr-spec.md`. It is used in a separate AI session from the one that generated the INR.

**Validator role:** Judge pass/fail. Do not suggest improvements, redesign content, or offer alternatives. Evaluate only what is explicitly present.

---

## Inputs Required

To run this validator, provide:
1. The completed INR (full document)
2. `docs/specs/inr-spec.md` (the spec — use this as the authoritative rules)

Do not use any other document as the source of truth for pass/fail criteria.

---

## Evaluation Procedure

Evaluate each hard gate in order. For each gate, apply the rules exactly as stated in the spec. Do not infer intent. Do not give partial credit. Ambiguity in the artifact is a failure condition — if you cannot determine whether a gate passes from what is explicitly present, the gate fails.

---

## Hard Gate Checks

### Gate 1: context_reference

Check:
- DCR ID is present (must follow format DCR-{SERVICE}-{NNN} or be clearly an artifact ID)
- DCR status is confirmed as Frozen — "Validated," "Approved," or any other status fails
- Service and environment are restated in this section (not merely assumed from context)

**Pass:** DCR ID present; DCR status explicitly confirmed as Frozen; service and environment restated.
**Fail:** DCR ID absent; DCR status not "Frozen"; service or environment not restated.

---

### Gate 2: hypothesis_register

Check:
- At least one hypothesis is present
- Each hypothesis has all four fields: hypothesis statement, evidence required, evidence found (or "Not yet gathered"), and status
- Status is from the enumerated list: Active | Confirmed | Refuted | Superseded
  - "Pending," "In Progress," or other non-enumerated values fail
- A row with only a hypothesis statement and no evidence fields fails

**Pass:** At least one hypothesis with all four fields; status from enumerated list.
**Fail:** No hypotheses; hypotheses without evidence fields; status not from enumerated list.

---

### Gate 3: diagnostic_trail

Check:
- Steps are listed in chronological order (earliest timestamp first)
- Each step has a timestamp with both date and time — date only fails
- Each step has both an action and a finding
- Findings must be substantive — "checked logs" without specifying what was found fails; "checked logs — no errors found matching the symptom window" passes; "nothing found" without specifying what was searched fails
- At least two steps are present

**Pass:** Chronological order; all steps have timestamps with date and time; all steps have action and substantive finding; at least two steps.
**Fail:** Steps out of chronological order; timestamps date-only or absent; steps missing action or finding; findings not substantive; fewer than two steps.

---

### Gate 4: probable_cause

Check:
- Probable cause is explicitly stated, OR "Undetermined" is stated with reasoning
- If a probable cause is stated, it must reference at least one piece of evidence from the Diagnostic Trail (by step timestamp, observation, or explicit reference)
- "Unknown" without reasoning fails
- Blank section fails

**Pass:** Probable cause stated with evidence reference; OR "Undetermined" with reasoning for why it could not be established.
**Fail:** Section blank; "Unknown" without reasoning; probable cause stated without any evidence reference.

---

### Gate 5: immediate_actions

Check:
- Actions taken to restore service are documented (what was done, not what should be done)
- Restoration is confirmed with a verification method (metric value returning to baseline, alert resolving, manual check with specific result), OR current non-restored status is explicitly stated
- If service was not yet restored, the section must say so explicitly — it must not claim restoration without confirmation

**Pass:** Actions documented; restoration confirmed with observable verification, OR non-restored status stated explicitly.
**Fail:** Section blank; actions claimed without specific actions listed; restoration claimed without confirmation method.

---

### Gate 6: open_questions

Check:
- Open questions are present as specific statements, OR "None" is stated explicitly
- Section must not be blank
- If the Diagnostic Trail shows unresolved hypotheses or investigation gaps, "None" without justification fails

**Pass:** Specific open questions present; OR "None" stated explicitly (with justification if diagnostic trail shows gaps).
**Fail:** Section blank; "TBD" or "to be determined"; "None" when diagnostic trail shows unresolved investigation gaps without explanation.

---

## Output Format

Produce a JSON result in exactly this format:

```json
{
  "status": "PASS | FAIL",
  "summary": "<one sentence verdict>",
  "hard_gates": {
    "context_reference": "PASS | FAIL",
    "hypothesis_register": "PASS | FAIL",
    "diagnostic_trail": "PASS | FAIL",
    "probable_cause": "PASS | FAIL",
    "immediate_actions": "PASS | FAIL",
    "open_questions": "PASS | FAIL"
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
- Do not redesign hypotheses, diagnostic steps, or root cause analysis
- Do not evaluate writing quality beyond spec requirements
- Evaluate only what is explicitly present in the document
- Do not infer that a hypothesis was "effectively addressed" if the evidence fields are absent
