# Operational Diagnostics Kit — Documentation Index

This kit governs structured diagnosis and resolution of production failures. It is Layer 8 of the AIEOS system — a reactive operational track triggered by production events.

---

## Start Here

| Document | Purpose |
|----------|---------|
| `playbook.md` | End-to-end process: trigger → DCR → INR → PMR → RB → cross-kit outputs |
| `how-to-use-with-ai.md` | AI session setup instructions for each step |
| `how-to-adapt.md` | Guide for adapting this kit to your organization |
| `governance-model.md` | AIEOS structural rules (synchronized copy) |

---

## Artifact Governing Files

### Diagnostic Context Record (DCR)
Human-authored entry gate. Validates enough context to begin structured diagnosis.

| File | Purpose |
|------|---------|
| `specs/dcr-spec.md` | 5 hard gates |
| `artifacts/dcr-template.md` | Section structure |
| `artifacts/incident-intake-template.md` | Intake form for gathering DCR inputs |
| `validators/dcr-validator.md` | Pass/fail evaluation procedure |

*No generation prompt — DCR is human-authored.*

### Investigation Record (INR)
Active diagnostic trail — hypotheses, evidence, diagnostic steps, probable cause.

| File | Purpose |
|------|---------|
| `specs/inr-spec.md` | 6 hard gates |
| `artifacts/inr-template.md` | Section structure |
| `prompts/inr-prompt.md` | Generation instructions |
| `validators/inr-validator.md` | Pass/fail evaluation procedure |

### Postmortem Record (PMR)
Post-resolution root cause analysis — contributing factors, SLO impact, corrective actions, lessons.

| File | Purpose |
|------|---------|
| `specs/pmr-spec.md` | 6 hard gates |
| `artifacts/pmr-template.md` | Section structure |
| `prompts/pmr-prompt.md` | Generation instructions |
| `validators/pmr-validator.md` | Pass/fail evaluation procedure |

### Runbook (RB)
Codified resolution procedure for a known failure class. Optional; versioned.

| File | Purpose |
|------|---------|
| `specs/rb-spec.md` | 5 hard gates |
| `artifacts/rb-template.md` | Section structure |
| `prompts/rb-prompt.md` | Generation instructions |
| `validators/rb-validator.md` | Pass/fail evaluation procedure |

---

## Utility Prompts

| Prompt | Purpose |
|--------|---------|
| `prompts/hypothesis-generation-prompt.md` | Generate candidate causes from DCR context |
| `prompts/diagnostic-checkpoint-prompt.md` | Assess investigation convergence; recommend continue/escalate/pivot |
| `prompts/pattern-match-prompt.md` | Compare DCR context against past PMRs/RBs |
| `prompts/dcr-srp-consistency-check-prompt.md` | Validate DCR service against known SRP baseline |

---

## Principles

| File | Coverage |
|------|---------|
| `principles/diagnostic-principles.md` | Organizational policy for incident investigation and postmortem analysis |

---

## Examples

`examples/basic-operation/` — TaskFlow notification-service queue exhaustion incident
- `00-diagnostic-context-record.md` — DCR-NOTIF-001
- `01-investigation-record.md` — INR-NOTIF-001
- `02-postmortem-record.md` — PMR-NOTIF-001
- `03-runbook.md` — RB-NOTIF-001
- `validator-outputs/` — PASS JSON for all four artifacts

---

## Guides

| Document | Purpose |
|----------|---------|
| `session-setup.md` | Per-artifact setup checklists, pre-flight gate checks, and common failure reminders |
| `troubleshooting.md` | Gate failure remediation guide |
| `entry-from-rrk.md` | Boundary briefing when arriving from the Reliability & Resilience Kit (SEV1/2 trigger) |

---

## Tests

`tests/kit-test-plan.md` — S-01 through S-08 structural checks + F-00 (SEV2 investigation) + F-01 (runbook generation)
