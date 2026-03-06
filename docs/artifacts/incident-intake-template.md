# Incident Intake Form

Use this form to gather the information needed to complete the Diagnostic Context Record (DCR). Complete this form as soon as an incident is declared. The DCR should be authored within 2 hours of declaration.

This form is not a governed artifact. It is an input to the DCR.

---

## Basic Information

**What is the name of the service or component experiencing the issue?**
{Answer}

**What environment is affected?** (production / staging / development / multi-environment)
{Answer}

**Who is the incident owner?** (named individual)
{Answer}

**What is the incident severity?** (SEV1 / SEV2 / SEV3 / SEV4 — refer to diagnostic-principles.md §4 for definitions)
{Answer}

**When did the incident start?** (date and time, include timezone)
{Answer}

---

## Symptom Description

**Describe what is wrong specifically.** What behavior is unexpected? How is it different from normal?
{Answer}

**What is the measurable or observable characteristic of the symptom?** (e.g., error rate X%, latency Pms, X users affected, queue depth Y)
{Answer}

---

## Scope and Impact

**Who is affected?** (all users / specific subset — describe the subset / internal users only / unknown)
{Answer}

**If a subset, describe the subset specifically:**
{Answer or N/A}

---

## Observable Signals

**What alerts have fired? List each by name and the monitoring system:**
{Answer}

**What metric values do you see that are abnormal?** (include metric name and current value)
{Answer}

**Have users reported issues? If so, what specifically did they report?**
{Answer or No user reports}

**Any other observable signals?**
{Answer or None}

---

## Recent Changes

**What changes were made to this service or its dependencies in the last 72 hours?** Include deployments, configuration changes, infrastructure changes, and dependency updates.

| Change | Type | When | Who |
|--------|------|------|-----|
| {description} | {deploy / config / infra / dependency} | {datetime} | {name} |

*If no changes were made in the last 72 hours, write: "No changes in last 72h"*

---

## SRP Reference

**Does a frozen Service Reliability Profile (SRP) exist for this service?**
{Yes — SRP ID: {SRP-XXX} / No SRP}

**If yes, what are the relevant SLO targets from the SRP?**
{Answer or N/A}

---

## Additional Context

**Anything else relevant to the investigation that doesn't fit the above fields?**
{Answer or None}
