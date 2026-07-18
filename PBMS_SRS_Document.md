---
document_id: PBMS-SRS
title: Parking Building Management System - Software Requirements Specification
version: 1.2
status: REVIEW
change_request: CR-GEN-001
baseline_date: 2026-07-18
last_updated: 2026-07-18
source_baseline:
  - parking-system-web
  - parking-system-api
---

# Parking Building Management System

## Software Requirements Specification

Version 1.2 - REVIEW

This revision is a code-aligned specification. It describes behavior observable in the current frontend and backend. It does not treat repository documentation, dormant database objects, or unreachable UI branches as implemented features.

## 1. Document Control

### 1.1 Revision History

| Version | Date | Status | Change Request | Description |
|---|---|---|---|---|
| 1.0 | 2026-06-15 | REVIEW | Historical | Initial consolidated SRS. |
| 1.1 | 2026-07-12 | REVIEW | Historical | Added booking, pricing, payment, and operational details. |
| 1.2 | 2026-07-18 | REVIEW | CR-GEN-001 | Reconciled the SRS with parking-system-web and parking-system-api; deprecated Monthly Subscription; corrected architecture, booking capacity, pricing, camera/OCR, payment, revenue, and implementation gaps. |

### 1.2 Approval Record

| Role | Name | Decision | Date |
|---|---|---|---|
| Product owner | TBD | Pending | TBD |
| Technical lead | TBD | Pending | TBD |
| QA lead | TBD | Pending | TBD |

No requirement in this revision is APPROVED. Changed and newly discovered behavior remains REVIEW until human approval.

### 1.3 Change Scope

CR-GEN-001 authorizes direct updates to the existing PBMS-SRS root documents. It also authorizes Monthly Subscription to be moved out of active scope and retained as deprecated history because its controller, service, repository, DTO, test, and frontend flows were removed from the current codebase.

## 2. Introduction

### 2.1 Purpose

This SRS defines the current functional and non-functional behavior of the Parking Building Management System, abbreviated PBMS. It is the review baseline for development, testing, acceptance, and future change control.

### 2.2 Product Scope

PBMS supports:

- driver account registration, login, vehicle management, bookings, session history, and payments;
- gate staff check-in, checkout, card handling, incidents, blacklists, and operational monitoring;
- manager configuration of facilities, pricing, operations, and reports;
- administrator management of users, configuration, audit logs, and selected operational records;
- camera capture and external license plate recognition;
- cash and VNPay payment flows;
- dynamic revenue reporting from paid payments.

Monthly Subscription is not an active product capability in this baseline.

### 2.3 Intended Audience

The intended audience is the PBMS product owner, analysts, developers, testers, operators, and maintainers.

### 2.4 Definitions and Acronyms

| Term | Meaning |
|---|---|
| Booking | A planned parking interval created before entry. |
| Walk-in | A parking session created without a booking. |
| Buffer time | Configured separation used by service-level slot overlap checks. |
| Zone booking limit rate | Percentage of zone capacity available to bookings. |
| OCR | Optical character recognition for license plates. |
| IPN | Server-to-server payment notification. |
| UTC+7 | Vietnam business reporting time zone used by revenue aggregation. |
| Dormant | Present in code or schema but not used by the active product flow. |
| Deprecated | Retained for traceability but excluded from active scope. |

### 2.5 Source Registry

| Source ID | Source | Role |
|---|---|---|
| REF-GEN-001 | parking-system-web source, package metadata, routes, feature modules, and API clients | Frontend implementation baseline |
| REF-GEN-002 | parking-system-api API and Application source | API and business logic baseline |
| REF-GEN-003 | parking-system-api Domain, Infrastructure, EF Core configuration, and migrations | Data and integration baseline |
| REF-GEN-004 | User authorization for Option B on 2026-07-18 | Change-control authority for direct document update and deprecation package |

### 2.6 Requirement Conventions

The word shall denotes a mandatory requirement. Status REVIEW means the behavior is implemented or inferred from code but still requires stakeholder review. DEPRECATED requirements are historical and are not acceptance targets for the current release.

## 3. Overall Description

### 3.1 Product Perspective

PBMS is a web-based system consisting of a Next.js single-page experience and an ASP.NET Core REST API backed by PostgreSQL. The backend follows Clean Architecture boundaries: Domain, Application, Infrastructure, and API.

### 3.2 User Classes

| Actor | Current responsibility |
|---|---|
| Guest | Register, verify registration email, sign in with password or Google. |
| Driver | Manage own vehicles, bookings, sessions, payments, history, and reports. |
| Staff | Operate gate entry and exit, cards, incidents, blacklists, and monitoring. |
| Manager | Manage facilities, pricing, operational records, and reports. |
| Admin | Manage accounts, facilities, configuration, incidents, blacklists, and audit logs. |
| VNPay | Receive payment requests and return signed payment results. |
| Plate Recognizer | Return OCR candidates for a captured plate image. |
| Background workers | Expire bookings and pricing policies and generate overtime warnings. |

### 3.3 Operating Environment

| Component | Current technology |
|---|---|
| Web client | Next.js 15.1.6, React 18.3.1, TypeScript 5.7.2, Tailwind CSS 3.4.17 |
| API | .NET 10, ASP.NET Core controllers |
| Persistence | Entity Framework Core 10 with Npgsql and PostgreSQL |
| Authentication | JWT bearer token, BCrypt password hashes, Google identity integration |
| External services | SMTP email, VNPay, Plate Recognizer |

### 3.4 Constraints

- Camera capture depends on browser media APIs and therefore requires a supported browser, permission, and a secure context except localhost.
- Business timestamps are persisted by the backend; revenue dates are grouped in UTC+7.
- VNPay and OCR require valid external credentials and network connectivity.
- Current API authorization coverage is incomplete. Frontend route guards are not a security boundary.
- Current repositories contain environment secrets in configuration files; production deployment shall externalize and rotate them before release.

### 3.5 Assumptions and Dependencies

