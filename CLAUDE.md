# CLAUDE.md — Operational Diagnostics Kit

## What This Repository Is

This is the **Operational Diagnostics Kit** — an AIEOS kit that governs structured investigation, hypothesis tracking, and postmortem analysis for production failures. It is Layer 8 of the AIEOS system. It is a reactive operational track triggered by production events, not SDLC progression. Its findings feed back into the Reliability & Resilience, Insight & Evolution, Engineering Execution, and Product Intelligence kits.

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
examples/              # Worked example (TaskFlow notification-service queue exhaustion)
tests/                 # Structural integrity checks
```

## Artifact Types

This kit produces four governed artifact types in sequence:

1. **Diagnostic Context Record (DCR)** — Entry gate. Validates enough context to begin structured diagnosis. Human-authored. ID format: `DCR-{SERVICE}-{NNN}`.
2. **Investigation Record (INR)** — Active diagnostic trail. Hypotheses, evidence, diagnostic steps, probable cause. Generated. ID format: `INR-{SERVICE}-{NNN}`.
3. **Postmortem Record (PMR)** — Post-resolution RCA. Contributing factors, SLO impact, corrective actions, lessons learned. Generated. ID format: `PMR-{SERVICE}-{NNN}`.
4. **Runbook (RB)** — Codified resolution procedure for a known failure class. Generated, versioned, optional. ID format: `RB-{SERVICE}-{NNN}`.

Each artifact type has exactly four governing files: spec, template, prompt, validator. The exception is DCR: as a human-authored entry gate, it does not have a generation prompt (per governance model entry gate exception FW-049). DCR has three governing files (spec, template, validator).

Note: DCR is human-authored (no AI generation session) — consistent with the SRER/RER/KER entry gate pattern.

## Utility Prompts

Four utility prompts support the flow but do not produce governed artifacts:

- **Hypothesis Generation** (`hypothesis-generation-prompt.md`) — Generates candidate causes from DCR context with evidence requirements for each
- **Diagnostic Checkpoint** (`diagnostic-checkpoint-prompt.md`) — Assesses investigation convergence; recommends continue/escalate/pivot
- **Pattern Match** (`pattern-match-prompt.md`) — Compares current DCR context against past PMRs/RBs for known failure signatures
- **DCR → SRP Consistency Check** (`dcr-srp-consistency-check-prompt.md`) — Validates DCR service identification matches known SRP baseline

## Key Rules

- **Specs are the source of truth** — prompts and validators reference specs, never inline rules
- **Validators judge, they do not help** — no suggestions, no redesign
- **Freeze before promote** — DCR must be frozen before INR generation; INR must be frozen before PMR generation
- **Separate generation and validation** — different AI sessions to prevent self-validation bias
- **No scope expansion** — downstream artifacts must not expand scope beyond upstream
- **No inferred information** — mark missing information explicitly, do not fill gaps
- **DCR is human-authored** — do not use AI to generate the DCR; complete the template from incident evidence
- **RB is optional** — only generate a Runbook when the PMR identifies a repeatable failure class with a known resolution procedure
- **Governance model sync** — `docs/governance-model.md` is a synchronized copy of `aieos-governance-foundation/governance-model.md` (canonical authority). Do not edit the kit copy directly; update `aieos-governance-foundation` first, then sync all kit copies to match exactly. See governance-model.md §15 for versioning and change protocol.
- **Engagement Record** — ODK maintains the Layer 8 section of the project's ER. Add artifact IDs as each artifact freezes. See `docs/playbook.md §Maintaining the Engagement Record` and `aieos-governance-foundation/docs/engagement-record-spec.md`.

## Artifact Flow

```
Trigger: SEV1/2 incident declared (or operator judgment)
         │
         ▼
Step 0: Diagnostic Context Record (DCR) — human-authored
         │ Validate → Freeze
         ▼
Step 1: Investigation Record (INR) — generated
         │ Validate → Freeze
         ▼
Step 2: Postmortem Record (PMR) — generated
         │ Validate → Freeze
         ▼
Step 3 (optional): Runbook (RB) — generated
         │ Validate → Freeze
         ▼
Cross-kit outputs: PMR → RRK (next RHR), EEK (corrective actions), PIK (recurring patterns), IEK (ER §8)
```

## Boundary Contracts

- **Upstream:** Incident evidence (alerts, monitoring data, responder notes) + frozen SRP from RRK (service baseline, SLO targets) + optionally frozen RR §7 from REK (recent changes) + optionally frozen SAD from EEK (service architecture). See `docs/entry-from-rrk.md` for the boundary briefing.
- **Downstream:** Produces frozen PMRs consumed by the next RRK RHR, and optionally frozen RBs for reuse in future incidents. PMR corrective actions may create EEK work items or PIK discovery intake context.

## Relationship to RRK Incident Records

RRK IR = lightweight operational record (required for all incidents, any severity). ODK adds depth for SEV1/2:
- **DCR**: enough context to begin structured diagnosis
- **INR**: how we figured out what was wrong
- **PMR**: what the organization learned

These are complementary, not overlapping. The IR and PMR cover different questions.

## File Naming

| Type | Pattern |
|------|---------|
| Spec | `{type}-spec.md` |
| Template | `{type}-template.md` |
| Prompt | `{type}-prompt.md` |
| Validator | `{type}-validator.md` |
| Utility Prompt | `{function}-prompt.md` |
| Intake Form | `{context}-intake-template.md` |

## When Working on This Kit

- Read the playbook (`docs/playbook.md`) for the full process definition
- Read the governance model (`docs/governance-model.md`) for structural rules
- Check `docs/how-to-use-with-ai.md` for session setup instructions
- Use `docs/session-setup.md` for per-artifact setup checklists and pre-flight gate checks
- Use `docs/troubleshooting.md` when a validator returns FAIL — maps each gate failure to a specific remediation
- Reference `examples/basic-operation/` for a complete worked example
- Arriving from RRK on a SEV1/2 trigger: see `docs/entry-from-rrk.md` for the boundary briefing

## Building or Auditing AIEOS Kits

- `aieos-governance-foundation/docs/kit-structure-standard.md` — compliance checklist for building and auditing kits
- `aieos-governance-foundation/docs/philosophy.md` — design rationale for governance model decisions
- `aieos-governance-foundation/docs/layer-model.md` — layer model and kit registry
