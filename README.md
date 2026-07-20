# PBMS Software Requirements Specification

This directory contains the code-aligned Software Requirements Specification for the Parking Building Management System.

## Current Baseline

| Item | Value |
|---|---|
| Version | 1.4 |
| Status | REVIEW |
| Change request | CR-GEN-003 |
| Baseline date | 2026-07-20 |
| Frontend source | parking-system-web |
| Backend source | parking-system-api |

## Documents

| Document | Purpose |
|---|---|
| PBMS_SRS_Document.md | Normative system, functional, non-functional, data, interface, risk, and traceability specification. |
| PBMS_Feature_Actor_Based.md | Companion actor-to-feature mapping and authorization-coverage notes. |
| Diagrams/PBMS_Business_Analysis.md | Activity swimlanes, logical ERDs, business-rule catalogue, and diagram traceability. |
| Diagrams/PBMS_Business_Analysis_Simplified.md | Review-friendly process/ERD/rule summary aligned with the detailed analysis. |
| PBMS_Remediation_Decision_Report.md | Team handover for remediation, removal, non-exposure, ownership, migration, and exit criteria. |

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

CR-GEN-003 defines the target release boundary:

- direct-JWT login remains current and login OTP/MFA is not exposed;
- password recovery must be completed with a recovery-specific OTP/token flow;
- Grace Period remains and must run in the active PricingEngine;
- every non-public endpoint requires server-side role and ownership enforcement;
- committed payable calculations require exactly one correlated calculation log;
- Refund, Monthly Subscription, incident file upload, and automatic API retry are removed from the project;
- user Notification and Shift Report/shift handover are excluded from the current release; retained backend code must be non-exposed and non-running.

## Known Review Priorities

- Complete server-side authorization for operational controllers.
- Externalize and rotate committed secrets before production.
- Remove the visible login-OTP branch and complete password recovery.
- Align frontend booking constants with backend validation.
- Apply Grace Period in the active PricingEngine and reconcile duplicate fee services.
- Complete RBAC/ownership enforcement and committed pricing logging.
- Execute reviewed Refund and Monthly Subscription removal migrations.
- Remove incident file upload UI, unused retry configuration, and current Notification/Shift Report exposure.

The detailed evidence, requirement status, open questions, and acceptance strategy are maintained in PBMS_SRS_Document.md.
