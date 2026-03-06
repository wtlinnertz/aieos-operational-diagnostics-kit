# DCR → SRP Consistency Check Prompt

**Version:** 1.0

## Purpose

This utility prompt validates that the Diagnostic Context Record's service identification and symptoms are consistent with the frozen Service Reliability Profile for that service. It detects mismatches between the incident context and the known SRP baseline — mismatches that may indicate the wrong SRP is being used, or that the service has changed since the SRP was frozen.

This is a cross-boundary consistency check — it validates an ODK artifact against an RRK artifact. It is a utility prompt; it does not produce a governed artifact.

---

## When to Use

Use this prompt after completing the DCR and before beginning INR generation, when a frozen SRP exists for the service. Run it in a separate session from DCR authoring.

---

## Inputs Required

- Completed DCR (frozen or in-progress)
- Frozen SRP for the service named in the DCR

---

## Instructions

### Step 1: Service Identity Match

Compare:
- DCR §2 Service Name vs. SRP §1 Service Name (or equivalent field)
- DCR §2 Environment vs. SRP §1 Environment

**Pass:** Service names match (accounting for minor naming variations that clearly refer to the same service); environments match.
**Flag:** Names differ in a way that is not clearly the same service; environments differ.

### Step 2: SLO Baseline Awareness

From the DCR §3 Symptom Description and §5 Observable Signals, identify which SLO dimensions are affected (availability, latency, throughput, etc.) and compare to the SRP §2 SLO definitions:

1. Does the DCR reference an SLI metric that is defined in the SRP? Or does it reference a metric not in the SRP?
2. Are the DCR's symptom values (error rates, latency values) in the range that would trigger SLO breach per the SRP §3 thresholds?
3. Does the SRP define an alert that corresponds to the alerts named in the DCR §5?

### Step 3: Known Failure Mode Context

From the SRP (if it contains known failure modes or watch items from the SRER):
1. Is the current incident signature consistent with any documented watch items or known failure modes in the SRP?
2. Are there SRP §3 burn rate alert policies that would have fired before this incident became visible?

### Step 4: Staleness Check

Review the SRP version and date:
1. Is the SRP version more than 6 months old?
2. Does the DCR §6 Recent Changes list infrastructure or architecture changes that may have invalidated the SRP baseline?

---

## Output Format

```
## DCR → SRP Consistency Check

**DCR:** {DCR ID}
**SRP:** {SRP ID and version}

### Service Identity
- Service match: Pass / Flag — {basis}
- Environment match: Pass / Flag — {basis}

### SLO Dimension Alignment
- Affected SLO dimension(s): {which SLO type the incident affects}
- SRP SLO match: {does the SRP define the relevant SLO? Yes / No / Partial}
- DCR metric in SRP: {metric from DCR §5 — present in SRP / not present in SRP}
- Breach threshold context: {are DCR symptom values in breach territory per SRP §3?}

### Known Failure Mode Context
- SRP watch items relevant: {list any relevant watch items, or "None relevant"}
- Burn rate alert alignment: {would the SRP burn rate alerts have fired at the DCR signal values?}

### SRP Staleness
- SRP age: {approximate age}
- Staleness risk: {Low / Medium / High — basis}

### Overall Assessment
**Consistent / Inconsistent / Partially consistent**

**Flags (if any):**
1. {specific mismatch or concern}
2. {specific mismatch or concern}

**Recommendation:**
{Proceed with this SRP as context / Verify SRP is for the correct service / Consider whether SRP revision is needed}
```

---

## Behavioral Rules

- This check is informational — it does not block INR generation; it surfaces risks for the investigation team
- Do not assess whether the DCR is correctly completed — use the DCR validator for that
- Flag mismatches even if they might have an innocent explanation — the human investigator decides whether to act
- Do not recommend SRP revision as part of this check — SRP revision follows its own protocol (RRK playbook §SRP Revision Protocol)