- Buildings, floors, zones, vehicle types, slots, roles, and necessary configuration are seeded or managed before operations begin.
- Plate OCR may fail or return an incorrect candidate; staff can use manual entry or correction.
- Online payment completion depends on a valid signed VNPay callback or return.
- PostgreSQL supports the btree_gist extension used by the booking exclusion constraint.

### 3.6 Scope Boundaries

Active scope does not include forgot-password recovery, an enforced permission matrix, a completed login-MFA flow, Monthly Subscription purchase/renewal, automated physical barriers, or guaranteed plate-recognition accuracy.

## 4. System Context and Architecture

### 4.1 Logical Context

Guest or authenticated browser -> Next.js App Router UI -> typed fetch client -> ASP.NET Core controllers -> Application services -> repositories and external gateways -> PostgreSQL, VNPay, SMTP, Google, and Plate Recognizer.

### 4.2 Frontend Architecture

- App Router pages are grouped into public and role-specific dashboard paths.
- Feature modules contain components, hooks, types, and service clients.
- ProtectedRoute checks the stored authenticated user and permitted role before rendering a dashboard route.
- The API wrapper reads the JWT from browser localStorage, sends it as a bearer token, applies a ten-second timeout, and clears the local session on HTTP 401.
- A retry count is declared in configuration but is not implemented by the current wrapper.

### 4.3 Backend Architecture

- PBMS.Domain contains entities, enums, rules, and pricing-engine domain logic.
- PBMS.Application contains DTOs, service interfaces, use-case services, and validation.
- PBMS.Infrastructure contains EF Core repositories, migrations, persistence configuration, and external gateways.
- PBMS.API contains controllers, middleware configuration, dependency injection, Swagger, CORS, and hosted workers.

### 4.4 Background Processing

| Worker | Current interval | Behavior |
|---|---:|---|
| Expired booking cleanup | 5 minutes | Expires unpaid pending bookings and marks confirmed bookings as no-show after grace. |
| Overtime warning | 1 minute | Checks sessions for overtime warning conditions. |
| Expired pricing policy cleanup | 12 hours | Deactivates expired active pricing policies. |

### 4.5 Deployment View

The current development topology is a browser client, a separately hosted API, PostgreSQL, and external services. CORS and HTTPS behavior are configurable. Swagger is currently enabled by API startup and should be restricted appropriately for production.

## 5. Actors and Access Model

### 5.1 Frontend Route Model

| Area | Driver | Staff | Manager | Admin |
|---|---:|---:|---:|---:|
| Own vehicles, bookings, sessions, payments | Yes | No | Operational views only | No |
| Gate check-in and checkout | No | Yes | No dedicated gate route | No |
| Cards, incidents, blacklists | Limited own context | Yes | Yes | Yes for selected modules |
| Facility management | No | Monitoring only | Yes | Yes |
| Pricing management | No | No | Yes | Configuration only |
| User and audit management | No | No | No | Yes |

### 5.2 Backend Enforcement Reality

The API currently applies authorization attributes to account, audit-log, pricing-policy, pricing-engine, penalty-configuration, and subscription-price-configuration controllers. Many other controllers do not currently declare authorization attributes. Therefore:

- the table above describes intended UI access and available pages;
- it shall not be interpreted as proof of server-side enforcement;
- RISK-SEC-001 requires endpoint-by-endpoint authorization remediation before production acceptance.

### 5.3 Permission Data

Role, Permission, and RolePermission entities exist. The current API primarily uses role checks where authorization is present and does not consistently enforce the stored permission matrix.

## 6. Functional Feature Catalogue

| Feature ID | Feature | Primary actors | Status |
|---|---|---|---|
| F-STR-001 | Parking structure and capacity management | Manager, Admin | REVIEW |
| F-DRV-001 | Driver account, profile, and vehicles | Guest, Driver | REVIEW |
| F-BOOK-001 | Booking lifecycle and capacity control | Driver, Staff, Manager | REVIEW |
| F-OPS-001 | Entry, OCR, and check-in | Staff | REVIEW |
| F-ALLOC-001 | Zone and slot allocation | Staff, Manager | REVIEW |
| F-SESSION-001 | Active parking session management | Driver, Staff, Manager | REVIEW |
| F-OPS-002 | Checkout, unpaid exit, and lost-card handling | Staff | REVIEW |
| F-PAY-001 | Cash and VNPay payments and refunds | Driver, Staff, Manager | REVIEW |
| F-PRICE-001 | Pricing policy management | Manager | REVIEW |
| F-PRICE-002 | Fee and penalty calculation | System, Manager | REVIEW |
| F-INC-002 | Incident, card, and blacklist management | Staff, Manager, Admin | REVIEW |
| F-MON-001 | Monitoring, revenue, and audit | Staff, Manager, Admin | REVIEW |
| F-MONTH-001 | Monthly Subscription | Historical | DEPRECATED |

## 7. Use Cases

### 7.1 UC-DRV-001 Register and Authenticate

- Primary actor: Guest
- Preconditions: The guest can access email or a valid Google identity.
- Main flow: Request registration OTP; verify OTP; submit registration token and profile; system creates a Driver account; user signs in and receives a JWT.
- Alternate flow: Existing Google-linked account signs in directly; a new Google account completes email verification first.
- Postcondition: An authenticated session is stored by the web client.
- Notes: Password login currently issues a JWT directly. The separate login-OTP endpoint is not connected to the normal login service flow.

### 7.2 UC-STR-001 Manage Parking Structure

- Primary actors: Manager, Admin
- Main flow: Manage buildings, floors, zones, vehicle types, and slots; configure zone booking rate and system capacity parameters.
- Postcondition: Active structure data becomes available to booking and gate operations.

### 7.3 UC-BOOK-001 Create and Pay for a Booking

- Primary actor: Driver
- Preconditions: Authenticated driver, owned vehicle, eligible building and type, and no blacklist or conflicting booking.
- Main flow: Select vehicle, building, times, and optionally a car slot; system validates lead time, duration, capacity, zone limit, and overlap; system calculates the full planned fee as deposit; driver completes VNPay payment; booking becomes Confirmed.
- Alternate flow: Unpaid booking remains Pending until its payment deadline and is later Expired.
- Postcondition: A Pending or Confirmed booking and associated payment are stored.

