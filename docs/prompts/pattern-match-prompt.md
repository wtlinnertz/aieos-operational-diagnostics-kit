# Pattern Match Prompt

**Version:** 1.0

## Purpose

This utility prompt compares a new DCR's context against past PMRs and RBs for the same service to identify whether the current incident matches a known failure signature. It helps investigation teams recognize recurrences quickly and apply existing runbooks when applicable.

This is a utility prompt — it does not produce a governed artifact. Its output is advisory to the investigation team.

---

## When to Use

Use this prompt at the start of a new investigation, after completing the DCR, to check whether this incident matches a prior incident pattern before beginning hypothesis generation. Can also be used mid-investigation to confirm whether a suspected cause has been seen before.

---

## Inputs Required

- Current DCR (frozen or in-progress)
- Past frozen PMRs for this service (provide as many as available — up to 5 most recent)
- Optional: Frozen RBs for this service

---

## Instructions

### Step 1: Extract DCR Signature

From the current DCR, extract the pattern signature:
1. Observable signals: alert names, metric names with approximate values, user-visible symptoms
2. Impact scope: all users / subset / internal
3. Recent changes: any changes in last 72h
4. Time characteristics: time of day, day of week (if relevant)

### Step 2: Compare Against Past PMRs

For each past PMR provided:
1. Extract the PMR's failure signature: what signals were present, what was the root cause
2. Score similarity to the current DCR signature on:
   - Signal match: same alerts? same metrics? similar values?
   - Scope match: same impact pattern?
   - Context match: similar recent change type (or no recent changes in both)?
3. Classify the match:
   - **Strong match** — 3 or more signal matches; likely the same failure class
   - **Partial match** — 1–2 signal matches; possible overlap but different root cause is plausible
   - **No match** — signals are substantively different

### Step 3: Check RBs

For each frozen RB provided:
1. Compare the RB's trigger conditions against the current DCR's observable signals
2. If all trigger conditions are met: flag as "Applicable RB — review before investigating"
3. If some but not all trigger conditions are met: flag as "Partial RB match — trigger conditions {X} met, {Y} not confirmed"
4. If no trigger conditions are met: note as "Not applicable"

### Step 4: Render Pattern Assessment

---

## Output Format

```
## Pattern Match Assessment

**Service:** {service name from DCR}

**Current DCR Signature:**
- Signals: {alert names, metrics}
- Scope: {impact scope}
- Recent changes: {yes/no + type, or none}

### PMR Comparisons

**PMR-{ID} — {root cause summary}**
- Signal match: {matched signals / not matched}
- Scope match: {yes / partial / no}
- Overall: Strong match / Partial match / No match
- If Strong match: {recommend checking PMR §4 root cause and §6 corrective actions for context}

**PMR-{ID} — {root cause summary}**
- {same structure}

### RB Applicability

**RB-{ID} — {failure class name}**
- Trigger condition 1: {Met / Not confirmed}
- Trigger condition 2: {Met / Not confirmed}
- Verdict: Applicable / Partial match / Not applicable

### Summary

**Known failure recurrence:** {Yes — strong match with PMR-XXX / Possible — partial match / No — novel failure}

**RB available:** {Yes — RB-XXX applies; review trigger conditions before executing / No applicable RB}

**Recommended next step:** {execute RB-XXX / investigate with awareness of prior PMR-XXX pattern / no prior pattern — begin fresh hypothesis generation}
```

---

## Behavioral Rules

- Do not diagnose the current incident — match patterns only
- Do not claim certainty — pattern matching informs investigation, it does not confirm cause
- If a strong match exists, recommend reviewing the prior PMR's investigation trail, not applying its root cause automatically — the current incident may share a failure class but have a different mechanism
- Do not compare against PMRs from different services — failure patterns are service-specific
