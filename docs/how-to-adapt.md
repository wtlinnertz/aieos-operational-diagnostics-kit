# How to Adapt This Kit to Your Organization

This guide covers the decisions you need to make to adapt the Operational Diagnostics Kit to your organization's tooling, terminology, and process preferences. The governance rules (hard gates, four-file system, freeze protocols) are not configurable. They are the structural invariants that make the kit trustworthy. everything else is configurable.

---

## Decision 1: Severity Definitions

The kit uses SEV1–SEV4. ODK engagement is required for SEV1 and SEV2.

**Adapt if:** Your organization uses a different severity classification scheme (P0–P4, Critical/Major/Minor, etc.).

**How to adapt:** Update `docs/principles/diagnostic-principles.md §4` with your organization's severity levels and definitions. The DCR spec requires severity from an enumerated list, update the spec to reflect your enumerated values. Update the DCR template and validator to match.

**Do not remove:** At least two severity levels where ODK engagement is required and at least one where it is optional. The distinction between "always do this" and "judgment call" is important.


## Decision 2: Timing Standards

The playbook defines targets: DCR within 2 hours, INR within 24 hours, PMR within 5 business days, RB within 10 business days.

**Adapt if:** Your organization's incident resolution cycle is faster or slower, or if you have SLA commitments that require different timing.

**How to adapt:** Update `docs/principles/diagnostic-principles.md §5` with your organization's timing standards. These are targets, not hard gates: the validator does not check timestamps against these targets. Update the playbook.


## Decision 3: Monitoring System References

The example artifacts reference specific monitoring systems (Datadog, PagerDuty). Your monitoring stack will differ.

**Adapt if:** Always. The kit is tool-agnostic in its governance files. the examples use specific tool names for realism.

**How to adapt:** Replace tool names in the `docs/principles/diagnostic-principles.md` and in the intake form with your organization's actual monitoring systems. Update the example artifacts to reflect your systems. Do not add monitoring system names to the specs, templates, or validators, keep those files tool-agnostic.


## Decision 4: On-Call and Escalation Routing

The escalation conditions in runbooks reference "escalation target" generically.

**Adapt if:** Your organization has specific on-call rotations, escalation channels, or paging systems.

**How to adapt:** Add escalation routing guidance to `docs/principles/diagnostic-principles.md` or create a supplementary bindings document that maps generic roles (incident commander, service owner, platform on-call) to actual channels and contacts. Do not hardcode contact information in the templates, contact information changes and should be in a living document, not a frozen template.


## Decision 5: Cross-Kit Integration

The playbook and PMR template describe cross-kit escalation to EEK, PIK, and RRK.

**Adapt if:** Your organization uses different upstream/downstream kits, or has not yet adopted adjacent AIEOS kits.

**How to adapt:** If EEK is not in use: modify the Cross-Kit Actions section in the PMR template to remove the EEK escalation row, and update the PMR spec accordingly. If PIK is not in use: same approach for the PIK pattern row. The RRK integration (PMR → next RHR) should be retained whenever RRK is in use, as it is the primary cross-kit feedback path.

**Minimum viable integration:** Even without other kits, the PMR §8 Cross-Kit Actions section should reference where engineering corrective actions go and where recurring pattern concerns are escalated. Adapt the language but retain the structure.


## Decision 6: SRP Integration

The DCR references SRPs from the Reliability & Resilience Kit. If RRK is not yet in use:

**Adapt if:** Your organization does not yet have frozen SRPs.

**How to adapt:** The SRP Reference in the DCR is optional, mark it "No SRP." The DCR validator does not fail if no SRP is provided. The PMR SLO impact section can be completed qualitatively if no SRP is available. Note in `docs/principles/diagnostic-principles.md` how SLO impact should be assessed without an SRP (e.g., qualitative assessment against documented availability targets).


## Decision 7: Principle File Coverage

The kit ships with one principle file: `docs/principles/diagnostic-principles.md`. This covers investigation principles, postmortem principles, runbook principles, severity definitions, and timing standards.

**Adapt if:** Your organization has separate policy for security incidents, compliance-required postmortem formats, or regulatory requirements for incident documentation.

**How to adapt:** Add a second principle file (e.g., `docs/principles/security-incident-principles.md`) for specialized incident types. Reference it in the playbook for the applicable artifact types. Do not merge everything into the existing principle file, keep it readable.


## What Not to Change

**Do not change the hard gates.** The hard gates are the quality contract. Weakening them defeats the purpose of the kit.

**Do not change the four-file system.** Spec, template, prompt, validator. One question per file. if you need additional files, add supplementary documents; do not collapse the four-file structure.

**Do not allow AI to generate the DCR.** The entry gate is human-authored by design. The human incident owner must confirm the incident context. Not an ai generating from memory or partial evidence.

**Do not allow the same AI session to generate and validate.** Self-validation bias is real. Keep generation and validation sessions separate.

**Do not edit a frozen artifact.** The amendment procedure exists for non-material corrections. Material changes require a new version.


## Organization-Specific Bindings (Optional)

Consider creating a `docs/bindings/` directory for organization-specific configurations that do not belong in the governance files:

- `docs/bindings/severity-mapping.md`. maps your internal severity labels to SEV1–SEV4
- `docs/bindings/escalation-routing.md`. maps roles to actual contacts/channels
- `docs/bindings/monitoring-systems.md`. your actual monitoring tool names and metric naming conventions

Bindings are not governed files. They don't have validators and don't follow the four-file system. they're reference documents for operators.
