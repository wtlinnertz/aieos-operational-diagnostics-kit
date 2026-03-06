# AIEOS Governance Model

> **Canonical source.** This is the authoritative version of the AIEOS governance model.
> All AIEOS kits carry a synchronized copy in `docs/governance-model.md`.
> When this file changes, all kit copies must be updated to match.
> Kit copies must never diverge from this file.

This document defines the structural rules, taxonomy, and invariants that govern every kit in the AIEOS system. It is the single source of truth for how kits are built, how artifacts behave, and how kits connect.

Any kit that follows this model is structurally compatible with every other kit in the system.

---

## 1. The Kit Model

A **kit** is a self-contained repository that governs one layer of the organization's operating system. Each kit provides the rules, templates, prompts, and validators needed to produce and verify artifacts within its domain.

Every kit is:
- **Standalone** — its own repository, versioned independently
- **Self-documenting** — all rules are in the repo, not in tribal knowledge
- **AI-native** — designed to be operated by AI assistants under human oversight
- **Tool-agnostic** — no references to specific vendors in policy files (tool mappings go in bindings)

---

## 2. The Four-File System

Every artifact type within a kit is governed by exactly four files. Each file answers one question.

| File | Question | Responsibility |
|------|----------|---------------|
| **Spec** | What are the rules? | Content rules, format requirements, hard gates, quality criteria |
| **Template** | What is the structure? | Section headings, ordering, placeholders — structure only |
| **Prompt** | How should the AI behave? | Generation instructions, input requirements, behavioral constraints |
| **Validator** | How do we judge pass/fail? | Evaluation procedure, hard gate checks, output format |

### Separation of Concerns

- **Specs are the single source of truth.** Hard gates, content rules, and quality criteria are defined in specs. Prompts and validators reference specs — they never inline their own rules.
- **Templates define structure, not content.** A template contains section headings and placeholders. It does not contain content rules or quality criteria.
- **Prompts define behavior, not rules.** A prompt tells the AI what to do and what inputs to use. It references the spec for the actual rules.
- **Validators judge, they do not help.** A validator evaluates what is present against the spec. It does not suggest improvements, redesign artifacts, or expand scope.

### When Files Reference Each Other

```
Prompt → references → Spec (for content rules to satisfy)
Prompt → references → Template (for structure to follow)
Validator → references → Spec (for hard gates to evaluate against)
```

No other cross-references are permitted. Templates do not reference specs. Validators do not reference prompts.

---

## 3. Repository Structure

Every kit follows this directory layout:

```
aieos-{layer-name}-kit/
  docs/
    principles/      # Organizational policy (input material, not governed artifacts)
    specs/           # Content rules and quality criteria per artifact type
    artifacts/       # Templates and intake forms
    prompts/         # AI generation prompts
    validators/      # Quality gate definitions
    playbook.md      # End-to-end process definition for this kit
    index.md         # Documentation entry point
    how-to-adapt.md  # Organizational adoption guidance
    how-to-use-with-ai.md  # AI tool usage guide
  examples/          # Worked examples demonstrating the full flow
  tests/             # Test plans and structural verification
  CLAUDE.md          # AI tool project instructions (Claude Code)
  README.md          # Repository overview
```

### Directory Responsibilities

| Directory | Contains | Does Not Contain |
|-----------|----------|-----------------|
| `principles/` | Organizational policy documents that feed artifact generation | Artifact specs, hard gates, or validation criteria |
| `specs/` | Authoritative content rules and hard gates per artifact type | Templates, prompts, or generation logic |
| `artifacts/` | Structural templates and human intake forms | Content rules or quality criteria |
| `prompts/` | AI behavior instructions for generation and utility tasks | Inline rules (must reference specs) |
| `validators/` | Evaluation procedures and judgment criteria | Suggestions, redesign, or scope expansion |

---

## 4. Naming Conventions

### File Naming

| File Type | Pattern | Example |
|-----------|---------|---------|
| Spec | `{type}-spec.md` | `prd-spec.md` |
| Template | `{type}-template.md` | `prd-template.md` |
| Intake form | `{context}-template.md` | `architecture-context-template.md` |
| Generation prompt | `{type}-prompt.md` | `prd-prompt.md` |
| Utility prompt | `{function}-prompt.md` | `codebase-analysis-prompt.md` |
| Validator | `{type}-validator.md` | `prd-validator.md` |
| Example artifact | `{nn}-{type}.md` | `01-prd.md` |

### Kit Naming

Kit repositories follow the pattern: `aieos-{layer-name}-kit`

