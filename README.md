# PBMS Software Requirements Specification

This directory contains the code-aligned Software Requirements Specification for the Parking Building Management System.

## Current Baseline

| Item | Value |
|---|---|
| Version | 1.2 |
| Status | REVIEW |
| Change request | CR-GEN-001 |
| Baseline date | 2026-07-18 |
| Frontend source | parking-system-web |
| Backend source | parking-system-api |

## Documents

| Document | Purpose |
|---|---|
| PBMS_SRS_Document.md | Normative system, functional, non-functional, data, interface, risk, and traceability specification. |
| PBMS_Feature_Actor_Based.md | Companion actor-to-feature mapping and authorization-coverage notes. |
| Diagrams/PBMS_Business_Analysis.md | Activity swimlanes, logical ERDs, business-rule catalogue, and diagram traceability. |

## Maintenance Workflow

This baseline follows Option B of CR-GEN-001: the two root Markdown documents are maintained directly. There is no active sections directory or build-srs script in this folder.

For each future change:

1. Compare both parking-system-web and parking-system-api behavior.
2. Create or reference a change request.
3. Preserve existing requirement IDs; mark removed behavior DEPRECATED instead of reusing IDs.
4. Update revision history, requirements, business rules, traceability, risks, and validation together.
5. Keep changed content in REVIEW until an authorized human approves it.
6. Validate headings, duplicate IDs, links, and Markdown formatting.

## Scope Note

Monthly Subscription is DEPRECATED in version 1.2. Its active frontend and backend flows were removed. Remaining entities, fields, tables, and SubscriptionPriceConfig are documented only as legacy or dormant compatibility artifacts.

## Known Review Priorities

- Complete server-side authorization for operational controllers.
- Externalize and rotate committed secrets before production.
- Resolve the disconnected login-OTP branch.
- Align frontend booking constants with backend validation.
- Decide whether the active PricingEngine applies or retires GracePeriodRule.

The detailed evidence, requirement status, open questions, and acceptance strategy are maintained in PBMS_SRS_Document.md.
