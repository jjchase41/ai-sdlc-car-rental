# PRD - Car Management

## Document Information

- Product / Feature Name: Car Management
- Author: @copilot
- Date:
- Version:

## Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Functional Requirements](#functional-requirements)
- [Non-Functional Requirements](#non-functional-requirements)
- [Dependency & Constraints](#dependency--constraints)
- [Success Metrics](#success-metrics)

## Overview

### Background

The company is expanding from car sales into car rental and has no dedicated rental fleet systems or prior rental operations. The Service, Delivery, and Pickup team needs a shared way to manage vehicles throughout their rental lifecycle and coordinate handovers, returns, maintenance, and availability.

### Objective

Define the product requirements for managing a company-owned rental fleet, from onboarding and location assignment through rental operations, maintenance, and retirement.

### Goals

- Maintain accurate vehicle records, lifecycle statuses, and home-location history.
- Prevent double bookings and prevent vehicles that are unsafe, under maintenance, or improperly insured from being rented.
- Support reliable pickup, delivery, return, damage, and fuel-charge workflows.
- Give staff timely fleet availability, location, maintenance, and utilization information.

## Problem Statement

Without a rental fleet management system, staff cannot reliably determine which vehicles are available, where they belong, or whether they are safe and ready to rent. Vehicle assignments, handovers, returns, maintenance, charges, and customer evidence must be coordinated without dedicated systems, creating operational risk and the potential for double bookings or incomplete customer charges.

The primary users are fleet managers and the Service, Delivery, and Pickup crew responsible for vehicle records, allocation, handover, return inspections, and maintenance coordination. Customers are affected by vehicle availability, pickup and delivery, substitutions, and return charges. This is important now because the company is starting a new rental business line and needs consistent processes from go-live.

## Functional Requirements

### 1. Fleet onboarding and lifecycle

**Statement:** As a fleet manager, I want to register company-owned rental vehicles and manage their lifecycle, so that the fleet can be accurately operated and retired.

**Requirement detail:** A vehicle record captures VIN, license plate, purchase date and cost, insurance details and expiration, odometer, brand, model, manufacturing year, size (small or medium), class (economy or luxury), seat count, fuel type (gas, electric, or hybrid), and ownership. Only company-owned vehicles are in scope. Support incoming, active, maintenance, decommissioning, and sold statuses. Retirement and disposal are initiated manually.

**Acceptance criteria:**

- **Given** a fleet manager is onboarding a vehicle, **when** they save its record, **then** the system stores the vehicle and its captured details with company ownership.
- **Given** a vehicle is in the fleet, **when** an authorized operational user changes its lifecycle status, **then** the new status is reflected in fleet records and availability.
- **Given** a vehicle is to be retired or disposed of, **when** a fleet manager initiates the change, **then** the vehicle is not automatically retired by an age, mileage, or depreciation rule.

### 2. Home locations and transfers

**Statement:** As a fleet manager, I want each vehicle to have a home location and a history of location changes, so that fleet inventory and transfer costs are traceable.

**Requirement detail:** Every vehicle belongs to a particular home location. A transfer changes the vehicle's home location and retains its prior location history. Costs incurred for transfers are charged to the company pool.

**Acceptance criteria:**

- **Given** a vehicle has a home location, **when** a fleet manager transfers it, **then** the new home location is recorded and the previous location remains in its history.
- **Given** a transfer incurs a cost, **when** that cost is recorded, **then** it is charged to the company pool.

### 3. Availability and reservation allocation

**Statement:** As a dispatcher, I want the system to show vehicle availability and automatically allocate eligible vehicles in first-come, first-served order, so that reservations are fulfilled without double booking.

**Requirement detail:** Show real-time availability through two hours after the current time and planned availability from H+1 through H+30. Allocation is automatic, first come first served, with no model or class prioritization and no overbooking. Do not allocate a vehicle that is not active, has a maintenance block, is in its one-day post-return turnaround, or is within one week of insurance expiration. The business targets zero double bookings.

**Acceptance criteria:**

- **Given** an eligible vehicle is available for a requested period, **when** a reservation is allocated, **then** the system assigns an eligible vehicle automatically according to first-come, first-served order.
- **Given** a vehicle is already allocated for any part of the requested period, **when** another allocation is attempted, **then** the system does not double-book that vehicle or overbook the fleet.
- **Given** a vehicle is inactive, under maintenance, in post-return turnaround, or within one week of insurance expiration, **when** allocation is evaluated, **then** that vehicle is excluded.
- **Given** staff view fleet availability, **when** they select a period, **then** the view reflects the near-term real-time and planned availability horizons.

### 4. Maintenance scheduling and vehicle readiness

**Statement:** As a fleet manager, I want to schedule mileage-based maintenance and block affected vehicles, so that maintenance takes precedence over rentals and vehicles are not released while unavailable.

**Requirement detail:** The crew schedules routine maintenance at 10,000–12,000 km intervals, using 10,000 km multiples as the stated scheduling basis. A maintenance scheduled for a day makes the vehicle unavailable for that day; maintenance blocks automatically remove availability. Following an incident, classify a vehicle as drivable or in maintenance; a vehicle in maintenance is unavailable for rent. Vehicles with critical faults must be marked unavailable for rent. Alert staff when maintenance is overdue. Predictive maintenance and diagnostic-code triggers are out of scope.

**Acceptance criteria:**

- **Given** a vehicle reaches its maintenance interval, **when** the crew schedules maintenance for a date, **then** the vehicle is unavailable for rent for that day.
- **Given** maintenance is scheduled, **when** availability is calculated, **then** the maintenance block is applied automatically.
- **Given** a vehicle has an active critical fault or overdue maintenance, **when** staff review fleet readiness, **then** the vehicle is unavailable for rent and overdue maintenance is alerted.
- **Given** a vehicle is assessed after an incident, **when** its post-incident status is set, **then** it is classified as drivable or in maintenance, and a vehicle in maintenance is unavailable.

### 5. Pickup, delivery, and handover

**Statement:** As a pickup or delivery crew member, I want to complete customer verification and record handover evidence, so that each vehicle release is documented and verified.

**Requirement detail:** Drop-off and pickup service is available every day from 06:00 to 19:00 local time and incurs an additional cost. At pickup, verify the driver's license and match a selfie against a national identifier or passport. Capture handover photos and an electronic form with signature, timestamp, and geolocation. Physical keys or smart locks may be used depending on the vehicle.

**Acceptance criteria:**

- **Given** a customer requests a pickup or delivery, **when** the service is scheduled, **then** its time is within 06:00–19:00 local time on any day and its additional cost is accounted for.
- **Given** a vehicle is being handed over, **when** the crew completes the handover, **then** the driver's license and selfie identity check are recorded and photos plus the signed electronic form, timestamp, and geolocation are captured.
- **Given** a handover has a late, fuel, or damage charge due, **when** the handover process is completed, **then** the charge is presented for settlement.

### 6. Returns, inspections, and customer charges

**Statement:** As a return crew member, I want to inspect a returned vehicle and record fuel and visible damage, so that its condition is documented and applicable customer charges can be settled.

**Requirement detail:** Return inspections capture photos, fuel level, and a damage checklist covering exterior, interior, engine, and electrical instruments. Record damage through a web form for the vehicle and support positional mapping of damage photos on a vehicle diagram. Chargeable damage is visible accident damage, such as a body dent or torn leather seat; engine and electrical instrument issues are treated as normal wear. Assess repair costs using an internal price list; do not auto-generate damage cases or severity scores. Returned fuel must match the fuel level at delivery or the customer is charged. Damage assessment and any applicable charge should be completed and settled with the customer on the spot. After-hours key drops are not supported.

**Acceptance criteria:**

- **Given** a vehicle is returned during staffed operating hours, **when** the crew completes its inspection, **then** photos, fuel level, and all four checklist areas are recorded.
- **Given** visible accident damage is found, **when** the crew records it, **then** the crew can attach photos mapped to the vehicle diagram and assess the repair amount using the internal price list.
- **Given** an engine or electrical instrument issue is observed, **when** it is assessed under this process, **then** it is treated as normal wear rather than chargeable damage.
- **Given** the returned fuel level differs from its delivery level, **when** the return is assessed, **then** the applicable fuel charge is calculated or recorded for settlement.
- **Given** a chargeable damage or fuel amount is identified, **when** the return inspection is completed, **then** the amount is presented for customer settlement.
- **Given** a customer attempts an after-hours return, **when** the return is arranged, **then** the system does not offer a key-drop process and supports staffed pickup/drop-off only from 06:00 to 19:00 local time.

### 7. Turnaround and crew task assignment

**Statement:** As an operations dispatcher, I want returned vehicles to have turnaround time and crew tasks scheduled, so that vehicles are prepared before being offered again.

**Requirement detail:** A vehicle has one day of turnaround after return. Cleaning-level tracking and turnaround SLA alerts are not required. Staff tasks are manually assigned on the return day, except vehicles returned after 15:00 local time are assigned on the next day. No mobile task list, completion scanning, or productivity metrics are required.

**Acceptance criteria:**

- **Given** a vehicle has been returned, **when** availability is calculated, **then** the vehicle remains unavailable during its one-day turnaround.
- **Given** a vehicle returns by 15:00 local time, **when** the dispatcher assigns follow-up work, **then** tasks may be assigned that day.
- **Given** a vehicle returns after 15:00 local time, **when** the dispatcher assigns follow-up work, **then** staff are assigned on the next day.

### 8. GPS, geofencing, and alerts

**Statement:** As a fleet manager, I want timely vehicle GPS information and geofence alerts, so that unauthorized vehicle movement can be acted on.

**Requirement detail:** GPS is the only required live telemetry. Updates must be no more than five minutes apart, with real-time updates preferred where possible. Alert staff when a vehicle leaves its permitted geofence and when maintenance is overdue. Deliver alerts through email, SMS, and dashboard.

**Acceptance criteria:**

- **Given** GPS telemetry is available, **when** vehicle location is refreshed, **then** the update interval is at most five minutes, with real-time updates preferred.
- **Given** a vehicle leaves its permitted geofence, **when** the event is detected, **then** an alert is delivered through the configured email, SMS, and dashboard channels.
- **Given** vehicle maintenance is overdue, **when** the overdue condition is detected, **then** staff receive an alert through the configured channels.

### 9. Utilization reporting and idle vehicles

**Statement:** As a fleet manager, I want daily utilization and idle reporting by location and across the fleet, so that I can monitor fleet use and prolonged idle periods.

**Requirement detail:** Report daily vehicle usage and idle time in both per-location and global views. Identify vehicles with five consecutive idle days.

**Acceptance criteria:**

- **Given** daily fleet activity is available, **when** a fleet manager opens utilization reporting, **then** usage and idle information is available by location and globally.
- **Given** a vehicle has been idle for five consecutive days, **when** the daily reporting period is processed, **then** the vehicle is identified as meeting the prolonged-idle threshold.

### 10. Substitution and reservation conflicts

**Statement:** As a dispatcher, I want a defined fallback when an allocated vehicle becomes unavailable, so that the customer receives an appropriate resolution.

**Requirement detail:** Offer a replacement in this order: another vehicle of the same type, a better vehicle, then a refund. Do not prioritize exact-model promises or automatically offer compensation. Warn staff when an extension conflicts with an upcoming reservation.

**Acceptance criteria:**

- **Given** an allocated vehicle becomes unavailable, **when** the reservation is resolved, **then** staff follow the substitution order of same type, better vehicle, and refund.
- **Given** an extension request conflicts with an upcoming reservation, **when** the request is evaluated, **then** staff receive a warning before the extension is confirmed.

### 11. Extensions and early returns

**Statement:** As a customer, I want extensions to be approved when requested and early returns to release the vehicle for reassignment, so that rental timing changes can be handled without unnecessary delay.

**Requirement detail:** Automatically approve extension requests. Charge the base rental cost plus a daily late charge. Warn about conflicts with upcoming reservations. When a vehicle is returned early, reposition it and make it available for a new customer subject to its normal readiness, maintenance, insurance, and turnaround rules.

**Acceptance criteria:**

- **Given** a customer requests an extension, **when** the request is processed, **then** it is automatically approved and the base rental cost plus daily late charge is applied.
- **Given** the extension conflicts with an upcoming reservation, **when** it is processed, **then** staff are warned about the conflict.
- **Given** a vehicle is returned early, **when** its readiness requirements are met, **then** it can be repositioned and made available to a new customer.

### 12. Key loss

**Statement:** As a fleet manager, I want lost physical keys to be chargeable, so that key replacement costs can be recovered.

**Requirement detail:** Charge the customer for a lost key. Key changes and replacement remain a manual process. Key custody tracking is not required.

**Acceptance criteria:**

- **Given** a physical key is reported lost, **when** the loss is recorded, **then** the applicable customer charge can be recorded and key replacement remains a manual task.

## Non-Functional Requirements

## Dependency & Constraints

- This PRD covers company-owned vehicles only; peer-to-peer vehicle inclusion is out of scope.
- No DMS, maintenance platform, telematics provider, repair-shop, insurance-adjuster, charging-station, or external fleet integration is currently required.
- GPS telemetry requires a vehicle tracking capability, but no provider has been selected.
- Fleet record minimum required fields will be decided after UI design.
- The availability horizon wording (“real time, on two hours after current time” and planned H+1 to H+30) needs clarification during design, including the intended boundary and overlap.
- Maintenance cadence is described as every 10,000–12,000 km and also as a schedule on 10,000 km multiples; the operating interval must be confirmed.
- Rental base rates, daily late charges, transfer accounting details, fuel charge rates, and the internal damage price list require business configuration; amounts are not specified here.
- Local time means the applicable operating location's local time. No after-hours key drop, automated route planning, cleaning-level tracking, predictive maintenance, damage AI, automatic damage severity scoring, task productivity reporting, or automated repositioning recommendation is in scope.
- Regulatory inspection scheduling, role-specific permissions, stolen-vehicle workflow, wrong-location return resolution, and draft inspection support are not specified for go-live.

## Success Metrics

- Zero double-booking incidents.
- Vehicle status and GPS updates are no more than five minutes apart.
- 100% of handovers have the required identity verification and handover evidence recorded.
- 100% of completed returns have the required inspection checklist, photos, and fuel level recorded.
- Vehicles with maintenance blocks, critical faults, or insurance expiring within one week are not allocated.
- Daily usage and idle reporting is available both per location and globally, with vehicles idle for five consecutive days identified.