### 7.4 UC-ALLOC-001 Allocate Zone or Slot

- Primary actor: Staff
- Main flow: System validates the facility hierarchy and vehicle type; car sessions may receive a concrete slot; non-car sessions use zone capacity without a required slot.
- Postcondition: The session is assigned without conflicting active use.

### 7.5 UC-OPS-001 Capture Plate and Check In

- Primary actor: Staff
- Main flow: Capture an image or enter a plate; OCR returns a candidate; system normalizes the plate; staff validates vehicle, booking, card, blacklist, structure, and capacity; system creates a parking session and marks relevant resources active.
- Alternate flow: Staff corrects the OCR result or proceeds using manual input.
- Postcondition: An active session exists, image-in is retained when provided, and a linked booking becomes CheckedIn.

### 7.6 UC-SESSION-001 View or Extend a Parking Session

- Primary actors: Driver, Staff
- Main flow: View session details and pricing estimate; request a later checkout; system limits the extension if a later slot booking conflicts and creates additional payment when required.
- Postcondition: The planned end time and payment state reflect the accepted extension.

### 7.7 UC-OPS-002 Check Out a Vehicle

- Primary actor: Staff
- Main flow: Capture exit evidence; verify plate and card; calculate fee; create or settle payment; complete checkout; release slot and card; resolve applicable open incidents.
- Alternate flow: Mark unpaid checkout, lost card, mismatch incident, replacement card, or rollback when permitted.
- Postcondition: Session is completed or marked UNPAID and resources are released consistently.

### 7.8 UC-PAY-001 Complete Payment or Refund

- Primary actors: Driver, Staff, Manager
- Main flow: Create CASH or ONLINE_BANKING payment; invalidate older pending payment for the same source; cash settles synchronously or VNPay settles after a verified callback; eligible canceled booking moves to REFUND_PENDING; authorized operator records REFUNDED.

### 7.9 UC-PRICE-001 Manage Pricing Policy

- Primary actor: Manager
- Main flow: Define effective dates, applicable building and vehicle type, base/increment/cap rules, and priority; system prevents overlap at the same priority and cleans up expired active policies.

### 7.10 UC-PRICE-002 Calculate Parking Charge

- Primary actor: System
- Main flow: Select policy according to segmented-pricing configuration; apply base and increment blocks, partial-block threshold, daily cap, and unresolved incident penalties; return a detailed calculation.

### 7.11 UC-INC-002 Handle Incident and Blacklist

- Primary actors: Staff, Manager, Admin
- Main flow: Record or update incident; apply configured penalty where applicable; block a vehicle or card; resolve the incident and update related resources.

### 7.12 UC-MON-001 Monitor and Report

- Primary actors: Staff, Manager, Admin
- Main flow: View active operations, payment-derived revenue, and audit logs through available pages and APIs.

### 7.13 UC-MONTH-001 Monthly Subscription

- Status: DEPRECATED
- This historical use case is not an acceptance target. The active frontend and backend no longer expose purchase, renewal, subscription payment, or subscription-based session processing.

## 8. Functional Requirements

### 8.1 Identity and Driver Data

#### FR-ACC-001 Registration Email Verification

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: The system shall require a six-digit email OTP before creating a password-based account.
- Rationale: Prove control of the registration email.
- Acceptance Criteria: Given a valid OTP issued within five minutes, when the guest verifies it, then the system returns a registration token valid for ten minutes; given five failed verifications, then verification is locked for fifteen minutes; a resend request within sixty seconds is rejected.
- Dependencies: SMTP service.

#### FR-ACC-002 Password and Google Authentication

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: The system shall authenticate an existing account by BCrypt-verified password or supported Google identity and issue a JWT on success.
- Acceptance Criteria: Given valid credentials for an active account, when login completes, then the response includes the authenticated account and bearer token; invalid credentials do not create a session.
- Notes: Normal password login does not currently require a second OTP.

#### FR-ACC-003 Client Session Handling

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001
- Requirement: The web client shall attach the stored JWT to authenticated API requests and clear the local authenticated session after an HTTP 401 response.
- Acceptance Criteria: Given a stored token, when the client calls the API, then an Authorization bearer header is sent; when the response is 401, then stored authentication data is removed.

#### FR-DRV-001 Vehicle Management

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002, REF-GEN-003
- Requirement: A Driver shall be able to create, view, update, and archive eligible vehicles associated with the Driver account.
- Acceptance Criteria: A license plate is normalized by removing whitespace, hyphens, and dots and converting letters to uppercase; an ownerless walk-in vehicle may be claimed; a vehicle with an active session cannot be archived.

### 8.2 Parking Structure

#### FR-STR-001 Facility Hierarchy Management

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: Authorized facility operators shall manage the Building, Floor, Zone, and ParkingSlot hierarchy and its operational status.
- Acceptance Criteria: A slot belongs to one zone, a zone to one floor, and a floor to one building; inactive or incompatible hierarchy nodes are rejected by booking or check-in validation as implemented.

#### FR-STR-002 Zone Booking Capacity

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-002, REF-GEN-003
- Requirement: Each zone shall define a BookingLimitRate from 1 through 100 percent, with a default of 80 percent.
- Acceptance Criteria: Values outside 1 through 100 are rejected; booking load is compared with floor(zone capacity multiplied by BookingLimitRate divided by 100).

#### FR-STR-003 Slot State Visibility

- Priority: Should
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: The system shall expose slot operational state and time-dependent booking reservation information to supported monitoring and allocation views.
- Acceptance Criteria: Available, Occupied, Reserved, Blocked, and Maintenance conditions can be represented; a future booking can make a slot unavailable for the requested interval without requiring permanent Reserved state.

### 8.3 Booking

#### FR-BOOK-001 Create Booking

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: A Driver shall be able to create a booking for an owned eligible vehicle at least fifteen minutes before planned check-in and for a minimum duration of four hours.
- Acceptance Criteria: Invalid ownership, blacklist, inactive structure, unsupported vehicle type, overlap, or insufficient capacity returns a validation failure; a valid request creates a Pending booking with a payment deadline fifteen minutes after creation and a check-in grace endpoint thirty minutes after planned check-in.
- Dependencies: FR-PRICE-001, FR-PAY-001.

