# CONTRIBUTING

Thank you for your interest in contributing to **aieos-reliability-resilience-kit**.

This repository is a **public, employer-neutral toolkit** for AI-assisted SLO governance, incident documentation, and service reliability operations. Contributions are welcome, provided they follow the rules below.

---

## Guiding Principles

All contributions must uphold these principles:

- **AI generates, validators decide**
- **Artifacts are promoted, not rewritten**
- **SLOs are measurable commitments, not aspirations**
- **Evidence standards are non-negotiable**
- **Ambiguity is a failure condition**
- **Anonymization is mandatory**

If a contribution weakens these principles, it will not be accepted.

---

## What You Can Contribute

We welcome contributions in the following areas:

### Documentation
- Clarifications or improvements to existing docs
- New explanations that improve understanding of SLO governance or incident management concepts
- Diagrams that reflect the documented flow

### Templates
- Improvements to existing artifact templates
- Template refinements that improve AI accuracy or evidence quality

### Validators
- Validator improvements that:
  - Reduce false positives without weakening enforcement
  - Improve determinism of gate evaluation
  - Strengthen evidence quality requirements
- New validators that fit the artifact promotion model

### Examples
- Anonymized examples demonstrating correct usage
- End-to-end operation flow walkthroughs
- Validator PASS/FAIL examples showing specific gate behavior

---

## What You Should NOT Contribute

The following will be rejected:

- Employer-specific content
- Proprietary system names, tooling references, or internal workflows
- Tool-specific implementations disguised as standards
- Large architectural rewrites without discussion
- AI prompts that generate reliability decisions rather than structure evidence
- SLO targets presented as universal standards (SLOs must be contextualized)
- Validators with reduced enforcement of the evidence standards
- Contributions that soften "validators judge, they do not help"

---

## Anonymization Requirements (MANDATORY)

All contributions must comply with **`ANONYMIZATION.md`**.

Before submitting a PR, confirm:
- No company names or internal acronyms appear
- No internal URLs, domains, or identifiers appear
- All examples use approved placeholders
- No screenshots or logs expose identifiers
- Person names in examples are clearly fictional (e.g., "Sarah Chen," "Marcus Rivera" are acceptable placeholders)

Violations may result in immediate rejection or removal.

---

## Contribution Workflow

### 1. Fork and Branch
- Fork the repository
- Create a branch from `main`
- Use a descriptive branch name:
  - `docs/…`
  - `template/…`
  - `validator/…`
  - `example/…`

### 2. Make Your Changes
- Keep changes **small and focused**
- One logical improvement per PR
- Do not bundle unrelated changes

### 3. Validate Your Contribution
Before opening a PR, ensure:
- Content aligns with existing terminology (`TERMS.md`)
- Examples follow anonymization rules
- Templates and validators align with the playbook
- No contradictions or scope creep introduced
- Hard gates in specs and validators remain aligned

### 4. Open a Pull Request
Your PR description should include:
- What problem this change solves
- Which documents are affected
- Any intentional trade-offs
- AI Usage Disclosure

PRs without a clear purpose may be closed.

---

## Review Expectations

Maintainers will review contributions for:

- Alignment with reliability governance principles
- Evidence standard enforcement (concrete, timestamped, traceable, retrievable)
- Anonymization compliance
- Appropriateness for a public, reusable kit

Maintainers may request changes or reject PRs that:
- Introduce ambiguity into SLO definitions or validation criteria
- Weaken evidence requirements
- Embed organization-specific assumptions

---

## Style & Formatting

- Use Markdown
- Prefer bullet points over long prose
- Keep language precise and unambiguous
- Avoid marketing language
- Favor enforceable rules over guidance where possible
- All thresholds must be numeric — not "high," "elevated," or "significant"
- Durations must be clock time (minutes, hours) — not "brief" or "adequate"

---

## AI Usage in Contributions

You may use AI tools to assist in drafting content, provided that:

- You review all AI-generated output
- You ensure anonymization compliance
- You do not paste proprietary material into AI tools
- You take responsibility for the final content

---

## Reporting Issues

If you find:
- Anonymization violations
- Ambiguities in templates or validators
- Inconsistencies across documents
- Hard gate misalignment between specs and validators

Please open an issue with a clear description and reference to the affected file(s).

---

## Final Note

This repository prioritizes **reliability clarity, evidence standards, and reuse** over speed or novelty.

If you are unsure whether a contribution fits, open an issue first and start a discussion.
