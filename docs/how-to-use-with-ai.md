# How to Use This Kit with AI

This guide explains how to set up AI sessions for each step in the Operational Diagnostics Kit workflow. Follow the session setup instructions precisely — incorrect session setup is the most common cause of poor artifact quality.

---

## Core Discipline

**One artifact per session.** Do not generate multiple artifacts in the same session.

**Separate generation and validation.** Always validate in a new session. Never ask the AI that generated an artifact to validate it — this produces self-validation bias.

**Include full frozen documents.** Do not summarize upstream artifacts. Provide the complete document.

**DCR is human-authored.** Do not use AI to complete the DCR. Complete the template yourself from the Incident Intake Form.

---

## DCR — Human-Authored (No AI Generation Session)

The DCR is human-authored. Do not use AI to complete it. Complete the template yourself using information from the incident evidence and the Incident Intake Form.

**Validation session setup:**
```
Documents to provide:
1. The completed DCR (full document)
2. docs/specs/dcr-spec.md

Prompt:
"Validate this Diagnostic Context Record against the DCR spec.
Use only the spec as the source of truth for pass/fail criteria.
Do not suggest improvements. Judge only what is explicitly present.
Output JSON using the format defined in docs/validators/dcr-validator.md."
```

---

## INR — Generation Session

**Pre-generation utility prompts (run separately, before INR generation):**

Pattern Match (optional — check for known failure signatures):
```
Documents to provide:
1. Completed DCR (full document)
2. Past frozen PMRs for this service (up to 5 most recent, full documents)
3. Optional: Frozen RBs for this service

Prompt:
"Run the pattern match analysis using docs/prompts/pattern-match-prompt.md.
Compare the current DCR against past PMRs and RBs for known failure signatures.
Output using the format in the prompt."
```

Hypothesis Generation (optional — seed the hypothesis register):
```
Documents to provide:
1. Completed DCR (full document)
2. Optional: Frozen SRP

Prompt:
"Generate candidate hypotheses for the investigation using docs/prompts/hypothesis-generation-prompt.md.
Output structured hypothesis entries suitable for the INR §3 Hypothesis Register."
```

**INR generation session setup:**
```
Documents to provide:
1. Frozen DCR (full document)
2. Incident evidence (alert records, metric data, log excerpts, responder notes — everything gathered)
3. docs/specs/inr-spec.md
4. docs/artifacts/inr-template.md
5. Optional: Frozen SRP (full document)
6. Optional: Frozen SAD from EEK (full document)

Prompt:
"Generate an Investigation Record for this incident using the provided evidence.
Follow the prompt in docs/prompts/inr-prompt.md.
Use the template exactly. Satisfy all hard gates in the spec.
Do not invent hypotheses or diagnostic steps without evidence basis.
Mark missing information with [MISSING: reason]. Output pure Markdown."
```

**After generation:** Review the INR. Confirm:
- The hypothesis register reflects all hypotheses actually considered, including those that were refuted
- The diagnostic trail is chronologically accurate and findings are substantive (not "checked logs — nothing found")
- The probable cause is grounded in specific evidence references

**If investigation stalls mid-incident, use the diagnostic checkpoint:**
```
Documents to provide:
1. Current INR draft (in-progress, not frozen)
2. Optional: Frozen DCR
3. Optional: Current monitoring data snapshot

Prompt:
"Assess the current state of this investigation using docs/prompts/diagnostic-checkpoint-prompt.md.
Evaluate hypothesis convergence, diagnostic trail progress, and gaps.
Output the checkpoint assessment in the format defined in the prompt."
```

**Validation session setup:**
```
Documents to provide:
1. The generated INR (full document)
2. docs/specs/inr-spec.md

Prompt:
"Validate this Investigation Record against the INR spec.
Use only the spec as the source of truth for pass/fail criteria.
Do not suggest improvements. Judge only what is explicitly present.
Output JSON using the format defined in docs/validators/inr-validator.md."
```

---

## PMR — Generation Session