#### FR-BOOK-002 Optional Car Slot Selection

- Priority: Should
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002, REF-GEN-003
- Requirement: The system shall allow an explicit SlotId only for a car booking and shall reject an incompatible, unavailable, or overlapping slot.
- Acceptance Criteria: The slot belongs to the selected building and vehicle type and is not Blocked, Maintenance, or Reserved; service validation includes configured buffer time; PostgreSQL prevents overlapping Pending or Confirmed intervals for the same non-null slot.
- Notes: The database exclusion interval is half-open and does not include the configurable service buffer.

#### FR-BOOK-003 Booking Capacity Calculation

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-002, REF-GEN-003
- Requirement: The system shall reserve vehicle-type capacity after subtracting the configured buffer ratio and shall enforce the selected zone booking limit.
- Acceptance Criteria: Near-term bookings within WALKIN_STAY_THRESHOLD_HOURS, default two, count active walk-ins plus overlapping active bookings; farther bookings count overlapping active bookings; the request is rejected when adding it exceeds the allowed zone load.

#### FR-BOOK-004 Booking Deposit

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-002
- Requirement: The system shall use the pricing engine estimate for the entire planned booking interval as the booking deposit amount.
- Acceptance Criteria: A valid booking creates or associates a payment for the calculated total; a successful booking payment changes Pending to Confirmed.

#### FR-BOOK-005 Modify, Cancel, and Extend Booking

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: The system shall support valid booking updates, cancellation, and extension according to current state and availability.
- Acceptance Criteria: Only Pending bookings are generally editable; Pending or Confirmed bookings may be canceled; a Confirmed booking canceled at least sixty minutes before check-in moves its paid deposit to REFUND_PENDING, while later cancellation forfeits that refund path; extension is allowed for Pending, Confirmed, or CheckedIn and creates additional payment when required.

#### FR-BOOK-006 Booking Expiration and No-show

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-002
- Requirement: The booking cleanup worker shall expire unpaid Pending bookings after their payment deadline and mark Confirmed bookings NoShow after their check-in grace endpoint.
- Acceptance Criteria: The cleanup executes approximately every five minutes and transitions only eligible records.

### 8.4 Entry, Allocation, and Session

#### FR-OPS-001 Camera Capture and OCR

- Priority: Should
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: The gate workflow shall support browser camera capture, Base64 image submission, external plate recognition, and manual correction.
- Acceptance Criteria: The backend submits the image to Plate Recognizer with Vietnam region context, selects the highest-confidence result, normalizes the result, and returns an actionable response; OCR failure does not remove manual entry capability.

#### FR-OPS-002 Check-in Validation

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: Before check-in, the system shall validate the plate, vehicle or booking, card, blacklists, active-session duplicates, building, vehicle type, floor, zone, slot, and capacity.
- Acceptance Criteria: A conflicting active vehicle, card, or slot prevents creation; successful check-in creates an active parking session, records ImageIn when supplied, activates the card, occupies the assigned car slot, and changes a linked booking to CheckedIn.

#### FR-OPS-003 Plate Normalization

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-002, REF-GEN-003
- Requirement: The system shall compare and persist vehicle plates using the normalized uppercase value with spaces, hyphens, and dots removed.
- Acceptance Criteria: Equivalent formatted variants resolve to the same normalized plate and cannot create duplicate normalized vehicle identities.

#### FR-ALLOC-001 Session Allocation

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-002
- Requirement: The system shall allocate a compatible active zone and, for cars when required, a compatible non-conflicting slot.
- Acceptance Criteria: The resulting allocation respects building and vehicle-type compatibility and does not exceed effective capacity.

#### FR-SESSION-001 Session Query and Update

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: Supported actors shall be able to retrieve active and historical parking sessions and update only fields permitted by the session workflow.
- Acceptance Criteria: Driver views are scoped to the account in the UI service flow; operational views expose identifiers, timing, allocation, image, and payment state needed for gate work.

#### FR-SESSION-002 Session Extension

- Priority: Should
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: The system shall calculate and apply an eligible extension and shall cap a slot-bound extension before a conflicting later booking including configured buffer time.
- Acceptance Criteria: A positive additional fee creates or updates a pending payment and returns a VNPay URL when online settlement is required.

#### FR-OPS-004 Checkout

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: The system shall record checkout evidence, calculate the current fee, settle the selected payment method, complete the session, and release associated resources.
- Acceptance Criteria: ImageOut and detected plate are retained when provided; mismatched plate or card conditions can be routed to incident handling; successful completion releases the slot and card and closes applicable open incidents.

#### FR-OPS-005 Unpaid Exit and Lost Card

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: Staff shall be able to complete the supported unpaid-exit, lost-card, replacement-card, and rollback workflows.
- Acceptance Criteria: An unpaid exit sets the unpaid state, releases operational resources, creates an incident and blacklist entry; lost-card handling marks the card Lost and creates related controls; rollback is rejected where a successful payment makes reversal unsafe.

### 8.5 Pricing and Payment

#### FR-PRICE-001 Pricing Policy Management

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002, REF-GEN-003
- Requirement: An authorized Manager shall manage effective pricing policies by building, vehicle type, effective interval, pricing rules, and integer Priority.
- Acceptance Criteria: Priority defaults to zero; policies at the same priority cannot have overlapping effective ranges for the same applicable context; expired active policies are deactivated by scheduled cleanup.

#### FR-PRICE-002 Policy Selection

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-002
- Requirement: The pricing service shall select policies according to APPLY_SEGMENTED_PRICING.
- Acceptance Criteria: When the key is absent or false, one active policy at check-in governs selection; when true, each pricing segment selects the applicable policy by highest Priority, then latest EffectiveStart, then identifier.