| Layer | Repository Name |
|-------|----------------|
| Strategic Direction | `aieos-strategic-direction-kit` |
| Product Intelligence | `aieos-product-intelligence-kit` |
| Flow Control | `aieos-flow-control-kit` |
| Engineering Execution | `aieos-engineering-execution-kit` |
| Release & Exposure | `aieos-release-exposure-kit` |
| Reliability & Resilience | `aieos-reliability-resilience-kit` |
| Insight & Evolution | `aieos-insight-evolution-kit` |

### Artifact ID Format

`{TYPE}-{PROJECT}-{NNN}`

- `TYPE`: Uppercase abbreviation of the artifact type (e.g., `PRD`, `SAD`, `OKR`, `SLO`)
- `PROJECT`: Short project identifier
- `NNN`: Sequential number, zero-padded to 3 digits

Example: `PRD-PAYMENTS-001`, `SLO-API-003`

### Project Artifact Directory

Generated artifacts live in the consuming project's repository under `docs/sdlc/`:

```
my-app/
  docs/sdlc/
    00-product-brief.md          ← human input (intake form)
    01-prd.md                    ← generated, validated, frozen
    01-prd-validation.json       ← final passing validator result
    02-acf.md
    ...
  docs/engagement/
    er-{initiative}.md           ← Engagement Record (cross-layer index)
```

The `00-` prefix groups human inputs before generated artifacts. Numbered prefixes follow artifact flow order.

Engagement Records live in `docs/engagement/`, not `docs/sdlc/`. They are maintained throughout the engagement lifecycle by following each kit's playbook, not generated by a single kit. See §16 for Engagement Record governance.

---

## 5. Validator Output Format

All validators across all kits produce JSON with this schema:

```json
{
  "status": "PASS | FAIL",
  "summary": "<one sentence verdict>",
  "hard_gates": { "<gate_name>": "PASS | FAIL" },
  "blocking_issues": [{ "gate": "", "description": "", "location": "" }],
  "warnings": [{ "description": "", "location": "" }],
  "completeness_score": "<0-100>"
}
```

### Interpretation Rules

- Any hard gate failure means `FAIL` — no exceptions.
- `blocking_issues` identifies exactly what failed and where.
- `warnings` are non-blocking observations. They do not affect the status.
- `completeness_score` is advisory. It does not override hard gate results.
- Validators evaluate only what is explicitly present. They do not infer, assume, or speculate.
- Ambiguity in the artifact is a failure condition, not a judgment call.

---

## 6. Artifact Promotion Model

### Promotion Rules

1. Artifacts are **generated**, **validated**, and **frozen** — in that order.
2. A frozen artifact is immutable. Changes require the re-entry protocol.
3. Downstream artifacts may not expand scope beyond what upstream artifacts define.
4. Prerequisite artifacts must be frozen before downstream generation begins.
5. Generation and validation happen in **separate AI sessions** to prevent self-validation bias.

### Artifact Provenance

Every artifact template must include two provenance fields in its Document Control section:

- `Governance Model Version` — The version of this governance model in effect when the artifact was generated. Retrieve from §15 of this document. Current value: `1.0`.
- `Prompt Version` — The version of the generation prompt used to produce this artifact. Retrieve from the prompt file's version header. Use `N/A` for human-authored entry gates and intake forms.

These fields are required for all artifacts generated after this governance model version. Existing frozen artifacts are grandfathered — no retrofitting required. Provenance tracking is forward-looking only.

### Lifecycle States

Artifacts progress through a defined set of lifecycle states:

| State | Meaning |
|-------|---------|
| Draft | Artifact is being generated or authored |
| Validated | Artifact has passed all hard gates |
| Freeze Pending | Validation passed; human freeze review in progress |
| Frozen | Artifact is authoritative input for downstream artifacts |
| Deprecated | Artifact was Frozen; the system or service it governed is no longer active |
| Abandoned | Artifact did not complete the lifecycle; initiative was cancelled before freeze |

`Deprecated` and `Abandoned` are terminal states. An artifact in either state is retained for audit purposes and must not be deleted. `Deprecated` does not mean invalid — it means the governed system has ended its operational life. `Abandoned` does not mean failed — it means the initiative was discontinued before the artifact reached a freeze.

### Freeze Semantics

A freeze means:
- The artifact has passed its validator (all hard gates PASS).
- A human has reviewed and approved it.
- It is now the authoritative input for downstream artifacts.
- It may not be modified without triggering impact analysis and re-validation of all downstream artifacts.

### Freeze Pending Status

