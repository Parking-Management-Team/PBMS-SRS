---
document_id: PBMS-FEATURE-ACTOR
title: PBMS Feature and Actor Matrix
version: 1.2
status: REVIEW
change_request: CR-GEN-001
last_updated: 2026-07-18
---

# PBMS Feature and Actor Matrix

This companion document maps the current parking-system-web pages and parking-system-api behavior to actors. The normative requirement details are in PBMS_SRS_Document.md.

## 1. Interpretation Rules

- UI means a current page, component, or client flow exists.
- API means a current controller/application flow exists.
- Enforced means the backend endpoint declares authentication or role enforcement.
- Partial means code exists but the end-to-end flow or security boundary is incomplete.
- Legacy means an entity, column, configuration API, or old document remains without an active product flow.
- A visible or role-guarded frontend page is not proof of server-side authorization.

## 2. Actor Summary

| Actor | Current purpose | Authentication context |
|---|---|---|
| Guest | Registration, email verification, password login, Google login | Public authentication routes |
| Driver | Own vehicles, bookings, sessions, payments, history, reports | JWT stored by web client; Driver route guard |
| Staff | Gate operations, monitoring, cards, incidents, and blacklists | JWT stored by web client; Staff route guard |
| Manager | Facilities, pricing, operations, finance, reports | JWT stored by web client; Manager route guard |
| Admin | Accounts, facilities, settings, incidents, blacklist, audit | JWT stored by web client; Admin route guard |
| VNPay | Online payment processing and callback | Signed external callback |
| Plate Recognizer | License plate OCR | Backend external-service credential |
| Background workers | Timed booking, overtime, and pricing maintenance | In-process hosted services |

## 3. Actor-to-Feature Matrix

Legend: P = primary actor, S = supporting actor, V = view or own-data access, dash = no current actor flow.

| Feature ID | Feature | Guest | Driver | Staff | Manager | Admin | System or external |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|
| F-DRV-001 | Registration, authentication, profile, vehicles | P | P | V | V | S | Google, SMTP |
| F-STR-001 | Parking structure and capacity | - | V | V | P | P | - |
| F-BOOK-001 | Booking lifecycle | - | P | S | V | - | Booking worker, VNPay |
| F-OPS-001 | Camera, OCR, and check-in | - | - | P | V | - | Plate Recognizer |
| F-ALLOC-001 | Zone and slot allocation | - | - | P | P | S | - |
| F-SESSION-001 | Parking sessions and extension | - | V | P | V | - | Pricing engine |
| F-OPS-002 | Checkout and exceptional exit | - | - | P | V | - | Pricing engine |
| F-PAY-001 | Payments and refund state | - | P | P | P | V | VNPay |
| F-PRICE-001 | Pricing policies | - | - | - | P | S | Cleanup worker |
| F-PRICE-002 | Fee and penalties | - | V | V | P | - | Pricing engine |
| F-INC-002 | Incidents, cards, blacklists | - | V | P | P | P | - |
| F-MON-001 | Monitoring, revenue, shift, audit | - | V | P | P | P | Revenue service |
| F-MONTH-001 | Monthly Subscription | - | - | - | - | - | DEPRECATED |

The matrix describes current product surfaces, not a complete permission policy. Server-side authorization must be validated per endpoint.

## 4. Guest Capabilities

| Capability | UI | API | Current behavior |
|---|:---:|:---:|---|
| Request registration OTP | Yes | Yes | Six-digit email OTP; five-minute validity; sixty-second resend cooldown. |
| Verify registration OTP | Yes | Yes | Five failed attempts cause fifteen-minute lockout; successful verification returns a ten-minute registration token. |
| Register password account | Yes | Yes | Creates a Driver account after verification. |
| Password login | Yes | Yes | BCrypt verification followed by direct JWT issuance. |
| Google login or registration | Yes | Yes | Existing account signs in; new account completes email verification. |
| Login OTP or MFA | Partial | Partial | Endpoint and UI branch exist, but the normal login service neither generates nor requires login OTP. |
| Forgot password | No | No | Not implemented in the active baseline. |
| Public support ticket | No | No | Help content exists inside dashboards; no public support workflow was found. |

## 5. Driver Capabilities