#### FR-PRICE-003 Fee Calculation

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-002
- Requirement: The active pricing engine shall calculate base blocks, increment blocks, partial-block threshold charges, calendar-day caps in UTC+7, and active incident penalties.
- Acceptance Criteria: The base block uses the policy selected at check-in; each later increment uses the policy at block start; a partial increment is charged only when its percentage meets the configured threshold; the checkout-selected policy supplies the daily cap; Open or Processing incident fees are added.
- Notes: GracePeriodRule entities exist but the active pricing engine does not apply them. The older FeeCalculationService remains registered but is not called by current booking, payment, session, or pricing-engine flows.

#### FR-PAY-001 Create and Complete Payment

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: The system shall support CASH and ONLINE_BANKING payments for an active booking or parking session source.
- Acceptance Criteria: Creating a new payment marks older pending payments for the same source Failed; cash is marked Paid synchronously after configured rounding; online payment is Pending and returns a VNPay URL; a verified successful VNPay result marks it Paid and triggers the associated booking or extension transition.

#### FR-PAY-002 VNPay Callback and Browser Return

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: The API shall validate VNPay response signatures and expose both an IPN response flow and a browser return flow.
- Acceptance Criteria: A valid success result within the allowed payment window is applied once; invalid, failed, or expired results do not produce a successful payment; browser return redirects to the configured frontend payment-result page.

#### FR-PAY-003 Refund State Transition

- Priority: Should
- Status: REVIEW
- Source: REF-GEN-002
- Requirement: The refund operation shall accept only a payment in REFUND_PENDING and record it as REFUNDED.
- Acceptance Criteria: Other payment states are rejected and no external automatic refund is claimed by this requirement.

### 8.6 Incidents, Cards, Blacklists, and Reporting

#### FR-INC-001 Incident Management

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: Supported operators shall create, retrieve, update, resolve, and soft-delete incident records and manage incident types.
- Acceptance Criteria: An incident can retain session, vehicle, card, evidence, status, and fee context; Open or Processing incidents can contribute penalties to active pricing.

#### FR-CARD-001 Card Management

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: Supported operators shall manage parking cards and their Available, Active, Lost, or blocked operational conditions.
- Acceptance Criteria: A card cannot be used by two active sessions and is released or changed by checkout and incident workflows.

#### FR-BLK-001 Blacklist Management

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: Supported operators shall manage vehicle and card blacklist controls, and booking or check-in shall reject a currently blocked subject.
- Acceptance Criteria: Plate lookup uses normalized plate comparison; unpaid exit and lost-card workflows can create blacklist entries.

#### FR-RPT-001 Dynamic Revenue Reporting

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-002
- Requirement: Revenue reports shall be calculated from current PAID payment records and grouped by Vietnam UTC+7 day, month, year, building, and vehicle type as requested.
- Acceptance Criteria: Non-paid payments are excluded; source context is resolved from a session or booking; TotalSubscriptions is zero; legacy RevenueStatistic tables are not presented as the active calculation source.

#### FR-AUD-001 Audit Log Query

- Priority: Should
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: An authorized administrator shall be able to query available audit-log records.
- Acceptance Criteria: The endpoint requires authorization and the admin UI can display returned records with supported filters or pagination.

#### FR-CFG-001 Dynamic Parking Configuration

- Priority: Should
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002, REF-GEN-003
- Requirement: Supported administration flows shall read and update dynamic parking configuration keys used by business services.
- Acceptance Criteria: BUFFER_TIME_MINUTES and WALKIN_STAY_THRESHOLD_HOURS are seeded with defaults 30 and 2; APPLY_SEGMENTED_PRICING defaults to false behavior when absent.

### 8.7 Deprecated Functional Requirement

#### FR-MONTH-001 Monthly Subscription

- Priority: Not applicable
- Status: DEPRECATED
- Source: REF-GEN-002, REF-GEN-003, REF-GEN-004
- Requirement: Historical only. The system previously specified monthly-subscription creation, renewal, payment, and session linkage.
- Rationale: Active subscription controller, application flow, repository, DTOs, tests, and frontend feature were removed. Remaining entity fields, database table, and SubscriptionPriceConfig are dormant compatibility artifacts.
- Acceptance Criteria: This item shall not be used as a current-release acceptance test. Any reactivation requires a new approved change request and end-to-end implementation.

## 9. Business Rules

### 9.1 Active Rules

| Rule ID | Rule | Status | Source |
|---|---|---|---|
| BR-ACC-001 | Registration OTP contains six digits and expires after five minutes. | REVIEW | REF-GEN-002 |
| BR-ACC-002 | OTP resend cooldown is sixty seconds. | REVIEW | REF-GEN-002 |
| BR-ACC-003 | Five failed OTP verifications cause a fifteen-minute lockout. | REVIEW | REF-GEN-002 |
| BR-ACC-004 | Registration verification token expires after ten minutes. | REVIEW | REF-GEN-002 |
| BR-BOOK-001 | Booking lead time is at least fifteen minutes. | REVIEW | REF-GEN-002 |
| BR-BOOK-002 | Booking duration is at least four hours. | REVIEW | REF-GEN-002 |
| BR-BOOK-003 | Booking payment deadline is fifteen minutes after creation. | REVIEW | REF-GEN-002 |
| BR-BOOK-004 | Confirmed booking check-in grace is thirty minutes after planned check-in. | REVIEW | REF-GEN-002 |
| BR-BOOK-005 | Only cars may select a concrete slot during booking. | REVIEW | REF-GEN-002 |
| BR-BOOK-006 | Default slot booking buffer is thirty minutes. | REVIEW | REF-GEN-003 |
| BR-BOOK-007 | Default near-term walk-in threshold is two hours. | REVIEW | REF-GEN-003 |
| BR-BOOK-008 | Default zone booking limit is eighty percent and valid values are 1 through 100. | REVIEW | REF-GEN-003 |
| BR-BOOK-009 | Effective general capacity subtracts the ceiling of the vehicle-type buffer ratio. | REVIEW | REF-GEN-002 |
| BR-BOOK-010 | The booking deposit equals the pricing estimate for the complete planned interval. | REVIEW | REF-GEN-002 |
| BR-BOOK-011 | A Confirmed cancellation at least sixty minutes before check-in is refund-pending; a later cancellation loses that refund path. | REVIEW | REF-GEN-002 |
| BR-ALLOC-001 | The same vehicle cannot have two active sessions. | REVIEW | REF-GEN-002 |
| BR-ALLOC-002 | The same card cannot serve two active sessions. | REVIEW | REF-GEN-002 |
| BR-ALLOC-003 | A concrete slot cannot serve overlapping active use. | REVIEW | REF-GEN-002, REF-GEN-003 |
| BR-FEE-001 | Non-segmented pricing is used when APPLY_SEGMENTED_PRICING is absent or false. | REVIEW | REF-GEN-002 |
| BR-FEE-002 | Segmented selection orders candidates by Priority, EffectiveStart, and identifier. | REVIEW | REF-GEN-002 |
| BR-FEE-003 | A partial increment is charged only at or above its threshold percentage. | REVIEW | REF-GEN-002 |
| BR-FEE-004 | Daily-cap calendar boundaries use UTC+7. | REVIEW | REF-GEN-002 |
| BR-FEE-005 | Open or Processing incident penalties are included in current pricing. | REVIEW | REF-GEN-002 |
| BR-PAY-001 | A newer payment invalidates older Pending payments for the same booking or session. | REVIEW | REF-GEN-002 |
| BR-PAY-002 | CASH settles synchronously and ONLINE_BANKING settles through VNPay verification. | REVIEW | REF-GEN-002 |
| BR-PAY-003 | Only REFUND_PENDING can transition through the recorded refund operation. | REVIEW | REF-GEN-002 |
| BR-PAY-004 | Revenue includes only PAID payments. | REVIEW | REF-GEN-002 |

