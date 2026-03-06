# ANONYMIZATION

This document defines the **mandatory anonymization rules** for all content in the **aieos-reliability-resilience-kit** repository.

The goal is to ensure this repository remains:
- Publicly shareable
- Employer-neutral
- Tool-agnostic
- Safe for reuse and contribution
- Suitable for AI-assisted generation and validation

These rules apply to **all files**, including examples, templates, validators, diagrams, and documentation.

---

## Core Rule

> **No content may reference a real employer, internal system, or proprietary environment.**

If there is any doubt, **anonymize**.

---

## Prohibited Content (Must NOT Appear)

### Employer & Organization Identifiers
- Company names (current or former)
- Business unit names
- Internal team names
- Employer-specific acronyms
- Internal program or initiative names

### Internal Systems & Tooling
- Internal platform names
- Proprietary service names
- Internal dashboards or portals
- Internal CI/CD system names
- Internal ticketing or change-management systems
- Internal observability or monitoring system names
- Internal incident management system names
- Internal on-call system names

### Identifiers & Metadata
- Internal URLs or domains
- IP addresses (internal or external)
- Email addresses tied to real organizations
- Real usernames or personal names
- Ticket IDs or prefixes tied to a real system
- Incident numbers from a real organization
- Internal SLO identifiers or policy reference numbers
- Cluster or environment identifiers tied to a real org

### Configuration & Secrets
- API keys or tokens (even redacted)
- Certificates
- Credentials
- Environment-specific secrets
- Real account, subscription, or tenant IDs

### Logs, Screenshots, and Output
- Screenshots containing identifiers
- Logs with timestamps, IDs, or hostnames tied to a real environment
- Command output revealing environment details
- Metric names tied to real internal service names

---

## Required Anonymization Patterns

When examples require realism, use **explicit placeholders**.

### Person Names
Use clearly fictional names consistently throughout examples. The same fictional names may be reused across examples to demonstrate role continuity:
- Reliability owners: e.g., "Marcus Rivera"
- Engineering managers: e.g., "David Park"
- On-call engineers: e.g., "Sarah Chen"

Do not use real person names, real initials, or names that might identify a real individual.

### Organization and Project Names
Use clearly fictional project/organization names:
- `TaskFlow` — the example SaaS product used throughout this repository's worked examples
- `notification-service` — the example service used in worked examples

Do not use:
- Real company names
- Real product names
- Internal initiative names

### Approved Placeholder Conventions

Use ALL CAPS placeholders where specific values are not yet known:

- `ORG_NAME`
- `TEAM_NAME`
- `SERVICE_NAME`
- `SYSTEM_NAME`
- `REPO_NAME`
- `CLUSTER_NAME`
- `ENV_NAME`

### Domains & URLs
Use:
- `example.com`
- `api.example.com`
- `app.example.com`
- `oncall.example.com` (for on-call system references in examples)

Never use:
- Real company domains
- Personal domains

### Monitoring and Tooling References
Examples may reference generic monitoring systems by category:
- "Prometheus" — an accepted open-source tool reference
- "Datadog" — an accepted commercial reference (tool-agnostic templates use `{monitoring system}` placeholder)

When referencing commercial tools in examples, do so to illustrate the pattern — not to endorse a specific tool. Templates must use generic placeholders (not tool names).

### Metric Names in Examples
Examples use clearly fictional but plausible metric names that follow the `{service}_{behavior}_{dimension}` convention:
- `notification_service_request_errors_total` — acceptable example metric name
- `notification_delivery_rate_1h` — acceptable example metric name

Do not use real metric names from production monitoring systems.

### Environments
Use generic names only:
- `dev`
- `staging`
- `preprod`
- `prod`
- `production`

Avoid:
- Internally meaningful environment names
- Region- or datacenter-specific labels

### Incident and Artifact IDs
Examples use the kit's prescribed ID format:
- `IR-NOTIF-001` — acceptable example IR ID
- `SRP-NOTIF-001` — acceptable example SRP ID
- `TASK-4821` — acceptable example tracking reference (generic task system)

Do not use:
- Real incident numbers
- Real ticket IDs with organization-specific prefixes

---

## Evidence Reference Paths

The Incident Record requires retrievable evidence locations. In examples, use fictional but plausible paths:
- `incidents/IR-NOTIF-001/restoration-metrics.png` — acceptable
- Internal URLs tied to a real organization — prohibited

---

## Checklist for Contributors

Before submitting a PR, verify:

- [ ] No company names, internal team names, or employer-specific acronyms appear
- [ ] No internal URLs, domains, or ticketing identifiers appear
- [ ] All person names are clearly fictional
- [ ] All system and service names are either fictional (TaskFlow, notification-service) or clearly generic
- [ ] Metric names in examples follow the fictional naming convention and do not expose real service names
- [ ] Log excerpts and command output do not expose real environment details
- [ ] Templates use generic placeholders, not tool-specific names
- [ ] On-call references use fictional channels and endpoints
