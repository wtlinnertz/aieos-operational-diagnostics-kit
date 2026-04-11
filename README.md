# aieos-operational-diagnostics-kit

**Layer 8 of the AIEOS system — Operational Diagnostics**

This kit governs structured diagnosis and resolution of production failures. It adds diagnostic depth for SEV1/2 incidents and incidents with organizational learning value, producing governed artifacts that feed back into the Reliability & Resilience, Insight & Evolution, Engineering Execution, and Product Intelligence kits.

---

## What This Kit Does

The Reliability & Resilience Kit (Layer 6) handles proactive reliability governance (SLOs, error budgets, health reviews) and lightweight incident records. But the *active diagnostic process* — structured investigation, hypothesis tracking, runbook codification, and deep postmortem analysis — has no home in the SDLC loop. This kit governs that gap:

- **Structured investigation** — Hypothesis register, evidence tracking, diagnostic trail
- **Root cause analysis** — From probable cause during the incident to authoritative RCA after resolution
- **Organizational learning** — Postmortem records that document lessons and drive corrective action
- **Runbook codification** — Transforming hard-won diagnostic knowledge into reusable procedures

This kit is a reactive operational track — triggered by production events, not SDLC progression.

---

## Artifact Types

This kit produces four governed artifact types:

| Step | Artifact | ID Format | Purpose |
|------|----------|-----------|---------|
| 0 | Diagnostic Context Record (DCR) | `DCR-{SERVICE}-{NNN}` | Entry gate: validates enough context to begin structured diagnosis |
| 1 | Investigation Record (INR) | `INR-{SERVICE}-{NNN}` | Active diagnostic trail — hypotheses, evidence, diagnostic steps, probable cause |
| 2 | Postmortem Record (PMR) | `PMR-{SERVICE}-{NNN}` | Post-resolution RCA — contributing factors, SLO impact, corrective actions, lessons |
| 3 | Runbook (RB) | `RB-{SERVICE}-{NNN}` | Codified resolution procedure for a known failure class; versioned |

Each governed artifact type has exactly four governing files: spec, template, prompt, validator.

---

## Quick Start

1. Read `docs/playbook.md` — the complete process definition
2. Read `docs/how-to-use-with-ai.md` — session setup and AI tool guidance
3. See `examples/basic-operation/` — a worked example (TaskFlow notification-service queue exhaustion)

---

## Repository Structure

```
docs/
  principles/          # Organizational diagnostic policy (input material)
  specs/               # Content rules and hard gates per artifact type
  artifacts/           # Templates and intake forms
  prompts/             # AI generation + utility prompts
  validators/          # Quality gate definitions
  playbook.md          # End-to-end process definition
  index.md             # Documentation entry point
  how-to-adapt.md      # Organizational adoption guidance
  how-to-use-with-ai.md # AI tool usage guide
  governance-model.md  # AIEOS structural rules (reference)
examples/
  basic-operation/     # Worked example: TaskFlow notification-service queue exhaustion
tests/
  kit-test-plan.md     # Structural integrity checks and flow scenarios
CLAUDE.md              # AI operating instructions
```

---

## Trigger

ODK is triggered by production events, not SDLC gate passage:

- **Required:** SEV1 or SEV2 incident declared
- **Optional:** Operator judgment for lower-severity incidents with learning value or recurring failure patterns

---

## AIEOS Layer

| Layer | Kit | Status |
|-------|-----|--------|
| 2. Product Intelligence | `aieos-product-intelligence-kit` | Built |
| 4. Engineering Execution | `aieos-engineering-execution-kit` | Built |
| 5. Release & Exposure | `aieos-release-exposure-kit` | Built |
| 6. Reliability & Resilience | `aieos-reliability-resilience-kit` | Built |
| 7. Insight & Evolution | `aieos-insight-evolution-kit` | Built |
| **8. Operational Diagnostics** | **`aieos-operational-diagnostics-kit`** | **Built** |

See `aieos-governance-foundation/docs/layer-model.md` for the full layer model.