`Freeze Pending` is an optional intermediate status between `Validated` and `Frozen`. An artifact is Freeze Pending when:
- It has passed all hard gates (all validator gates PASS).
- A human freeze review is in progress (review scheduled, awaiting approval).

Freeze Pending is opt-in and informational. Downstream teams may choose to begin preparatory work against a Freeze Pending artifact, but must acknowledge the pre-freeze status explicitly. If the freeze review results in a change, any preparatory downstream work must be discarded and restarted from the updated artifact.

Freeze Pending does not substitute for a freeze. An artifact is not authoritative input for downstream generation until it is Frozen.

### Non-Material Amendment

A frozen artifact may be corrected in place without re-validation when **all** of the following criteria are met:

1. The correction does not affect any field evaluated by a hard gate.
2. The correction does not change scope, decisions, owners, or technical specifications.
3. The correction does not affect any field referenced by a downstream artifact.

**Procedure:** Make the correction and add an Amendment Log entry to the artifact's Document Control section (date, what changed, materiality criterion cited, authorized by). No re-validation is required.

**If there is any ambiguity** about whether a change is non-material, it is material. The amendment path must not become a workaround for the re-entry protocol. Material changes trigger the Re-Entry Protocol.

### Re-Entry Protocol

When a frozen artifact must change:
1. Run impact analysis to assess downstream effects.
2. Modify the artifact.
3. Re-validate the modified artifact.
4. Re-validate all affected downstream artifacts.
5. Obtain human approval at each step.

### Project-Level Artifacts

Some AIEOS artifacts belong to the consuming project rather than to any single kit. They span multiple layers and are maintained incrementally as the engagement progresses.

**Engagement Record (ER)** is the only project-level artifact in the current AIEOS system. It captures the full artifact lineage and key outcomes for a single initiative across all layers.

- **Format:** Defined in `aieos-governance-foundation/docs/engagement-record-spec.md`
- **Location:** `{project}/docs/engagement/er-{initiative}.md` in the consuming project
- **ID format:** `ER-{INITIATIVE}-{NNN}`
- **Governance:** Not governed by a four-file system. No generation prompt or validator. Operators maintain it by following each kit's playbook instructions.
- **Retention:** Retained permanently. When an initiative ends, the ER status is updated to `Deprecated` or `Abandoned` (matching the terminal state of its governing artifacts). ERs are never deleted.

Project-level artifacts are not subject to the four-file completeness invariant (§13). They are operational records, not kit-governed artifacts.

---

## 7. The Playbook

Every kit has a `playbook.md` that defines:
- The artifact flow (the non-negotiable order of generation and validation)
- The inputs and outputs of each step
- The freeze points
- The re-entry protocol for that kit's artifacts
- Any kit-specific iteration or escalation rules

The playbook is the process definition. It is not a tutorial — that's what `how-to-use-with-ai.md` is for.

---

## 8. Principles and Standards

### What Principles Are

Principles are **organizational policy documents** that define how the organization operates within the kit's domain. They are input material for artifact generation — not governed artifacts themselves.

Principles answer: **"What standards does this organization hold?"**

### What Principles Are Not