**Session setup:**
```
Documents to provide:
1. Frozen INR (full document)
2. Post-incident evidence: full event timeline, monitoring data export for the incident window,
   post-incident review notes (everything gathered after resolution)
3. docs/specs/pmr-spec.md
4. docs/artifacts/pmr-template.md
5. docs/principles/diagnostic-principles.md
6. Optional: Frozen SRP (for SLO impact calculation)

Prompt:
"Generate a Postmortem Record for this incident using the provided inputs.
Follow the prompt in docs/prompts/pmr-prompt.md.
Apply blameless postmortem principles (diagnostic-principles.md §2).
Calculate SLO impact from the frozen SRP if provided.
Every corrective action must have a named owner, specific date, and tracking reference.
Mark missing information with [MISSING: reason]. Output pure Markdown."
```

**After generation:** Review the PMR. Confirm:
- Root cause analysis reflects what was actually learned, not initial speculation
- Corrective actions have real owners, real deadlines, and real tracking references
- SLO impact is calculated correctly against the SRP error budget
- Lessons are specific — not "improve monitoring" but "monitoring was configured only at saturation, providing no lead time"
- §8 Cross-Kit Actions accurately identifies whether EEK escalation, PIK re-engagement, or RB generation is warranted

**Validation session setup:**
```
Documents to provide:
1. The generated PMR (full document)
2. docs/specs/pmr-spec.md

Prompt:
"Validate this Postmortem Record against the PMR spec.
Use only the spec as the source of truth for pass/fail criteria.
Do not suggest improvements. Judge only what is explicitly present.
Output JSON using the format defined in docs/validators/pmr-validator.md."
```

---

## RB — Generation Session

**Session setup:**
```
Documents to provide:
1. Frozen PMR (full document)
2. Frozen INR (full document, recommended)
3. docs/specs/rb-spec.md
4. docs/artifacts/rb-template.md
5. Optional: Frozen SRP (for SLO thresholds in trigger conditions)

Prompt:
"Generate a Runbook for the failure class identified in this PMR.
Follow the prompt in docs/prompts/rb-prompt.md.
Use only documented resolution steps from the INR and PMR — do not invent procedures.
Show commands where applicable. Include specific observable criteria for verification and escalation.
Output pure Markdown."
```

**After generation:** Review the RB. Confirm:
- Trigger conditions are observable from monitoring without investigation
- Resolution steps are specific enough to execute under pressure — any vague step is a failure
- Commands are shown where applicable (not paraphrased)
- Verification steps have specific observable criteria (not "monitor for stability")
- Escalation conditions are observable and name an escalation target

**Validation session setup:**
```
Documents to provide:
1. The generated RB (full document)
2. docs/specs/rb-spec.md

Prompt:
"Validate this Runbook against the RB spec.
Use only the spec as the source of truth for pass/fail criteria.
Do not suggest improvements. Judge only what is explicitly present.
Output JSON using the format defined in docs/validators/rb-validator.md."
```

---

## Utility Prompts

### DCR → SRP Consistency Check (after DCR, before INR)

```
Documents to provide:
1. Completed DCR (frozen or in-progress)
2. Frozen SRP for the service

Prompt:
"Run the DCR → SRP consistency check using docs/prompts/dcr-srp-consistency-check-prompt.md.
Compare the DCR service identification and symptoms against the SRP baseline.
Output the consistency check assessment in the format defined in the prompt."
```

---

## Troubleshooting

**Validator returns FAIL on diagnostic_trail**
The most common cause is findings stated as "nothing found" or "checked X" without a substantive result. Even negative findings must specify what was searched and what the absence implies. Provide more detailed responder notes to the generation session.

**Validator returns FAIL on corrective_actions**
Check that actions have specific owners (not team names), specific dates (not "next sprint"), and actual tracking references (not "TBD"). Update the PMR directly for these fields.

**Validator returns FAIL on lessons_learned**
Generic lessons fail validation. Review the lessons against the root cause — each lesson should name a specific condition that existed and will now be changed. Regenerate with more specific post-incident review notes.

**RB trigger conditions are too vague**
The generation session may have paraphrased alerts or metrics. Provide exact alert names and metric names from your monitoring system in the PMR or as supplementary evidence in the RB generation session.

**PMR SLO impact section fails**
Ensure the SRP was provided to the PMR generation session. Without the SRP, the prompt cannot calculate error budget consumption. Provide the SRP and regenerate.