| Capability | UI | API | Key conditions |
|---|:---:|:---:|---|
| View and edit profile | Yes | Yes | Account APIs have authorization, but ownership and role tests remain required. |
| Manage own vehicles | Yes | Yes | Normalized unique plate; active-session vehicle cannot be archived. |
| Claim ownerless walk-in vehicle | Yes | Yes | Vehicle must currently have no account owner. |
| Create booking | Yes | Yes | At least fifteen-minute lead; minimum four hours; eligible vehicle and capacity. |
| Select a concrete slot | Yes | Yes | Cars only; slot must be compatible and conflict-free. |
| Pay booking deposit | Yes | Yes | Deposit equals the pricing estimate for the full planned period. |
| Modify or cancel booking | Yes | Yes | Update is generally Pending-only; cancellation supports Pending and Confirmed. |
| Extend booking or session | Yes | Yes | Availability rechecked; extra payment created when required. |
| View sessions and history | Yes | Yes | Driver pages present active and historical records. |
| View or initiate payments | Yes | Yes | Cash is operationally staff-facing; online flow redirects to VNPay. |
| View reports | Yes | Yes | Driver report page exists; scope is limited to exposed account data. |
| Purchase Monthly Subscription | No | No | Deprecated; no active frontend or backend flow. |

## 6. Staff Capabilities

| Capability | UI | API | Key conditions |
|---|:---:|:---:|---|
| Camera capture and OCR | Yes | Yes | Browser permission and secure context; manual correction remains available. |
| Check booking at entry | Yes | Yes | Plate, booking, timing, structure, blacklist, and state checks. |
| Walk-in check-in | Yes | Yes | Prevents duplicate active vehicle, card, or slot and validates effective capacity. |
| Assign zone or car slot | Yes | Yes | Allocation must match building and vehicle type. |
| Monitor active sessions and slots | Yes | Yes | Dynamic reservation and operational states are exposed. |
| Start and complete checkout | Yes | Yes | Plate/card match, fee calculation, payment, and resource release. |
| Record unpaid exit | Yes | Yes | Creates incident and blacklist and releases operational resources. |
| Handle lost or replacement card | Yes | Yes | Supports Lost state, incident, blacklist, replacement, and safe rollback. |
| Manage incidents and types | Yes | Yes | Staff has operational incident flows and incident-type reference. |
| Manage cards and blacklist entries | Yes | Yes | Used during entry and exit controls. |

## 7. Manager Capabilities

| Capability | UI | API | Key conditions |
|---|:---:|:---:|---|
| Manage buildings, floors, zones, slots | Yes | Yes | Includes zone BookingLimitRate and operational state. |
| View bookings and sessions | Yes | Yes | Operational management pages exist. |
| Manage pricing policies and rules | Yes | Yes | Backend supports Priority; current pricing UI does not clearly expose it. |
| Manage incidents, cards, blacklists, vehicles | Yes | Yes | Management pages and controllers exist. |
| Review payments and refunds | Yes | Yes | Refund endpoint records REFUND_PENDING to REFUNDED; external refund is not claimed. |
| View revenue | Yes | Yes | Calculated dynamically from PAID payments in UTC+7. |
| Configure Monthly Subscription price | Legacy | Yes | SubscriptionPriceConfig remains, but it has no active subscription consumer. |

## 8. Admin Capabilities

| Capability | UI | API | Key conditions |
|---|:---:|:---:|---|
| Manage accounts and roles | Yes | Yes | Account controller has authorization; permission-matrix enforcement is incomplete. |
| Manage selected facilities | Yes | Yes | Admin facility pages exist. |
| View audit logs | Yes | Yes | Audit-log controller has authorization. |
| Manage system configuration | Yes | Yes | Generic key-value settings; business services consume selected keys. |
| Manage incidents and blacklists | Yes | Yes | Pages exist; endpoint authorization must be reviewed. |
| View admin dashboard | Partial | Partial | Route exists; content and source completeness vary. |

## 9. System and External Actors

### 9.1 Booking Cleanup Worker

- Runs approximately every five minutes.
- Changes unpaid Pending bookings past deadline to Expired.
- Changes Confirmed bookings past check-in grace to NoShow.

### 9.2 Overtime Warning Worker

- Runs approximately every minute.
- Evaluates overtime warning conditions for parking sessions.

### 9.3 Pricing Policy Cleanup Worker

- Runs approximately every twelve hours.
- Deactivates active policies whose effective period has ended.

### 9.4 VNPay

- Receives signed online-payment redirects.
- Returns signed IPN and browser-return parameters.
- Does not provide a documented automatic-refund flow in the current application service.

### 9.5 Plate Recognizer

- Receives Base64 image content from the backend with Vietnam region context.
- Returns plate candidates; the backend initially selects the highest-confidence result.
- Does not replace staff verification.

## 10. Workflow Ownership

