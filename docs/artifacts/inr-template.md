# Investigation Record

---

## 1. Document Control

| Field | Value |
|-------|-------|
| INR ID | INR-{SERVICE}-{NNN} |
| Date and Time | {YYYY-MM-DD HH:MM UTC} |
| Status | Draft / Validated / Frozen |
| Governance Model Version | 1.0 |
| Prompt Version | {version of inr-prompt.md used} |
| Spec Version | {spec version} |
| Principles Version | {principles file versions} |

---

## 2. Context Reference

| Field | Value |
|-------|-------|
| DCR ID | DCR-{SERVICE}-{NNN} |
| DCR Status | Frozen |
| Service | {service name from DCR} |
| Environment | {environment from DCR} |

---

## 3. Hypothesis Register

{For each hypothesis considered during the investigation, complete a row. Hypotheses should be listed in the order they were considered.}

| # | Hypothesis | Evidence Required | Evidence Found | Status |
|---|-----------|------------------|---------------|--------|
| H-1 | {proposed cause} | {what would confirm or refute} | {what was actually observed, or "Not yet gathered"} | Active / Confirmed / Refuted / Superseded |
| H-2 | {proposed cause} | {what would confirm or refute} | {what was actually observed, or "Not yet gathered"} | Active / Confirmed / Refuted / Superseded |

---

## 4. Diagnostic Trail

{List diagnostic steps in chronological order. Each step must have a timestamp, action, and substantive finding.}

| Timestamp | Action | Finding |
|-----------|--------|---------|
| {YYYY-MM-DD HH:MM UTC} | {what was done — specific} | {what was observed — substantive} |
| {YYYY-MM-DD HH:MM UTC} | {what was done — specific} | {what was observed — substantive} |
| {YYYY-MM-DD HH:MM UTC} | {what was done — specific} | {what was observed — substantive} |

---

## 5. Probable Cause

{State the probable cause with a reference to the evidence in §4 that supports it. If the probable cause cannot be determined, state "Undetermined" and explain why it could not be established — what evidence is missing or what remains ambiguous.}

**Probable Cause:**

{Statement}

**Evidence Basis:**

{Reference to diagnostic trail steps or hypothesis evidence that supports this conclusion}

---

## 6. Immediate Actions

{Document all actions taken to restore service. These are actions that were taken — not actions that should be taken.}

**Actions Taken:**

| Time | Action | Taken By |
|------|--------|---------|
| {YYYY-MM-DD HH:MM UTC} | {what was done} | {name} |

**Restoration Confirmed:**

{How was restoration confirmed? Include the metric value, alert resolution, or manual verification result. If service is not yet restored at time of authoring, state current status explicitly.}

---

## 7. Open Questions

{List questions that remain unanswered and are relevant to understanding the full cause. If no open questions remain, write "None."

Open questions are investigation gaps — not improvement suggestions.}

1. {specific open question}
2. {specific open question}

---

## 8. Freeze Declaration

By freezing this record, the author confirms:
- The DCR referenced is in Frozen status
- The hypothesis register reflects all hypotheses considered, including refuted ones
- The diagnostic trail is chronologically accurate and findings are substantive
- The probable cause is grounded in evidence, or "Undetermined" with reasoning
- Immediate actions and restoration status are accurately documented
- This record is complete and authorizes PMR generation to begin

**Frozen by:** {name}
**Freeze date:** {YYYY-MM-DD}
**Status:** Frozen
