# Entry from Reliability & Resilience Kit (RRK)

**You are here because:** A SEV1/2 incident has been declared for a service managed in RRK, or a lower-severity incident with high organizational learning value has been escalated by operator judgment.

**What you must bring:**

| Artifact | Status Required | Where to Find It |
|----------|----------------|-----------------|
| Service Reliability Profile (SRP-{SERVICE}-{NNN}) | Frozen | RRK `docs/artifacts/` in the consuming project |
| Active or recent Incident Record (IR) | In progress or frozen | RRK `docs/artifacts/` — lightweight operational record |
| Release Record §7 (if incident is change-related) | Frozen (optional) | REK `docs/artifacts/` — for recent changes context |

**First artifact to generate in this kit:** Diagnostic Context Record (DCR) — human-authored, complete during or immediately after incident declaration

**Where to start:** `docs/session-setup.md` → "Diagnostic Context Record (DCR)"

**What changes at this boundary:**

- You shift from proactive operational management to structured failure investigation. The mode is reactive, time-pressured, and evidence-driven.
- The priority is diagnosis clarity, not documentation completeness. DCR is captured under pressure; completeness is validated after the incident is contained.
- The investigation must separate observations from interpretations. What you observed is not the same as what it means. The INR requires both, clearly labeled.

**Common mistakes at this transition:**

| Mistake | How to Avoid |
|---------|--------------|
| Creating the PMR before the INR is frozen | Investigation must complete before postmortem synthesis begins. INR freeze is a hard prerequisite for PMR generation. |
| Treating the DCR as a formality and skipping it during the incident | The DCR is the scope declaration that makes the INR coherent. Without it, the investigation lacks a defined boundary. |
| Concluding root cause as "human error" without systemic analysis | ODK requires identifying the systemic conditions that allowed the failure, not just the proximate cause. |

**If you arrived here without a complete upstream artifact:**

Stop. ODK requires an actual incident or observable operational failure. For proactive operational improvements without an incident trigger, return to RRK for SRP revision or initiate PIK discovery if the scope warrants it. Do not create a DCR speculatively — a declared incident or confirmed symptom is required.

---

*For the full entry flow, see `docs/playbook.md`.*