Principles do not define:
- What makes an artifact document good (that's what specs do)
- How AI should behave when generating (that's what prompts do)
- How to judge pass/fail (that's what validators do)

### Principles Structure

Each kit's `principles/` directory contains domain-relevant policy files. Examples:

| Kit | Principle Files |
|-----|----------------|
| Engineering Execution | `code-craftsmanship.md`, `product-craftsmanship.md` |
| Release & Exposure | `deployment-policy.md`, `progressive-delivery-policy.md` |
| Reliability & Resilience | `slo-policy.md`, `incident-management-policy.md` |

Principles feed into the kit's context files (ACF, DCF, or kit-specific equivalents), which translate policy into enforceable guardrails.

### Principle File Versioning

Every principle file must carry a version field in its header. Changes to principle files follow a categorized versioning protocol (minor, significant, breaking) defined in `aieos-governance-foundation/docs/principle-file-standard.md`. That document is the authoritative reference for version field format, change categories, and the enforcement mapping requirement.

---

## 9. Intake Forms

### Purpose

Intake forms bridge human intent and AI generation. They structure human thinking into machine-consumable input so the AI has concrete material to work with — not vague intent.

### Design Rules

- Intake forms are **templates with structured questions**, not blank pages.
- They may be filled by humans directly or pre-filled by analysis prompts (e.g., codebase analysis for brownfield projects).
- Pre-filled forms must be reviewed and edited by a human before use as generation input.
- Intake forms live in `docs/artifacts/` alongside the artifact templates they feed.

### Brownfield Support

For kits operating on existing systems (not greenfield), provide:
1. An **analysis prompt** that examines the current state and produces structured output.
2. The analysis output maps to the intake form's sections.
3. The human reviews, corrects, and approves before using as input.

This pattern prevents the AI from generating artifacts based on assumptions about existing systems.

---

## 10. Inter-Kit Interfaces

### Handoff Contracts

When one kit's output becomes another kit's input, the relationship is defined as a **handoff contract**. Each kit's playbook documents:

- **Upstream**: What this kit accepts as input and from which kit.
- **Downstream**: What this kit produces as output and which kit consumes it.

### Contract Format

A handoff contract specifies:
- The artifact type being handed off
- The required state (must be frozen and validated)
- The specific sections or fields the downstream kit depends on
- What happens if the upstream artifact changes (re-entry trigger)

### Example: Engineering Execution ↔ Product Intelligence

```
Product Intelligence Kit
  Output: Frozen Discovery PRD (validated, approved)

Engineering Execution Kit
  Input: Frozen PRD (from Product Intelligence Kit or direct human input)
  Dependency: PRD §1 Problem Statement, §2 Goals, §3 Non-Goals, §10 Acceptance Criteria
  Re-entry trigger: Any change to PRD goals, scope, or acceptance criteria
```

### Cross-Kit Consistency

When artifacts from different kits reference each other, consistency must be verified at handoff boundaries. Each kit is responsible for validating that its inputs meet its intake requirements — it does not trust upstream kits blindly.

---

## 11. AI Operating Rules

### Generation Rules

When generating artifacts:
- Output pure Markdown.
- Use the template exactly as written — do not add or remove sections.
- Satisfy all content rules, format requirements, and hard gates defined in the corresponding spec.
- Do not infer missing information — mark it explicitly.
- Do not expand scope beyond upstream artifacts.
- Non-goals are enforceable — do not violate them.

### Validation Rules

When validating artifacts:
- Evaluate against the hard gates and content rules defined in the corresponding spec.
- Be strict — ambiguity is a failure condition.
- Do not redesign or suggest solutions.
- Evaluate only what is explicitly present.
- Any hard gate failure means FAIL.

### Session Discipline

- One artifact per session for generation.
- Separate sessions for generation and validation.
- Include the full frozen upstream artifacts — do not summarize.
- Include the spec, prompt, and template for generation sessions.
- Include the spec and validator for validation sessions.

---

## 12. Tool Bindings

### Policy vs Implementation

Kits define **policy** — the rules, constraints, and quality criteria. When a kit must reference specific tools (Jira, ServiceNow, LaunchDarkly, etc.), the tool-specific details go in a **bindings file**, not in the policy files.

### Bindings Structure

```
docs/
  bindings/
    jira-mapping.md           # Maps kit concepts to Jira fields/workflows
    servicenow-mapping.md     # Maps kit concepts to ServiceNow
    launchdarkly-mapping.md   # Maps feature flag policy to LaunchDarkly
```

When the organization changes tools, only bindings files are updated. Specs, templates, prompts, and validators remain unchanged.

Bindings are not governed artifacts — they have no spec, validator, or prompt. They are reference material maintained by the team responsible for the tooling.

---

## 13. Kit Invariants

The following rules apply to every kit in the AIEOS system. Violating these breaks the system's guarantees.

### Structural Invariants

1. **Four-file completeness** — Every artifact type has exactly four files: spec, template, prompt, validator.
2. **Specs are the single source of truth** — Prompts and validators reference specs. Rules are never inlined.
3. **Validators are hard gates** — Ambiguity is failure. Validators do not help, suggest, or redesign.
4. **Non-goals are enforceable** — Validators block violations. Scope expansion is not permitted.

### Process Invariants

5. **Freeze before promote** — Upstream artifacts must be frozen before downstream generation.
6. **Separate generation and validation** — Different AI sessions to prevent self-validation bias.
7. **Human in the loop** — Every freeze requires human approval. AI does not self-approve.
8. **Impact before re-entry** — Changes to frozen artifacts require impact analysis first.

### Quality Invariants

9. **No silent modification** — Do not change existing constraints without explicit acknowledgment.
10. **No scope expansion** — Downstream artifacts may not introduce scope that upstream artifacts do not define.
11. **No inferred information** — If information is missing, mark it explicitly. Do not fill gaps with assumptions.

---

## 14. Adopting This Model

### For a New Kit

1. Create the repository following the naming convention (`aieos-{layer-name}-kit`).
2. Set up the directory structure from §3.
3. Write the playbook first — define the artifact flow, freeze points, and inputs/outputs.
4. For each artifact type, write files in this order: validator → spec → template → prompt. Starting with the validator forces you to define "what does good look like?" before generating anything.
5. Create intake forms for any artifact that requires human input.
6. Write a worked example that exercises the full flow.
7. Run structural integrity checks (four-file completeness, cross-reference verification, naming compliance).

### For an Existing Kit

1. Audit against the structural invariants (§13).
2. Identify any specs with inline rules in prompts or validators — extract to specs.
3. Verify naming conventions match §4.
4. Ensure the playbook documents the full flow including freeze points and re-entry.
5. Add inter-kit interface documentation if the kit has upstream or downstream dependencies.

### Starting Order

Do not build all kits simultaneously. Start with 2–3 kits that have direct handoff relationships to prove the inter-kit contract model:

1. **Engineering Execution Kit** (proven — this is the reference implementation)
2. **Product Intelligence Kit** (direct upstream — its outputs become Engineering Execution inputs)
3. **Release & Exposure Kit** (direct downstream — consumes Engineering Execution outputs)

Extract shared governance to a dedicated repo only when duplication across 3+ kits becomes painful.

---

## 15. Versioning and Evolution

### Kit Versioning

Each kit is versioned independently using semantic versioning:
- **Major**: Breaking changes to artifact structure, hard gates, or inter-kit contracts.
- **Minor**: New artifact types, additional hard gates, new intake forms.
- **Patch**: Clarifications, typo fixes, example updates.

### Governance Model Versioning

This document is versioned as part of the `aieos-governance-foundation` repository. The canonical version lives at `aieos-governance-foundation/governance-model.md`. All kit copies must remain synchronized with that file.

Current version: `1.0`

### Change Protocol

Changes to the governance model require:
1. Update `aieos-governance-foundation/governance-model.md` first.
2. Impact analysis across all kits that reference it.
3. Review by kit maintainers.
4. Update all kit copies of `docs/governance-model.md` to match exactly.
5. Migration guidance for breaking changes.

---

## 16. Prompt Evolution

### Purpose

Prompt evolution is the mechanism by which AIEOS improves its artifact generation quality over time. When cross-engagement analysis (via Portfolio Evolution Signals) identifies patterns suggesting a generation prompt produces systematically weak outputs, a targeted prompt improvement is proposed, reviewed, approved, and deployed.

### Prompt Evolution Log

Every kit that contains generation prompts must maintain a Prompt Evolution Log at `docs/prompts/prompt-evolution-log.md`. This file records every version change to every generation prompt in the kit.

**Required columns:** Prompt | From | To | Date | Triggered By | Change Summary | Approved By | Amendment

- `Triggered By` cites the Portfolio Evolution Signal (PES-NNN) and proposal number that motivated the change, or `Initial release` for the v1.0 entry.
- `Amendment` cites the governance amendment record (AM-NNN) from the §15 change protocol, or `—` for the initial entry.

The Prompt Evolution Log is not a governed artifact. It has no spec, prompt, or validator. Its format is defined here. It is maintained by kit operators whenever a prompt version increments.

### Improvement Loop

Prompt improvements follow this sequence:

1. **Observe** — Engagement Records capture gate failures, pivot decisions, and outcome verdicts as engagements run.
2. **Synthesize** — A Portfolio Evolution Signal (PES) aggregates across multiple Engagement Records to identify cross-initiative patterns.
3. **Propose** — PES §6 produces specific, evidence-grounded improvement proposals referencing the affected prompt file and observed pattern.
4. **Approve** — A human reviewer approves the proposal. Approved proposals enter the §15 governance amendment process.
5. **Deploy** — The prompt file is updated; its version header increments. A Prompt Evolution Log entry is added.
6. **Validate** — Future engagements use the improved prompt. The next PES can assess whether the improvement reduced the pattern frequency.

### Constraints

- Prompts are never modified autonomously. All changes require human approval and a §15 amendment record.
- Prompt version increments are reflected in the `Prompt Version` field of all artifacts subsequently generated by that prompt (§6 Artifact Provenance).
- The connection between a generated artifact's `Prompt Version` field and the Prompt Evolution Log entry for that version provides a complete audit trail from artifact to the pattern that motivated the generating prompt's design.

---

## Summary

The AIEOS governance model ensures every kit is structurally consistent, independently operable, and safely connectable. The rules are intentionally rigid where they matter (file taxonomy, validation semantics, promotion model) and intentionally flexible where they should be (domain-specific artifacts, organizational principles, tool bindings).

Adapt the **edges**, not the **core**.