| Workflow | Initiator | Supporting actors | System result |
|---|---|---|---|
| Registration | Guest | SMTP or Google | Driver account and JWT after required verification |
| Booking | Driver | Pricing engine, VNPay, cleanup worker | Pending, Confirmed, Expired, Cancelled, CheckedIn, or NoShow booking |
| Entry | Staff | OCR, allocation, booking service | Active session, active card, occupied slot where applicable |
| Extension | Driver or Staff | Pricing engine, VNPay | Updated planned end and optional additional payment |
| Exit | Staff | Pricing engine, payment service | Completed or UNPAID session and released resources |
| Incident | Staff | Manager or Admin | Recorded issue, penalty, card state, or blacklist control |
| Revenue report | Manager | Revenue service | Payment-derived UTC+7 aggregation |

## 11. Authorization Coverage Warning

The following controller groups currently declare authorization: Accounts, AuditLogs, PricingPolicies, PricingEngine, PenaltyConfigs, and SubscriptionPriceConfigs.

Many other operational controllers do not declare authorization attributes, including booking, structure, session, payment, revenue, card, incident, blacklist, vehicle, dashboard, and system-configuration areas. This list is an implementation audit result, not permission to expose the endpoints publicly.

Production acceptance requires:

- authenticated-by-default API policy or explicit authorization on every non-public endpoint;
- role checks matching this document's intended actor ownership;
- server-side account and resource ownership validation;
- automated anonymous, wrong-role, and wrong-owner tests.

## 12. Deprecated and Dormant Capability Map

| Item | Classification | Current handling |
|---|---|---|
| Monthly Subscription purchase, renewal, and use | DEPRECATED | Removed from active API and frontend; excluded from acceptance scope. |
| MonthlySubscription entity and database table | Legacy schema | Retained for compatibility and historical data. |
| MonthlySubscriptionId fields | Legacy schema | Not used by current session, payment, booking, or revenue flow. |
| SubscriptionPriceConfig | Dormant supporting API | Persists configuration but has no active subscription consumer. |
| RevenueStatistic tables | Dormant | Current revenue reads PAID Payment records directly. |
| GracePeriodRule in active fee calculation | Partial | Data model remains; active PricingEngine does not apply it. |
| Stored permission matrix | Partial | Entities exist; endpoint enforcement does not consistently use them. |
| Login OTP verification | Partial and unreachable | Controller and frontend branch exist; normal login does not enter the flow. |

## 13. Feature Acceptance Pointers

| Feature ID | Normative SRS section | Highest-priority validation |
|---|---|---|
| F-DRV-001 | FR-ACC-001 to FR-DRV-001 | Email verification, direct login behavior, vehicle ownership |
| F-STR-001 | FR-STR-001 to FR-STR-003 | Hierarchy, capacity, BookingLimitRate |
| F-BOOK-001 | FR-BOOK-001 to FR-BOOK-006 | Four-hour minimum, capacity, buffer, payment, cleanup |
| F-OPS-001 | FR-OPS-001 to FR-OPS-003 | Camera fallback, OCR normalization, entry validation |
| F-ALLOC-001 | FR-ALLOC-001 | Type-compatible allocation and concurrency |
| F-SESSION-001 | FR-SESSION-001 to FR-SESSION-002 | Query scope, extension, conflict cap |
| F-OPS-002 | FR-OPS-004 to FR-OPS-005 | Checkout, unpaid, lost card, rollback |
| F-PAY-001 | FR-PAY-001 to FR-PAY-003 | Cash, signed VNPay, idempotency, refund state |
| F-PRICE-001 | FR-PRICE-001 | Effective range, overlap, priority, cleanup |
| F-PRICE-002 | FR-PRICE-002 to FR-PRICE-003 | Segmentation, blocks, threshold, cap, penalties |
| F-INC-002 | FR-INC-001, FR-CARD-001, FR-BLK-001 | Incident lifecycle and operational controls |
| F-MON-001 | FR-RPT-001, FR-AUD-001 | PAID-only revenue, UTC+7, and audit access |
| F-MONTH-001 | FR-MONTH-001 | Verify feature remains absent from active surfaces |

## 14. Review Items

- Decide whether login uses direct JWT issuance or mandatory OTP/MFA.
- Define the production server-side role and ownership policy.
- Decide whether the active PricingEngine must apply GracePeriodRule.
- Define who may approve refunds.
- Define image, plate, incident, payment, and audit retention periods.

---

End of PBMS Feature and Actor Matrix version 1.2, status REVIEW.
