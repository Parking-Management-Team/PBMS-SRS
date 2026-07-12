# Capstone Project Report
## Report 3 – Software Requirement Specification
**Project**: Parking Building Management System (PBMS)  
**Date**: July 12, 2026  

---

## Table of Contents
- [I. Project Report](#i-project-report)
  - [1. Status Report](#1-status-report)
  - [2. Team Involvements](#2-team-involvements)
  - [3. Issues/Suggestions](#3-issuessuggestions)
- [II. Software Requirement Specification](#ii-software-requirement-specification)
  - [1. Product Overview](#1-product-overview)
  - [2. User Requirements](#2-user-requirements)
    - [2.1 Actors](#21-actors)
    - [2.2 Use Cases](#22-use-cases)
  - [3. Functional Requirements](#3-functional-requirements)
    - [3.1 System Functional Overview](#31-system-functional-overview)
      - [3.1.1 Screens Flow](#311-screens-flow)
      - [3.1.2 Screen Descriptions](#312-screen-descriptions)
      - [3.1.3 Screen Authorization](#313-screen-authorization)
      - [3.1.4 Non-Screen Functions](#314-non-screen-functions)
      - [3.1.5 Entity Relationship Diagram](#315-entity-relationship-diagram)
    - [3.2 Operations Module](#32-operations-module)
    - [3.3 Facility & Pricing Module](#33-facility--pricing-module)
  - [4. Non-Functional Requirements](#4-non-functional-requirements)
    - [4.1 External Interfaces](#41-external-interfaces)
    - [4.2 Quality Attributes](#42-quality-attributes)
  - [5. Requirement Appendix](#5-requirement-appendix)
    - [5.1 Business Rules](#51-business-rules)
    - [5.2 Common Requirements](#52-common-requirements)
    - [5.3 Application Messages List](#53-application-messages-list)

---

# I. Project Report

## 1. Status Report
- **Current Status**: REVIEW.
- **Progress**: All core software requirements have been audited against the C# backend (`parking-system-api`) and React frontend (`parking-system-web`) implementations. 

## 2. Team Involvements
- **BA Team**: Gia Thai (Lead BA) - In charge of requirement specifications, rule modeling, and template formatting.
- **Development Team**: Technical Lead & Developers - In charge of database schema design (EF Core), background services, and API gateways.

## 3. Issues/Suggestions
- **OCR Engine Migration**: Client-side OCR (Tesseract.js) has been replaced with the cloud-based **Plate Recognizer OCR API** for improved license plate recognition accuracy.
- **Optimistic Concurrency Control**: Implemented PostgreSQL system column `xmin` (mapped to `RowVersion` in EF Core) to resolve race conditions on shared database resources (e.g., Slot capacity, payment transactions).

---

# II. Software Requirement Specification

## 1. Product Overview
The **Parking Building Management System (PBMS)** is a modern parking facility management solution designed for multi-floor parking buildings. It replaces manual check-in/check-out and ledger-based tracking with a digital simulation platform. 

### Context Diagram
```text
[Insert Context Diagram here]
Description: The diagram presents the boundary and connections between PBMS and external entities:
- Drivers (Mobile Portal)
- Parking Staff (Web Portal Gate Operations)
- Parking Managers (Web Portal Administration & Monitoring)
- VNPay Bank Payment Gateway (API Callback and Redirection)
- Plate Recognizer OCR Cloud API (License plate recognition processing)
```

---

## 2. User Requirements

### 2.1 Actors
The system supports the following user roles and system components:

| # | Actor | Description |
|---|---|---|
| 1 | **Parking Manager** | Supervises operations, configures parking slots/zones, creates pricing rules, defines penalty rates, and reviews/approves staff shift reports. |
| 2 | **Parking Staff** | Operates gates. Executes check-in (LPR, Base64 image) and check-out (fee collection, cash payment, VNPay status verification), handles incident reporting (lost cards), and submits shift reports. |
| 3 | **Driver / User** | Registers accounts, manages personal profiles and vehicles, creates bookings, pays booking deposits, registers/renews monthly subscriptions, and makes online payments. |
| 4 | **System** | Background processes executing overdue booking cleanup, pricing policy expiration updates, and overtime parking warnings. |
| 5 | **VNPay Gateway** | External banking gateway verifying payment transactions. |
| 6 | **Plate Recognizer API** | External cloud service resolving license plate strings from captured vehicle photos. |

### 2.2 Use Cases

#### Use Case Diagram
```text
[Insert Use Case Diagram here]
Description: The diagram shows actors (Driver, Staff, Manager) and their associated use cases.
```

#### Use Case Catalogue

| ID | Use Case | Actors | Use Case Description |
|---|---|---|---|
| UC-OPS-001 | Vehicle Check-in | Staff, Driver, Plate Recognizer | Creates a parking session when a vehicle enters the gate by capturing a webcam photo, running LPR, and assigning a physical card. |
| UC-OPS-002 | Vehicle Check-out | Staff, Driver, VNPay Gateway | Concludes a parking session, calculates fees via the Pricing Engine, processes payment (cash/online), and releases the slot. |
| UC-BOOK-001 | Create Booking | Driver, VNPay Gateway | Allows drivers to book a parking space 1-8 hours in advance by paying a deposit fee equal to the base block price. |
| UC-MONTH-001 | Register Subscription | Driver, Manager, VNPay | Allows drivers to register for monthly parking. Cars are assigned a dedicated slot in the `MONTHLY` zone. |
| UC-REP-001 | Submit Shift Report | Staff, Manager | Staff calculates expected cash vs actual cash collected during their shift, submits the report, and awaits approval. |
| UC-REP-002 | Approve Shift Report | Manager | Manager reviews discrepancies, approves or rejects staff shift submissions. |

---

## 3. Functional Requirements

### 3.1 System Functional Overview

#### 3.1.1 Screens Flow
```text
[Insert Screens Flow Diagram here]
Description: Map of the application:
Login -> Home Dashboard
Staff View: -> Check-in Form | Check-out Form | Active Sessions List | Shift Report Submission
Manager View: -> Structure Config (Building, Floor, Zone, Slot) | Pricing Rules Config | Monthly Subscriptions List | Shift Reports Review
Driver View: -> Booking Form | Active Subscriptions | Payment History
```

#### 3.1.2 Screen Descriptions
Below is the registry of screens in the flow:

| # | Feature | Screen | Description |
|---|---|---|---|
| 1 | Authentication | Login / Register | Allows users to authenticate and register driver profiles. |
| 2 | Operations | Gate Check-in | Displays live camera feed (simulated), plate input, card scan input, and allocated parking zones. |
| 3 | Operations | Gate Check-out | Displays session details, calculated price breakdown, payment options (Cash/VNPay QR Code). |
| 4 | Reporting | Shift Report Submit | Staff form to enter actual cash collected, showing system revenue, differences, and notes. |
| 5 | Administration | Manage Pricing Rules | Manager interface to configure pricing policies, rules (Base, Increment, Daily Cap, Grace), and execution order. |
| 6 | Monitoring | Shift Report Review | Manager panel to check and approve staff shift submissions. |

#### 3.1.3 Screen Authorization Matrix

| Screen | Admin | Manager | Staff | Driver |
|---|---|---|---|---|
| Login / Register | X | X | X | X |
| Gate Check-in | | | X | |
| Gate Check-out | | | X | |
| Shift Report Submit | | | X | |
| Manage Pricing Rules | | X | | |
| Shift Report Review | | X | | |
| Structure Config | X | X | | |
| Booking Portal | | | | X |
| Subscription Portal | | | | X |
| System Audit Logs | X | | | |

#### 3.1.4 Non-Screen Functions
Background services and external components running on the server:

| # | Feature | System Function | Description |
|---|---|---|---|
| 1 | Booking | `ExpiredBookingCleanupWorker` | Runs every 5 minutes. Cancels bookings that remain unpaid after the 15-minute payment deadline. |
| 2 | Pricing | `ExpiredPricingPolicyCleanupWorker` | Runs every 12 hours. Changes active policies to 'Expired' once they pass their `effective_end` date. |
| 3 | Warnings | `OvertimeWarningWorker` | Runs every 1 minute. Triggers system warnings for vehicles exceeding the maximum allowed session hours. |
| 4 | Recognition | `PlateRecognizerOcrService` | Integrates with Plate Recognizer Cloud API to parse license plates from Base64 images. |
| 5 | Integration | `VNPayGateway` | Interfaces with the bank to generate secure payment URLs and process callbacks/IPNs. |

#### 3.1.5 Entity Relationship Diagram
The system structure consists of **33 physical tables** in PostgreSQL:

```text
[Insert Physical ERD Diagram here]
```

##### Physical Tables Glossary

| # | Entity/Table | Description |
|---|---|---|
| 1 | `role` | Stores user roles (Admin, Manager, Staff, Driver). |
| 2 | `permission` | Stores application permissions. |
| 3 | `role_permission` | Junction table for N-N relation between roles and permissions. |
| 4 | `account` | Stores user profiles. Staff accounts are mapped to relevant roles. |
| 5 | `building` | Stores parking building definitions. |
| 6 | `floor` | Stores floors belonging to a building. |
| 7 | `vehicle_type` | Defines vehicle classes (e.g., Motorcycle, Car). |
| 8 | `zone` | Defines parking zones within a floor. |
| 9 | `parking_slot` | Specific slots for cars. ô tô Walk-in/Booking use general capacity, ô tô month use slots. |
| 10 | `vehicle` | Registered vehicles owned by drivers. |
| 11 | `card` | RFID/NFC cards (NORMAL for walk-ins/bookings, MONTHLY for subscriptions). |
| 12 | `parking_session` | Individual check-in/out session details. |
| 13 | `incident_type` | Categories of incident rules. |
| 14 | `incident` | Incidents reported (e.g., lost card). |
| 15 | `blacklist` | Blocked vehicles or cards. |
| 16 | `booking` | Advance parking booking records. |
| 17 | `monthly_subscription` | Recurrent monthly subscriptions. |
| 18 | `pricing_policy` | Main pricing container by vehicle type. |
| 19 | `pricing_window` | Time window within a pricing policy. |
| 20 | `payment` | Payment transactions linked to bookings, subscriptions, or sessions. |
| 21 | `revenue_statistic` | Daily financial summary. |
| 22 | `revenue_statistic_payment` | Junction table tracking payments aggregated into revenue statistics. |
| 23 | `notification` | Notifications pushed to accounts. |
| 24 | `audit_log` | System action logs for security auditing. |
| 25 | `pricing_rule` | Core rule configurations inside a pricing policy. |
| 26 | `base_pricing_rule_config` | Configurations for the base pricing duration and fee. |
| 27 | `increment_pricing_rule_config` | Configurations for progressive billing blocks and fee increments. |
| 28 | `daily_cap_rule_config` | Configures the maximum fee cap within a 24-hour window. |
| 29 | `grace_period_rule_config` | Configures grace periods where billing is deferred. |
| 30 | `pricing_calculation_log` | Detailed step-by-step price calculation breakdown for audit. |
| 31 | `SubscriptionPriceConfigs` | Historical price tracking for monthly subscriptions. |
| 32 | `PenaltyConfigs` | Historical penalty fee config for lost cards or incident reports. |
| 33 | `shift_report` | Daily gate staff shift report tracking cash balances. |

---

### 3.2 Operations Module

#### 3.2.1 Vehicle Check-in (UC-OPS-001)
- **Function Trigger**: A vehicle arrives at the gate and triggers gate operation.
- **Description**: The staff clicks "Capture Photo", which compresses the webcam stream into a Base64 string. The backend sends the image to Plate Recognizer API to extract the license plate. The staff selects a normal card (for walk-in/booking) or monthly card (for subscriptions) and confirms check-in.
- **Screen Layout Mockup**:
  ```text
  [Insert Gate Check-in Mockup Screen here]
  ```
- **Function Details**:
  - Validates that the vehicle does not have an active session open in the system.
  - Ensures a pricing policy is active for the vehicle type.
  - Dedacts capacity/allocates zones: suggest `GENERAL` zone for ô tô walk-in/booking, gán assigned slot for monthly cars.

#### 3.2.2 Vehicle Check-out (UC-OPS-002)
- **Function Trigger**: A vehicle arrives at the exit gate.
- **Description**: Staff scans the card or inputs the plate number. The system retrieves the active session, captures the exit webcam image (Base64), calculates the total price, applies cash rounding if paying cash, or generates a VNPay QR redirect URL for online banking. Once paid, the session is completed and the slot is released.
- **Screen Layout Mockup**:
  ```text
  [Insert Gate Check-out Mockup Screen here]
  ```
- **Function Details**:
  - Calculations run through the rule-based `PricingEngine`.
  - Cash rounding applies to cash payments only.
  - VNPay callbacks (IPN) are processed asynchronously to verify transaction status.

---

### 3.3 Facility & Pricing Module

#### 3.3.1 Pricing Rules Configuration
- **Function Trigger**: Manager navigates to the Pricing Management tab.
- **Description**: Manager configures pricing rules (Base, progressive, daily caps, grace periods) and maps them to a vehicle type. Once active, the rule set is locked to prevent edits on active sessions.
- **Function Details**:
  - Enforces that pricing windows within a policy must cover exactly 24 hours.

#### 3.3.2 Shift Report Submission
- **Function Trigger**: Staff shift ends.
- **Description**: The system displays the check-in/out counts, total online transactions, and expected cash balance. Staff enters the actual cash count. If there's a difference, staff must provide a note before submitting.
- **Function Details**:
  - `difference_amount` = `actual_cash_amount` - `expected_cash_amount`.
  - Rejects new logins for the staff member until the current shift report is submitted.

---

# 4. Non-Functional Requirements

## 4.1 External Interfaces
- **VNPay Gateway**: Communication over HTTPS using HMAC-SHA512 secure hashing to verify transaction integrity.
- **Plate Recognizer API**: Image payload sent as POST requests containing Base64 encoded JPEG strings. Returns JSON containing parsed alphanumeric string plates.

## 4.2 Quality Attributes
- **Usability**: Response time for camera capture and LPR processing must not exceed 2.5 seconds to prevent gate queue build-up.
- **Reliability**: Database transactions use optimistic concurrency mapping (`xmin` row versions) to prevent double allocation of capacity.
- **Performance**: Key queries (such as checking slot occupancy or building capacity) must respond within 500ms.

---

# 5. Requirement Appendix

## 5.1 Business Rules

| ID | Rule Definition |
|---|---|
| BR-HW-001 | All license plate recognition (LPR) must be processed through the external Plate Recognizer API. |
| BR-BOOK-001 | Booking deposits are calculated using the base block price of the booking policy at creation time. |
| BR-MONTH-001 | Monthly subscriptions are restricted to one vehicle per subscription. |
| BR-FEE-001 | Concurrency control must use PostgreSQL `xmin` column versions to prevent data overwrites. |

## 5.2 Common Requirements
- All entity deletions must implement soft-deletion via the `ISoftDeletable` interface (retaining files for audit).
- Date and time formatting across database records must use local timezone conversion (UTC+7).

## 5.3 Application Messages List

| # | Message Code | Message Type | Context | Content |
|---|---|---|---|---|
| 1 | `MSG-ERR-001` | In-line Red | Check-in input validation | The license plate field is required. |
| 2 | `MSG-ERR-002` | Alert Dialog | Double session check | Vehicle already has an active parking session. |
| 3 | `MSG-ERR-003` | Alert Dialog | Capacity check | Building has no available parking capacity. |
| 4 | `MSG-SUC-001` | Toast | Gate operations | Check-in session created successfully. |
| 5 | `MSG-SUC-002` | Toast | Payment | Payment processed and session closed. |
