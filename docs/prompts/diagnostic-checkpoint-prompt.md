# Diagnostic Checkpoint Prompt

**Version:** 1.0

## Purpose

This utility prompt assesses the current state of an in-progress investigation and recommends whether to continue the current investigation path, pivot to a different hypothesis, or escalate. It evaluates investigation convergence and identifies investigation gaps.

This is a utility prompt — it does not produce a governed artifact. Its output is advisory to the investigation team.

---

## When to Use

Use this prompt when:
- The investigation has been running for more than 30 minutes without a confirmed probable cause
- The team is uncertain whether to continue on the current hypothesis or pivot
- A decision must be made about whether to escalate the severity level
- Investigation has stalled and the team wants a structured second opinion

---

## Inputs Required

- Current INR draft (in-progress, not frozen)
- Optional: DCR (frozen)
- Optional: Current monitoring data (real-time or recent snapshot)

---

## Instructions

### Step 1: Assess Hypothesis State

For each hypothesis in the INR §3 Hypothesis Register:
1. Is the hypothesis grounded in the DCR signals, or was it added without evidence basis?
2. Has evidence gathering for this hypothesis made progress? (partially gathered / not started / complete)
3. Is the evidence gathered so far confirming or refuting this hypothesis?

Conclusion: Classify each hypothesis as:
- **Converging** — evidence gathered is accumulating in one direction (confirm or refute)
- **Stalled** — no evidence has been gathered despite the hypothesis being present
- **Refuted** — evidence gathered is sufficient to refute; should be marked Refuted in the INR
- **Confirmed** — evidence gathered is sufficient to confirm; probable cause should be updated

### Step 2: Assess Diagnostic Trail

Review the INR §4 Diagnostic Trail:
1. Are diagnostic steps producing findings, or are steps producing "no data" results? (a pattern of "no data" may indicate the team is looking in the wrong place)
2. Is the trail converging toward the hypothesis register, or exploring in an unstructured direction?
3. Are any hypotheses in the register completely uninvestigated despite time passing?

### Step 3: Identify Investigation Gaps

What has not been investigated that should be, given the DCR signals:
1. Hypotheses not yet investigated
2. Evidence sources not yet checked (logs, metrics, dependencies)
3. Recent changes from the DCR that haven't been correlated with the timeline

### Step 4: Render Recommendation

Based on the above, recommend one of:

**Continue** — The investigation is converging. The current path is producing evidence. Continue with the next diagnostic step from the current hypothesis.
- State: what specific step to take next, and what finding would confirm or refute

**Pivot** — The current hypothesis has been sufficiently investigated with no confirmation. Refute the current hypothesis and move to the next-highest-likelihood hypothesis.
- State: which hypothesis to refute (with basis), and which hypothesis to pivot to

**Escalate** — The investigation has not converged and the severity or scope warrants escalation.
- State: the escalation trigger (time elapsed, scope increase, evidence of broader impact), and who to escalate to

**Pause for Evidence** — Investigation cannot continue without specific evidence that isn't available yet (e.g., waiting for logs to populate, waiting for a dependency team to respond).
- State: what evidence is needed, who can provide it, and what the waiting condition is

---

## Output Format

```
## Investigation Checkpoint Assessment

**Current time in investigation:** {duration since DCR creation}

**Hypothesis State:**
- H-1 {name}: {Converging / Stalled / Refuted / Confirmed} — {basis}
- H-2 {name}: {state} — {basis}

**Diagnostic Trail Assessment:** {1–3 sentences on whether the trail is converging}

**Investigation Gaps:**
1. {gap — what hasn't been checked that should be}
2. {gap}

**Recommendation:** Continue / Pivot / Escalate / Pause for Evidence

**Rationale:** {2–4 sentences explaining the recommendation}

**Next step:** {specific action if Continue or Pivot; escalation target if Escalate; pending evidence if Pause}
```

---

## Behavioral Rules

- Do not anchor on the team's current hypothesis — assess the evidence objectively
- Do not recommend escalation solely based on elapsed time — assess evidence convergence
- Do not diagnose the incident — assess the investigation state, not the incident cause
- Recommendations are advisory — the incident commander decides whether to act on them
