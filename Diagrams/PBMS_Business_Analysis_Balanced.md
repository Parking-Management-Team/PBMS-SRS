# PBMS Business Process Swimlanes - Balanced Detail

## Purpose

This document presents review-friendly swimlane diagrams for the main PBMS business processes. It preserves the principal actors, decisions, state changes, exception paths, and background processing while omitting exact thresholds, formulas, configuration values, and low-level implementation details. For normative rules and precise values, refer to `PBMS_Business_Analysis.md` and `../PBMS_SRS_Document.md`.

## Conventions

- Decision labels describe the business question without embedding detailed parameters.
- Rejected or unsuccessful paths stop without changing a successful business state unless explicitly shown.
- Payment, notification, recognition, and background services are shown only where they materially affect the process outcome.

## 1. Registration and Authentication

**Traceability:** UC-DRV-001; FR-ACC-001 to FR-ACC-003; BR-ACC-001 to BR-ACC-004.

```plantuml
@startuml
title Registration and Authentication - Balanced Detail
skinparam WrapWidth 250
skinparam DefaultFontSize 40
skinparam TitleFontSize 60
skinparam TitleFontStyle bold
skinparam SwimlaneTitleFontSize 60
skinparam SwimlaneTitleFontStyle bold
skinparam ArrowFontSize 30
skinparam Nodesep 100
skinparam Ranksep 100
skinparam shadowing false
skinparam activity {
  BackgroundColor #F7F7F7
  BorderColor #454545
  DiamondBackgroundColor #FFF2CC
  DiamondBorderColor #8A6D1D
  StartColor #2E7D32
  EndColor #2E7D32
}

|Guest|
start
if (Account already exists?) then (No)
  :Choose password or Google registration;
  if (Registration method?) then (Password)
    :Request and submit email verification code;
    |PBMS|
    :Issue and validate verification code;
    if (Verification allowed and valid?) then (No)
      :Reject or temporarily restrict verification;
      |Guest|
      :Retry when permitted;
      stop
    endif
    :Issue registration approval;
    |Guest|
    :Submit profile and password;
    |PBMS|
    if (Registration information valid?) then (No)
      :Reject account creation;
      stop
    endif
  else (Google)
    |Google Identity|
    :Validate Google identity;
    |PBMS|
    if (Linked account exists?) then (No)
      |Guest|
      :Complete verified profile information;
      |PBMS|
      if (New account information valid?) then (No)
        :Reject account creation;
        stop
      endif
    endif
  endif
  :Create active Driver account;
endif

|Guest|
:Submit password or Google credential;
|PBMS|
:Authenticate account and status;
if (Login accepted?) then (Yes)
  :Create authenticated session;
  |User|
  :Access authorized features;
  |PBMS|
  if (Session later becomes unauthorized?) then (Yes)
    :End local authenticated session;
  endif
else (No)
  |Guest|
  :Receive login failure;
endif
stop
@enduml
```

## 2. Parking Structure, Pricing, and Configuration Management

**Traceability:** UC-STR-001; UC-PRICE-001; FR-STR-001 to FR-STR-003; FR-PRICE-001; FR-CFG-001.

```plantuml
@startuml
title Parking Structure, Pricing, and Configuration Management - Balanced Detail
skinparam WrapWidth 250
skinparam DefaultFontSize 40
skinparam TitleFontSize 60
skinparam TitleFontStyle bold
skinparam SwimlaneTitleFontSize 60
skinparam SwimlaneTitleFontStyle bold
skinparam ArrowFontSize 30
skinparam Nodesep 100
skinparam Ranksep 100
skinparam shadowing false
skinparam activity {
  BackgroundColor #F7F7F7
  BorderColor #454545
  DiamondBackgroundColor #FFF2CC
  DiamondBorderColor #8A6D1D
  StartColor #2E7D32
  EndColor #2E7D32
}

|Manager or Admin|
start
:Open management workspace;
if (Management area?) then (Parking structure)
  :Create or update facility hierarchy,\nvehicle types, or parking slots;
  |PBMS|
  :Validate authorization, relationships,\nand operational values;
elseif (Pricing policy)
  |Manager|
  :Define policy scope, effective period,\nrules, caps, and priority;
  |PBMS|
  :Check policy validity and conflicts;
else (System configuration)
  |Manager or Admin|
  :Update a supported setting;
  |PBMS|
  :Check setting type and allowed range;
endif

if (Management data acceptable?) then (Yes)
  |Database|
  :Save management data;
  |PBMS|
  :Make active data available to operations;
else (No)
  |Manager or Admin|
  :Review validation result and correct data;
  stop
endif

|Policy Cleanup Worker|
:Find active policies that are no longer effective;
|Database|
:Deactivate eligible policies;
|Manager or Admin|
:Use current management data;
stop
@enduml
```