### 9.2 Deprecated Rule Family

BR-MONTH-001 through BR-MONTH-024 are DEPRECATED as one historical family under CR-GEN-001. They remain reserved and shall not be reused. None is an active acceptance rule. Historical monthly-subscription behavior can be recovered from version control if a future change request reintroduces the capability.

### 9.3 Superseded Historical Rule IDs

Historical rule IDs not enumerated in the active table remain reserved. Their omission from the current active baseline means the previous statement was duplicated, too implementation-specific, contradicted by current code, or belonged to the deprecated subscription flow. IDs shall not be reassigned to unrelated behavior.

## 10. External Interface Requirements

### 10.1 User Interface

- Public routes provide home, login, registration, and payment result.
- Driver routes provide dashboard, booking, parking utilities, sessions, payments, vehicles, reports, history, help, and profile.
- Staff routes provide dashboard, gate flows, booking review, monitoring, incidents, incident-type reference, cards, blacklists, reports, help, and profile.
- Manager routes provide dashboard, facilities, slots, pricing, incidents, bookings, sessions, payments, cards, blacklists, vehicles, help, and profile.
- Admin routes provide user, facility, incident, blacklist, audit-log, settings, help, and profile views; an admin dashboard route also exists.

### 10.2 REST API

- The API uses controller routes under the /api prefix.
- Responses generally use a BaseResponse envelope and HTTP status codes.
- Authenticated requests use JWT bearer tokens where controller authorization is applied.
- The frontend base URL is environment-configurable. Its default /api/v1 value must be reconciled with backend routes in deployment configuration.

### 10.3 VNPay

- Outbound payment URLs are signed with HMAC-SHA512.
- The payment amount is sent in VNPay minor-unit format.
- The API validates returned signature data and separates server IPN response from browser return redirection.

### 10.4 Plate Recognizer

- The backend submits an image payload and region vn.
- The highest-confidence result is used as the initial candidate.
- The service is advisory; staff verification remains necessary.

### 10.5 Email and Google

- SMTP delivers registration OTP email.
- Google identity data supports an existing-account login or a verified new-account registration path.

## 11. Data Requirements

### 11.1 Active Core Entities

| Domain | Main entities |
|---|---|
| Identity | Role, Permission, RolePermission, Account |
| Structure | Building, Floor, Zone, ParkingSlot, VehicleType |
| Driver and operations | Vehicle, Card, Booking, ParkingSession |
| Incidents | IncidentType, Incident, Blacklist, PenaltyConfig |
| Pricing | PricingPolicy, PricingWindow, PricingRule, BasePricingRuleConfig, IncrementPricingRuleConfig, DailyCapRuleConfig, GracePeriodRuleConfig, PricingCalculationLog |
| Finance and reporting | Payment, AuditLog |
| Configuration | ParkingSystemConfig |

### 11.2 Dormant or Compatibility Data

| Object | Current state |
|---|---|
| MonthlySubscription and related foreign-key fields | Schema compatibility only; no active end-to-end service or UI flow. |
| SubscriptionPriceConfig | API and persistence remain, but there is no active Monthly Subscription consumer. |
| RevenueStatistic and RevenueStatisticPayment | Not used by current dynamic revenue calculation. |
| GracePeriodRuleConfig | Persisted and used by legacy fee logic, but not applied by the active PricingEngine. |

### 11.3 Integrity Rules

- NormalizedLicensePlate provides normalized vehicle identity.
- A PostgreSQL exclusion constraint prevents overlapping Pending or Confirmed bookings for the same non-null SlotId over half-open planned intervals.
- Soft-delete is used by multiple managed entities.
- Application validation remains required for configurable booking buffer and broader capacity constraints.

### 11.4 Retention and Privacy

The current code stores plate images in parking-session ImageIn and ImageOut fields and stores identity, vehicle, payment, incident, and audit information. A production retention period and deletion policy are not defined in code and remain OQ-DATA-001.

## 12. State Models

### 12.1 Booking

Pending -> Confirmed after successful deposit payment.

Pending -> Expired after payment deadline.

Pending or Confirmed -> Cancelled by eligible cancellation.

Confirmed -> CheckedIn when a linked session is created.

Confirmed -> NoShow after the check-in grace endpoint.

### 12.2 Payment

Pending -> Paid after cash completion or verified VNPay success.

Pending -> Failed when replaced, rejected, failed, or expired under the active payment flow.

Paid -> RefundPending after eligible booking cancellation.

RefundPending -> Refunded after the refund-record operation.

### 12.3 Parking Session and Resources

