# Hypothesis Generation Prompt

**Version:** 1.0

## Purpose

This utility prompt generates candidate failure hypotheses from a completed (or partially completed) Diagnostic Context Record. It produces structured hypothesis entries suitable for use in the INR §3 Hypothesis Register.

This is a utility prompt — it does not produce a governed artifact. Its output is input to the INR generation session or to the investigation team directly.

---

## When to Use

Use this prompt at the start of a structured investigation, after the DCR has been completed but before INR generation. It helps investigation teams avoid anchoring on a single hypothesis and ensures the hypothesis register begins with a structured set of candidate causes.

May also be used mid-investigation when the investigation has stalled on a single hypothesis and the team wants to consider alternatives.

---

## Inputs Required

- Completed DCR (frozen or in-progress — hypothesis generation does not require a frozen DCR)
- Optional: Frozen SRP — for known failure modes and SLO context
- Optional: Past PMR IDs for this service (for pattern context)

---

## Instructions

### Step 1: Analyze DCR Content

From the DCR, extract:
1. Service and environment
2. Symptom description — specifically, what observable behavior is unexpected
3. Observable signals — alert names, metric values, user reports
4. Recent changes — deployments, configuration, infrastructure, dependencies

### Step 2: Generate Candidate Hypotheses

For each candidate hypothesis:
1. State the proposed cause specifically — not "the database is slow" but "database query contention on the notifications table due to missing index"
2. State the evidence required to confirm or refute — what metric, log entry, or observable state would prove or disprove this hypothesis?
3. State the evidence already available from the DCR that supports this hypothesis (or "No direct evidence yet")
4. Suggest the diagnostic step that would gather the required evidence

Apply the following reasoning patterns when generating hypotheses:

**Change-correlated**: For each recent change in the DCR, generate a hypothesis connecting that change to the observed symptoms. Deployment → check deployment timing vs. symptom onset. Config change → check if the changed value affects the symptom behavior.

**Signal-correlated**: For each observable signal, generate a hypothesis that explains why that signal would fire. Alert name → what condition triggers that alert? → what would cause that condition?

**Dependency-correlated**: If service architecture is known, generate hypotheses for each upstream and downstream dependency failure that could produce the observed symptoms.

**Resource-exhaustion**: Generate hypotheses for common resource exhaustion scenarios relevant to the service type: queue depth, connection pool, memory, CPU, disk.

### Step 3: Prioritize by Likelihood

Rank hypotheses from most likely to least likely based on:
1. Direct correlation with recent changes (change-correlated hypotheses rank higher)
2. Alignment with observable signals (hypotheses that explain all signals rank higher than those that explain only one)
3. Prior incident patterns (if past PMR data is available)

### Step 4: Format Output

For each hypothesis, output:

```
## H-{N}: {hypothesis name}

**Proposed Cause:** {specific causal statement}

**Supporting Evidence (from DCR):** {what from the DCR points to this hypothesis, or "No direct evidence yet"}

**Evidence Required to Confirm:** {what metric, log, or observable state would confirm this}

**Evidence Required to Refute:** {what would definitively rule this out}

**Suggested Diagnostic Step:** {the specific action to gather the required evidence}

**Likelihood Basis:** {why this was ranked at this position}
```

---

## Behavioral Rules

- Do not generate hypotheses without basis in the DCR evidence — if there is no signal pointing to a cause, note the reasoning for including it
- Prioritize hypotheses connected to recent changes — change-correlated failures are statistically more common
- Do not anchor on a single hypothesis — generate a minimum of 3 candidates, maximum of 8
- Do not claim certainty — all outputs are candidates pending investigation
- If the DCR is sparse (insufficient observable signals), state what additional information would improve hypothesis quality