## 3. Booking Lifecycle

**Traceability:** UC-BOOK-001; UC-PAY-001; FR-BOOK-001 to FR-BOOK-006; BR-BOOK-001 to BR-BOOK-011; BR-PAY-001 to BR-PAY-003.

```plantuml
@startuml
title Booking Lifecycle - Balanced Detail
skinparam shadowing false
skinparam activity {
  BackgroundColor #F7F7F7
  BorderColor #454545
  DiamondBackgroundColor #FFF2CC
  DiamondBorderColor #8A6D1D
  StartColor #2E7D32
  EndColor #2E7D32
}

|Driver|
start
:Select owned vehicle, facility,\nplanned time, and optional slot;
|PBMS|
:Validate ownership, time, facility,\nvehicle compatibility, and restrictions;
if (Booking request eligible?) then (No)
  |Driver|
  :Correct information or choose another option;
  stop
endif

|PBMS|
if (A concrete slot is requested?) then (Yes)
  :Check slot eligibility and schedule conflicts;
endif
:Check effective parking capacity;
if (Suitable capacity available?) then (No)
  |Driver|
  :Choose another time, area, or slot;
  stop
endif

|Pricing Service|
:Estimate the planned parking fee;
|PBMS|
:Create pending booking and payment deadline;
:Create the current payment request;
|Payment Gateway|
:Process online payment and return result;
|PBMS|
if (Verified payment successful?) then (Yes)
  :Record payment once and confirm booking;
  |Driver|
  :Receive booking confirmation;
else (No)
  :Keep payment unsuccessful;
  |Driver|
  :Receive payment result;
endif

|Driver|
if (Next booking action?) then (Modify or extend)
  :Submit revised booking plan;
  |PBMS|
  :Recheck state, availability, and fee impact;
  if (Update acceptable?) then (Yes)
    :Update booking;
    if (Additional payment required?) then (Yes)
      :Create an additional payment request;
    endif
  else (No)
    |Driver|
    :Receive update rejection;
    stop
  endif
elseif (Cancel)
  |PBMS|
  if (Booking can be cancelled?) then (Yes)
    :Cancel booking;
    if (Paid booking qualifies for refund?) then (Yes)
      :Open refund handling;
    endif
  else (No)
    |Driver|
    :Receive cancellation rejection;
    stop
  endif
else (Wait for check-in)
endif

|Booking Cleanup Worker|
:Review bookings that passed action deadlines;
|PBMS|
if (Unpaid booking expired?) then (Yes)
  :Mark booking expired;
elseif (Confirmed booking missed check-in?) then (Yes)
  :Mark booking as no-show;
endif
|Driver|
:View the current booking and payment state;
stop
@enduml
```

## 4. Gate Check-in and Allocation

**Traceability:** UC-OPS-001; UC-ALLOC-001; FR-OPS-001 to FR-OPS-003; FR-ALLOC-001; BR-ALLOC-001 to BR-ALLOC-003.