An active check-in binds the vehicle, card, allocation, and optional booking. Checkout completes or marks the session unpaid and releases the slot and card. Lost-card and rollback operations provide controlled exceptional transitions.

## 13. Non-Functional Requirements

#### NFR-SEC-001 Credential and Transport Security

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: Production deployment shall use HTTPS, keep passwords BCrypt-hashed, validate JWT and VNPay signatures, and supply external service secrets outside committed source configuration.
- Acceptance Criteria: No production secret is committed; HTTPS is enforced; invalid token or VNPay signature is rejected; credential rotation can occur without code changes.
- Gap: Current configuration contains committed secret values and requires remediation and rotation.

#### NFR-SEC-002 Server-side Authorization

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-002
- Requirement: Every non-public API operation shall enforce authenticated server-side authorization appropriate to its actor and ownership rules.
- Acceptance Criteria: Anonymous and wrong-role calls are rejected for every protected controller; account-owned data is filtered or ownership-checked on the server; automated authorization tests cover the endpoint inventory.
- Gap: Current coverage is incomplete; see RISK-SEC-001.

#### NFR-PERF-001 Interactive API Response

- Priority: Should
- Status: REVIEW
- Source: REF-GEN-001
- Requirement: The web client shall stop an ordinary API request after its configured ten-second timeout and show a recoverable failure state.
- Acceptance Criteria: A request exceeding ten seconds is aborted and does not leave the relevant interface permanently busy.
- Notes: Automatic retries are not currently implemented.

#### NFR-CON-001 Booking Concurrency

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-002, REF-GEN-003
- Requirement: Concurrent booking requests shall not persist overlapping Pending or Confirmed reservations for the same concrete slot.
- Acceptance Criteria: A race between otherwise valid requests results in at most one persisted conflicting slot interval, with the other request reported as a conflict.

#### NFR-TIME-001 Time Consistency

- Priority: Must
- Status: REVIEW
- Source: REF-GEN-002
- Requirement: The system shall handle persisted timestamps consistently and shall group revenue and daily pricing boundaries using explicitly documented UTC+7 business dates.
- Acceptance Criteria: A payment near UTC midnight appears in the correct Vietnam reporting day and daily-cap calculation uses the same documented business zone.

#### NFR-AVAIL-001 External Service Degradation

- Priority: Should
- Status: REVIEW
- Source: REF-GEN-001, REF-GEN-002
- Requirement: OCR, email, Google, or VNPay failure shall return a controlled error without corrupting booking, session, or payment state.
- Acceptance Criteria: Staff retains manual plate entry after OCR failure; a failed online payment remains non-paid; failed email or Google verification does not create an unverified password account.

#### NFR-USAB-001 Responsive Role Workspace

- Priority: Should
- Status: REVIEW
- Source: REF-GEN-001
- Requirement: Role workspaces shall remain operable on supported desktop and mobile browser sizes and shall expose validation and loading feedback for business actions.
- Acceptance Criteria: Primary forms and tables are usable without hidden mandatory controls and rejected operations display actionable feedback.

#### NFR-OBS-001 Operational Traceability

- Priority: Should
- Status: REVIEW
- Source: REF-GEN-002, REF-GEN-003
- Requirement: Security-sensitive and operationally significant changes shall produce sufficient logs or audit records to diagnose actor, time, object, and outcome.
- Acceptance Criteria: Authentication failures, payment callbacks, configuration changes, and privileged record changes are traceable without logging secrets.

## 14. Configuration Requirements

| Key or setting | Current behavior | Default or note |
|---|---|---|
| BUFFER_TIME_MINUTES | Service-level booking and extension separation | 30 |
| WALKIN_STAY_THRESHOLD_HOURS | Determines whether active walk-ins count for near-term booking capacity | 2 |
| APPLY_SEGMENTED_PRICING | Enables policy reselection by pricing segment | Missing means false |
| PaymentSettings:CashRoundingUnit | Cash amount rounding unit | 500 in current API configuration |
| JWT settings | Token signing and lifetime | Environment-specific; secret must be externalized |
| VNPay settings | Merchant, secret, endpoints, frontend return | Environment-specific |
| Plate Recognizer settings | Endpoint and token | Environment-specific |
| Frontend API base URL | API origin and prefix | Must match deployed backend /api routes |

Dynamic configuration changes shall validate type and range before affecting business logic.

## 15. Error Handling and Recovery

- Validation errors shall identify the rejected business condition without exposing secrets or stack traces.
- Duplicate and overlap conflicts shall return a deterministic conflict or validation response.
- Payment callbacks shall be idempotent for a payment order and shall not double-apply a successful transition.
- Checkout rollback shall restore only states that can be safely reversed and shall not reverse a completed paid transaction.
- Background-worker failure shall be logged and shall not terminate the API process without a recoverable error path.

## 16. Reporting and Audit

Revenue is calculated at query time from PAID payments. Reports can group by day, month, year, building, and vehicle type and include an overall row. Monthly-subscription revenue is zero in the active model. Audit logs are separate operational records and do not replace payment-source revenue calculation.

PricingCalculationLog is produced by the explicit calculate-and-log path; the SRS does not claim that every ordinary fee calculation is persisted to that table.

## 17. Migration and Compatibility

- Existing MonthlySubscription, SubscriptionPriceConfig, and subscription foreign-key columns may remain to preserve migration compatibility, but they shall not appear as active product scope.
- Existing RevenueStatistic tables may remain without being the source of current reports.
- Existing GracePeriodRule data may remain until the active and legacy fee engines are reconciled.
- Deprecated IDs and historical rules remain reserved.
- Database deployments shall apply the booking exclusion migration and enable btree_gist before relying on concurrent slot protection.

## 18. Traceability Matrix

