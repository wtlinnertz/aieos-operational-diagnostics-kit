# Runbook — Validator

This validator evaluates a completed Runbook (RB) against `docs/specs/rb-spec.md`. It is used in a separate AI session from the one that generated the RB.

**Validator role:** Judge pass/fail. Do not suggest improvements, redesign content, or offer alternatives. Evaluate only what is explicitly present.

---

## Inputs Required

To run this validator, provide:
1. The completed RB (full document)
2. `docs/specs/rb-spec.md` (the spec — use this as the authoritative rules)

Do not use any other document as the source of truth for pass/fail criteria.

---

## Evaluation Procedure

Evaluate each hard gate in order. For each gate, apply the rules exactly as stated in the spec. Do not infer intent. Do not give partial credit. Ambiguity in the artifact is a failure condition — if you cannot determine whether a gate passes from what is explicitly present, the gate fails.

---

## Hard Gate Checks

### Gate 1: trigger_conditions

Check:
- At least two trigger conditions are present
- Each condition is specific and observable — an on-call engineer can confirm or deny from monitoring without investigation
- Each condition references a specific metric name, alert name, or observable state — vague triggers ("service is slow," "high error rate" without threshold, "users are complaining") fail

**Pass:** At least two conditions; each references a specific metric, alert, or observable state; each is confirmable from monitoring.
**Fail:** Fewer than two conditions; any condition stated vaguely without specific observable reference.

---

### Gate 2: resolution_steps

Check:
- Steps are numbered
- Each step specifies what to do (action) and what to observe to confirm the step succeeded (expected outcome)
- Steps are specific enough for an engineer unfamiliar with the service to execute — "restart the service" without specifying which service or command fails; "restart notification-worker using `systemctl restart notification-worker`" passes
- Commands are shown where applicable — "run the drain command" without showing the command fails
- At least three steps are present

**Pass:** Numbered steps; each step has action and expected outcome; steps are specific; commands shown where applicable; at least three steps.
**Fail:** Steps not numbered; fewer than three steps; steps missing expected outcomes; steps too vague to execute; commands referenced but not shown.

---

### Gate 3: verification

Check:
- At least one verification step is present confirming the failure condition is resolved
- The verification has an observable criterion — a specific metric returning to a specific value, a specific alert resolving, or a manual check with a specific expected result
- "Monitor for a while," "observe for stability," or "wait and see" without a specific criterion fail

**Pass:** At least one verification step with a specific observable criterion for resolution.
**Fail:** Verification section blank; verification stated without observable criterion; "monitor" without specific metric and threshold.

---

### Gate 4: escalation_conditions

Check:
- At least one escalation condition is present
- Conditions are observable — "if it doesn't work" fails; "if queue depth has not decreased within 10 minutes of executing Step 3" passes
- An escalation target is identified — who to escalate to, which channel, or which pager

**Pass:** At least one observable escalation condition; escalation target identified.
**Fail:** Section blank; "if it doesn't work" or other non-observable conditions; escalation target not identified.

---

### Gate 5: evidence_grounded

Check:
- At least one INR or PMR is referenced by artifact ID (must be in INR-{SERVICE}-{NNN} or PMR-{SERVICE}-{NNN} format, or clearly an artifact ID)
- The reference includes a note on what the runbook draws from that artifact
- "Based on past incidents" without an artifact ID fails
- A reference with an artifact ID but no note on what was drawn from it fails

**Pass:** At least one INR or PMR referenced by ID with a note on what the runbook draws from it.
**Fail:** No artifact IDs referenced; "based on past incidents" without ID; artifact ID present but no note on what was drawn.

---

## Output Format

Produce a JSON result in exactly this format:

```json
{
  "status": "PASS | FAIL",
  "summary": "<one sentence verdict>",
  "hard_gates": {
    "trigger_conditions": "PASS | FAIL",
    "resolution_steps": "PASS | FAIL",
    "verification": "PASS | FAIL",
    "escalation_conditions": "PASS | FAIL",
    "evidence_grounded": "PASS | FAIL"
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
- Do not redesign resolution steps or trigger conditions
- Do not evaluate whether the runbook would actually work — only evaluate what is present against spec requirements
- Do not evaluate writing quality beyond spec requirements
- Evaluate only what is explicitly present in the document