```plantuml
@startuml
title Gate Check-in and Allocation - Balanced Detail
skinparam shadowing false
skinparam activity {
  BackgroundColor #F7F7F7
  BorderColor #454545
  DiamondBackgroundColor #FFF2CC
  DiamondBorderColor #8A6D1D
  StartColor #2E7D32
  EndColor #2E7D32
}

|Staff|
start
:Start gate check-in;
|Gate Camera|
if (Automatic plate recognition available?) then (Yes)
  :Capture and recognize license plate;
else (No)
  :Keep manual plate entry available;
endif
|Staff|
:Verify or correct plate;
:Provide card and facility information;

|PBMS|
:Find vehicle and eligible booking;
:Check restrictions, active use,\nfacility, and vehicle compatibility;
if (Check-in context valid?) then (No)
  |Staff|
  :Explain rejected check-in;
  stop
endif

|Allocation Service|
:Select a compatible active parking area;
if (Vehicle requires a concrete slot?) then (Yes)
  :Select an eligible non-conflicting slot;
endif
:Check effective capacity and allocation conflicts;
if (Suitable allocation available?) then (No)
  |Staff|
  :Direct driver to another option;
  stop
endif

|PBMS|
:Create active parking session;
:Bind vehicle, card, allocation,\nand eligible booking;
:Store entry evidence when available;
:Activate card and occupy assigned slot;
:Mark linked booking checked in;
|Staff|
:Allow vehicle entry;
stop
@enduml
```

## 5. Session Query and Extension

**Traceability:** UC-SESSION-001; UC-PRICE-002; UC-PAY-001; FR-SESSION-001; FR-SESSION-002; FR-PRICE-002; FR-PRICE-003.

```plantuml
@startuml
title Session Query and Extension - Balanced Detail
skinparam shadowing false
skinparam activity {
  BackgroundColor #F7F7F7
  BorderColor #454545
  DiamondBackgroundColor #FFF2CC
  DiamondBorderColor #8A6D1D
  StartColor #2E7D32
  EndColor #2E7D32
}

|Driver or Staff|
start
:Request active or historical session;
|PBMS|
if (Actor may access session?) then (No)
  |Driver or Staff|
  :Receive access rejection;
  stop
endif
:Retrieve timing, allocation, evidence,\nand payment context;
|Pricing Service|
:Calculate current fee estimate;
|Driver or Staff|
:View session details and estimate;

if (Extension requested?) then (Yes)
  :Submit requested checkout time;
  |PBMS|
  :Check session state and extension eligibility;
  if (Extension eligible?) then (No)
    |Driver or Staff|
    :Receive extension rejection;
    stop
  endif
  if (Future allocation conflict exists?) then (Yes)
    :Limit extension to an available time;
  endif
  |Pricing Service|
  :Calculate additional fee;
  |PBMS|
  if (Additional payment required?) then (Yes)
    if (Payment method?) then (Online)
      |Payment Gateway|
      :Process and return verified result;
      |PBMS|
      if (Payment successful?) then (No)
        |Driver or Staff|
        :Receive unsuccessful extension result;
        stop
      endif
    else (Cash)
      |PBMS|
      :Record cash settlement;
    endif
  endif
  :Update planned checkout time;
  |Driver or Staff|
  :Receive extension confirmation;
else (No)
  :Finish viewing session;
endif
stop
@enduml
```

## 6. Standard Checkout

**Traceability:** UC-OPS-002; UC-PRICE-002; UC-PAY-001; FR-OPS-004; FR-PRICE-002 to FR-PRICE-003; FR-PAY-001 to FR-PAY-002; BR-FEE-001 to BR-FEE-005; BR-PAY-001 to BR-PAY-002.

```plantuml
@startuml
title Standard Checkout - Balanced Detail
skinparam shadowing false
skinparam activity {
  BackgroundColor #F7F7F7
  BorderColor #454545
  DiamondBackgroundColor #FFF2CC
  DiamondBorderColor #8A6D1D
  StartColor #2E7D32
  EndColor #2E7D32
}

|Staff|
start
:Start checkout and identify session;
|Gate Camera|
:Capture exit evidence or allow manual plate entry;
|PBMS|
:Validate active session, vehicle, and card;
if (Mismatch or lost-card situation?) then (Yes)
  |Staff|
  :Continue with exceptional checkout;
  stop
endif

|Pricing Service|
:Select the applicable pricing policy;
:Calculate parking charge and applicable caps;
:Include unresolved incident penalties;
|PBMS|
:Create the current checkout payment;
if (Payment method?) then (Cash)
  :Apply cash settlement rules;
  :Record payment as paid;
else (Online)
  |Payment Gateway|
  :Process payment and return signed result;
  |PBMS|
  if (Verified payment successful?) then (No)
    |Staff|
    :Retry payment or handle unpaid exit;
    stop
  endif
  :Record payment once as paid;
endif

|PBMS|
:Store exit evidence;
:Complete parking session;
:Release slot and card;
:Resolve applicable open incidents;
|Staff|
:Allow vehicle exit;
stop
@enduml
```