| Feature | Use case | Functional requirements | Principal verification |
|---|---|---|---|
| F-DRV-001 | UC-DRV-001 | FR-ACC-001 to FR-ACC-003, FR-DRV-001 | Auth, registration, token, and vehicle service tests |
| F-STR-001 | UC-STR-001 | FR-STR-001 to FR-STR-003, FR-CFG-001 | Facility CRUD, range, and availability tests |
| F-BOOK-001 | UC-BOOK-001 | FR-BOOK-001 to FR-BOOK-006 | Booking service, capacity, cancellation, worker, and concurrency tests |
| F-ALLOC-001 | UC-ALLOC-001 | FR-ALLOC-001, FR-STR-002 | Allocation and capacity tests |
| F-OPS-001 | UC-OPS-001 | FR-OPS-001 to FR-OPS-003 | OCR adapter, normalization, and check-in tests |
| F-SESSION-001 | UC-SESSION-001 | FR-SESSION-001, FR-SESSION-002 | Session query and extension tests |
| F-OPS-002 | UC-OPS-002 | FR-OPS-004, FR-OPS-005 | Checkout, mismatch, unpaid, lost-card, and rollback tests |
| F-PRICE-001 | UC-PRICE-001 | FR-PRICE-001, FR-CFG-001 | Policy overlap, priority, and cleanup tests |
| F-PRICE-002 | UC-PRICE-002 | FR-PRICE-002, FR-PRICE-003 | Pricing engine segment, threshold, cap, and penalty tests |
| F-PAY-001 | UC-PAY-001 | FR-PAY-001 to FR-PAY-003 | Cash, VNPay signature, callback, idempotency, and refund-state tests |
| F-INC-002 | UC-INC-002 | FR-INC-001, FR-CARD-001, FR-BLK-001 | Incident, card, blacklist, and penalty tests |
| F-MON-001 | UC-MON-001 | FR-RPT-001, FR-AUD-001 | Revenue UTC+7 and audit authorization tests |
| F-MONTH-001 | UC-MONTH-001 | FR-MONTH-001 | Deprecated; verify absence from active API and UI inventory |

## 19. Verification and Acceptance Strategy

### 19.1 Test Levels

- Unit tests verify booking, pricing, payment, normalization, and service rules.
- Integration tests verify controller authorization, persistence constraints, migrations, workers, and external gateway adapters.
- End-to-end tests verify role workspaces, camera fallback, booking-to-payment-to-check-in, checkout, and reporting.
- Security tests enumerate every controller action and verify anonymous, wrong-role, and wrong-owner rejection.

### 19.2 Minimum Release Evidence

- All Must requirements have a passing test or an approved manual test record.
- No active traceability row is missing verification evidence.
- Monthly Subscription is absent from active navigation and public API inventory.
- Production secrets are externalized and rotated.
- Frontend API base URL matches deployed API routes.
- Booking exclusion migration is applied in the target database.

## 20. Risks and Technical Gaps

| Risk ID | Severity | Description | Required disposition |
|---|---|---|---|
| RISK-SEC-001 | Critical | Many operational controllers lack server-side authorization attributes. Frontend route guards can be bypassed. | Add explicit authentication, roles, ownership checks, and endpoint tests before production. |
| RISK-SEC-002 | Critical | Sensitive configuration values are committed in API settings. | Revoke or rotate exposed values and use environment or secret storage. |
| RISK-AUTH-001 | High | Controller and frontend contain a login-OTP branch, but normal AuthService login neither generates nor requires that OTP. | Decide direct JWT login versus MFA, then remove the dead branch or implement the complete flow. |
| RISK-API-001 | High | Frontend default base path can be /api/v1 while backend controller paths use /api. | Require and validate a deployment-specific base URL. |
| RISK-BOOK-001 | Medium | Frontend constants still describe one-hour minimum, eight-hour maximum, and forty-five-minute grace, while backend enforces four-hour minimum and thirty-minute grace and no matching maximum. | Make the frontend consume backend configuration or align constants and validation. |
| RISK-PRICE-001 | High | GracePeriodRule is modeled and used by legacy fee service but ignored by active PricingEngine. | Choose one fee engine and specify or remove grace behavior. |
| RISK-PRICE-002 | Medium | Pricing priority is supported by backend and schema but is not exposed clearly by the manager pricing UI. | Add a priority control or document an API-only administration process. |
| RISK-LEG-001 | Medium | Dormant subscription and revenue-statistic schema can be mistaken for active capability. | Label legacy objects and plan a compatibility-safe cleanup migration. |
| RISK-WEB-001 | Low | Client retryAttempts is configured but unused. | Implement bounded retry for safe requests or remove the misleading setting. |

## 21. Open Questions

| Question ID | Question | Owner | Impact |
|---|---|---|---|
| OQ-GEN-001 | Should the product use direct password login or mandatory login OTP/MFA? | Product and Security | Authentication acceptance criteria and UI flow |
| OQ-DATA-001 | What are the retention and deletion periods for entry/exit images, plates, incidents, audit logs, and payment metadata? | Product and Legal | Privacy, storage, and deletion implementation |
| OQ-PRICE-001 | Should GracePeriodRule be applied by the active PricingEngine or retired with the legacy fee service? | Product and Backend | Fee correctness |
| OQ-PAY-001 | Is the current refund endpoint only an internal record transition, or must it invoke an external refund gateway? | Product and Finance | Financial reconciliation |
| OQ-OPS-001 | Which roles may approve refunds at the API boundary? | Product and Security | Authorization matrix |

## 22. Validation Checklist

- [x] Document baseline identifies current frontend and backend sources.
- [x] Architecture matches Next.js and .NET Clean Architecture code.
- [x] Active actor and route inventory is represented.
- [x] Monthly Subscription is marked DEPRECATED and remains traceable.
- [x] Booking capacity, buffer, exclusion constraint, and timing are aligned with backend behavior.
- [x] Pricing priority and segmented-pricing switch are represented.
- [x] OCR, plate normalization, images, and manual fallback are represented.
- [x] Revenue is defined from PAID payments using UTC+7 and not legacy statistics.
- [x] Server authorization and login-OTP gaps are disclosed rather than claimed as complete.
- [x] Functional requirements use mandatory shall wording and contain acceptance criteria.
- [x] Feature-to-use-case-to-requirement traceability is present.
- [ ] Product owner has resolved open questions.
- [ ] Security owner has accepted or remediated critical risks.
- [ ] QA has attached release evidence for all Must requirements.

---

End of PBMS SRS version 1.2, status REVIEW.
