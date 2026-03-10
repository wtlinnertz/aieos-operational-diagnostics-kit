# Diagnostic Context Record

---

## 1. Document Control

| Field | Value |
|-------|-------|
| DCR ID | DCR-{SERVICE}-{NNN} |
| Date and Time | {YYYY-MM-DD HH:MM UTC} |
| Severity | SEV1 / SEV2 / SEV3 / SEV4 |
| Incident Owner | {named individual — not a team} |
| Status | Draft / Validated / Frozen |
| Governance Model Version | 1.0 |
| Prompt Version | N/A |
| Spec Version | {spec version} |
| Principles Version | {principles file versions} |

---

## 2. Service Identification

| Field | Value |
|-------|-------|
| Service Name | {specific service or component name} |
| Environment | production / staging / development / multi-environment |
| SRP Reference | {SRP-{SERVICE}-{NNN} if frozen SRP exists, or "No SRP"} |

---

## 3. Symptom Description

**Incident Start Time:** {YYYY-MM-DD HH:MM UTC}

**Symptoms:**

{Describe what is wrong specifically. Each symptom must have a measurable or observable characteristic — not "service is slow" but "notification delivery success rate dropped from 99.7% to 71% at 14:22 UTC." List each distinct symptom.}

---

## 4. Scope and Impact

**Impact Scope:** all users / subset of users / internal only / unknown

**Scope Details:**

{If "subset of users": describe the subset specifically. If "all users": state all users affected. If "internal only": state what internal systems or teams are affected. If "unknown": state unknown and why.}

---

## 5. Observable Signals

{List each signal. Include the signal source (monitoring system name, user report channel, or engineer name). At least one signal must be documented.}

| Signal | Source | Value / Detail |
|--------|--------|---------------|
| {alert name or metric name} | {monitoring system or person} | {current value or observation} |
| {alert name or metric name} | {monitoring system or person} | {current value or observation} |

---

## 6. Recent Changes

{Document all changes to this service or its dependencies in the last 72 hours. If no changes were made, write "No changes in last 72h" explicitly.}

| Change | Type | When | Who |
|--------|------|------|-----|
| {description} | deploy / config / infra / dependency | {YYYY-MM-DD HH:MM} | {name} |

---

## 7. Freeze Declaration

By freezing this record, the incident owner confirms:
- The service, environment, and incident owner are correctly identified
- The symptoms are described specifically and the start time is accurate
- At least one observable signal is documented with its source
- Recent changes in the last 72 hours are documented, or "no changes in last 72h" is explicitly confirmed
- This record is complete and authorizes INR generation to begin

**Frozen by:** {name}
**Freeze date:** {YYYY-MM-DD}
**Status:** Frozen