## 7. Exceptional Checkout, Incidents, and Restrictions

**Traceability:** UC-OPS-002; UC-INC-002; FR-OPS-005; FR-INC-001; FR-CARD-001; FR-BLK-001.

```plantuml
@startuml
title Exceptional Checkout, Incidents, and Restrictions - Balanced Detail
skinparam WrapWidth 250
skinparam DefaultFontSize 40
skinparam TitleFontSize 60
skinparam TitleFontStyle bold
skinparam SwimlaneTitleFontSize 60
skinparam SwimlaneTitleFontStyle bold
skinparam ArrowFontSize 30
skinparam Nodesep 100
skinparam Ranksep 100
skinparam shadowing false
skinparam activity {
  BackgroundColor #F7F7F7
  BorderColor #454545
  DiamondBackgroundColor #FFF2CC
  DiamondBorderColor #8A6D1D
  StartColor #2E7D32
  EndColor #2E7D32
}

|Staff|
start
if (Exceptional operation?) then (Vehicle or card mismatch)
  :Capture evidence and description;
  |PBMS|
  :Create or update incident and penalty;
elseif (Unpaid exit)
  |PBMS|
  :Mark session unpaid and record incident;
  :Restrict affected vehicle or card;
  :Release operational card and slot;
elseif (Lost card)
  |Staff|
  :Confirm active session and lost card;
  |PBMS|
  :Mark old card lost;
  :Create incident, penalty, and restriction;
  if (Replacement card supplied?) then (Yes)
    :Validate replacement card availability;
    if (Replacement card eligible?) then (No)
      |Staff|
      :Receive replacement rejection;
      stop
    endif
    :Bind replacement card to session;
  endif
else (Checkout correction)
  |PBMS|
  if (Completed payment prevents safe reversal?) then (Yes)
    |Staff|
    :Receive correction rejection;
    stop
  endif
  :Restore only safely reversible states;
  :Retain payment history for audit;
endif

|Staff, Manager, or Admin|
:Review incident or restriction;
|PBMS|
if (Update or resolution authorized?) then (Yes)
  :Update incident, restrictions,\nand affected resources;
  |Staff, Manager, or Admin|
  :Receive recorded handling result;
else (No)
  |Staff, Manager, or Admin|
  :Receive authorization rejection;
endif
stop
@enduml
```

## 8. Monitoring, Revenue, and Audit

**Traceability:** UC-MON-001; FR-RPT-001; FR-AUD-001; NFR-OBS-001; BR-PAY-004.

```plantuml
@startuml
title Monitoring, Revenue, and Audit - Balanced Detail
skinparam shadowing false
skinparam activity {
  BackgroundColor #F7F7F7
  BorderColor #454545
  DiamondBackgroundColor #FFF2CC
  DiamondBorderColor #8A6D1D
  StartColor #2E7D32
  EndColor #2E7D32
}

|Staff, Manager, or Admin|
start
:Open operational or reporting view;
if (Information needed?) then (Current operations)
  :Request active sessions, bookings,\nslots, cards, or incidents;
  |PBMS|
  :Retrieve role-appropriate operational data;
elseif (Revenue report)
  |Staff, Manager, or Admin|
  :Select reporting criteria;
  |PBMS|
  :Aggregate paid payments by requested scope;
else (Audit history)
  |Staff, Manager, or Admin|
  :Request filtered audit records;
  |PBMS|
  if (Audit access authorized?) then (No)
    |Staff, Manager, or Admin|
    :Receive access rejection;
    stop
  endif
  :Retrieve available audit records;
endif

|PBMS|
:Prepare monitoring or report result;
|Staff, Manager, or Admin|
:View information and take follow-up action;

|Overtime Warning Worker|
:Review active sessions for warning conditions;
|PBMS|
:Record actionable warning outcome;
|Staff, Manager, or Admin|
:Receive current operational information;
stop
@enduml
```

---

For exact timing, percentages, formulas, configuration keys, state-transition constraints, and unresolved gaps, refer to the detailed business analysis and the SRS.
