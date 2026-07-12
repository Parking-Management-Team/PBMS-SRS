# PBMS Feature Structure (SRS-Aligned)

## HOME

---

# Authentication

- Login
- Register
- Forgot Password
- Unauthorized / Error Page

---

# Public Information

- View pricing and policies
- View supported vehicle types
- View building parking information
- Submit support request or feedback

---

# Driver / User Portal

## User Dashboard

- View parking activity overview
- View active booking status
- View active monthly subscription status
- View payment status and pending payments

## Account & Vehicle Management

- Manage personal profile
- Add vehicle
- Update vehicle information
- View registered vehicles
- Deactivate vehicle

## Parking Booking

- View parking availability by building
- Select vehicle or enter license plate
- Select planned check-in and check-out time
- Create advance booking
- Pay booking deposit
- View booking payment deadline
- Confirm booking after successful deposit payment
- Cancel booking before allowed cancellation deadline
- View booking history

## Monthly Subscription

- Register monthly subscription for one vehicle
- Select applicable building
- Pay monthly subscription fee
- View assigned monthly card
- View assigned monthly slot for car subscription
- Renew monthly subscription
- View monthly subscription history

## Parking Session

- View active parking session
- View ticket or simulated card code
- Track temporary fee
- View assigned zone for motorcycle session
- View assigned slot for car session
- View parking history

## Payment

- View payment summary
- Pay online through banking
- View cash payment status
- View booking deposit payments
- View monthly subscription payments
- View payment history
- View refunded payments

## Feedback & Incident Report

- Report lost ticket or card code
- Report wrong fee
- Report occupied slot
- Report wrong license plate
- Submit general feedback

---

# Staff Portal

## Staff Dashboard

- View operational overview
- View active parking sessions
- View pending check-out payments
- View active incidents

## Vehicle Check-in

- Enter license plate manually
- Capture check-in vehicle image (Base64 string) via camera/webcam
- Run automatic license plate recognition (LPR) using external Plate Recognizer API
- Validate vehicle and account information
- Validate booking status and check-in grace time
- Validate monthly subscription status
- Validate blacklist status
- Assign normal card for walk-in or booking session
- Validate monthly card for monthly subscription session
- Allocate motorcycle to available zone
- Allocate walk-in or booking car to `GENERAL` zone slot
- Allocate monthly car to assigned `MONTHLY` zone slot
- Generate ticket or simulated card code
- Create parking session

## Vehicle Check-out

- Search active parking session
- Capture check-out vehicle image (Base64 string) via camera/webcam
- Verify license plate and card code
- Calculate parking fee using rule-based Pricing Engine (BasePricing, IncrementPricing, DailyCap, GracePeriod)
- Apply cash rounding rules
- Receive cash payment
- Verify online payment (VNPay integration)
- Complete check-out
- Release occupied zone capacity or slot

## Slot & Zone Monitoring

- View parking map by building and floor
- View zone capacity
- View slot status
- Update zone operational status
- Update slot operational status
- Mark slot as maintenance
- Block slot from operation
- Relocate vehicle when allowed

## Card Operations

- Search card by card code
- Assign normal card to parking session
- Validate monthly card
- Mark card as lost
- Mark card as active or inactive

## Incident Handling

- Handle lost ticket or simulated card
- Handle wrong license plate
- Handle overtime parking
- Handle wrong area parking
- Handle unpaid vehicle
- Create incident record
- Update incident status
- Create blacklist record for vehicle or card when required

## Shift Report

- View daily activity
- View payment summary
- View incident summary
- View check-in and check-out count
- Submit shift report with cash reconciliation (expected vs actual, difference, note)
- Track shift report submission status (Submitted, Approved, Rejected)

---

# Manager Portal

## Manager Dashboard

- View parking operation overview
- View capacity utilization
- View revenue overview
- View incident overview

## Facility Management

- Manage building
- Manage floor
- Manage zone
- Configure zone access type
- Configure vehicle allocation rules
- Configure motorcycle capacity by zone

## Slot Management

- Manage parking slots
- Control slot status
- Manage car slots in `GENERAL` zones
- Manage monthly car slots in `MONTHLY` zones
- Manage maintenance slots
- Manage blocked slots

## Vehicle Type Management

- Manage vehicle types
- Activate vehicle type
- Deactivate vehicle type

## Monthly Subscription Management

- View monthly subscription requests
- Approve monthly subscription
- Assign monthly card
- Assign monthly car slot
- Activate monthly subscription
- Renew monthly subscription
- Suspend or cancel monthly subscription
- Track monthly subscription capacity usage

## Pricing Management

- Configure pricing policies by vehicle type
- Configure pricing windows
- Configure base duration and base price
- Configure increment block and increment price
- Configure window cap
- Configure grace period
- Configure penalty policies
- Configure cash rounding rule

## Payment & Revenue Management

- View payment transactions
- Trace payment source by parking session, booking, or monthly subscription
- View cash and online banking revenue
- View refunded payments
- View revenue statistics
- Trace aggregated revenue payments

## Shift Report Approval

- View staff shift reports
- Review cash reconciliation details and notes
- Approve or reject shift reports (updating status to Approved or Rejected)

## Blacklist Management

- View blacklist records
- Add vehicle to blacklist
- Add card to blacklist
- Link blacklist record to incident
- Remove blacklist record when allowed

## Analytics & Reports

- View parking utilization
- View vehicle traffic
- View revenue analytics
- View peak-hour analysis
- View incident statistics
- View monthly subscription statistics

---

# Administrator Portal

## Admin Dashboard

- View administration overview

## User Management

- Create account
- Update account
- Deactivate account
- Reset password
- Manage driver accounts without registered vehicles

## Role & Permission

- Assign role
- Manage permission matrix
- Configure access control

## System Configuration

- Configure notifications
- Configure payment settings
- Configure booking timeout
- Configure booking grace time
- Configure cancellation and no-show policies
- Configure security settings

## System Monitoring

- View audit logs
- View error logs
- View activity logs
- Monitor system health
