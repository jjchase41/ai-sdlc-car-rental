# TRD - Vehicle Onboarding

## Document Information

- Feature Name: Vehicle Onboarding
- Author: @copilot
- Date:
- Version:

## Table of Contents

- [Background](#background)
- [In Scope](#in-scope)
- [Constraints](#constraints)
- [Technical Requirements](#technical-requirements)
  - [Database Design](#database-design)
  - [Frontend](#frontend)
  - [Backend](#backend)
- [Security Requirement](#security-requirement)
- [Non-Functional Requirements](#non-functional-requirements)
- [AI usage disclaimer](#ai-usage-disclaimer)

## Background

This document defines technical requirements for the “Fleet onboarding and lifecycle” functional requirement in the [Car Management PRD](../prd/prd-car-management.md#1-fleet-onboarding-and-lifecycle). It covers recording company-owned vehicles, capturing their specified details, and maintaining their lifecycle status.

## In Scope

- Create and maintain vehicle records for company-owned rental vehicles.
- Capture VIN, license plate, purchase date and cost, insurance details and expiration, odometer, brand, model, manufacturing year, size, class, seat count, fuel type, ownership, and the vehicle's required home-location reference.
- Support the incoming, active, maintenance, decommissioning, and sold lifecycle statuses.
- Record lifecycle status changes with an audit history.
- Provide vehicle record and status operations through a REST API.
- Validate submitted data and restrict record operations to authorized users.

## Constraints

- Peer-to-peer vehicles are out of scope; ownership is company-owned.
- Vehicle minimum-required fields are subject to confirmation after UI design. The fields listed in the PRD are the capture scope; field-level requiredness must be configurable or finalized before implementation.
- No automatic retirement based on age, mileage, or depreciation is allowed. Retirement and disposal are initiated manually.
- Reservation allocation, availability calculations, home-location assignment and transfers, maintenance scheduling, insurance-expiry allocation rules, and other rental operations are outside this TRD. The shared vehicle record still carries the required current home-location reference.
- No database vendor, application framework, identity provider, or deployment platform is prescribed.

## Technical Requirements

### Database Design

Use the conceptual tables in [Database Design - Vehicle Onboarding](database-design-vehicle-onboarding.md):

- `vehicles`
- `vehicle_status_histories`

### Frontend

- Provide a vehicle onboarding form for all vehicle attributes listed in the PRD.
- Present size, class, and fuel type as selections constrained to the supported values. Set the initial lifecycle status to `incoming`; provide status changes through the separate authorized status flow. Require a home location when creating a vehicle.
- Set ownership to company-owned; do not allow the user to register a non-company-owned vehicle.
- Validate entered values before submission and present field-specific validation errors inline. Server validation remains authoritative.
- After successful creation, show the persisted vehicle record and its initial status. Do not imply that an incoming vehicle is active or available for rental.
- Provide an authorized lifecycle-status update flow and show the result only after the server confirms the change.

### Backend

#### REST API

All endpoints require authentication and return a consistent error response containing an error code, a human-readable message, and field-level details when applicable. Vehicle identifiers are server-assigned and opaque to clients.

| Method and URL | Purpose | Request | Successful response |
|---|---|---|---|
| `POST /api/v1/vehicles` | Create a vehicle record. | Vehicle attributes listed below, including `home_location_id`, excluding server-managed identifiers, audit fields, and ownership. | `201 Created`; the persisted vehicle record, including its identifier, ownership, home location, and initial lifecycle status. |
| `GET /api/v1/vehicles/{vehicleId}` | Retrieve a vehicle record. | `vehicleId` path parameter. | `200 OK`; the vehicle record, current lifecycle status, and home location. |
| `PATCH /api/v1/vehicles/{vehicleId}` | Update editable vehicle details. | `vehicleId` path parameter and one or more editable vehicle attributes. Excludes ownership, lifecycle status, and home location. | `200 OK`; the updated vehicle record. |
| `PATCH /api/v1/vehicles/{vehicleId}/status` | Change lifecycle status. | `vehicleId` path parameter and the requested `lifecycle_status`. | `200 OK`; the updated lifecycle status and status-change record. |

The vehicle request fields are `vin`, `license_plate`, `purchase_date`, `purchase_cost`, `insurance_details`, `insurance_expires_on`, `odometer_km`, `brand`, `model`, `manufacturing_year`, `size`, `vehicle_class`, `seat_count`, `fuel_type`, and, on creation, `home_location_id`. These correspond to the vehicle attributes in the PRD; `vehicle_class` represents its “class” attribute. Ownership is set by the service to company-owned. `lifecycle_status` is server-set to `incoming` on creation and cannot be supplied in the create request. The approved field-requirements matrix, to be finalized after UI design, determines which attributes must be provided at onboarding. A valid home location is required at creation.

#### Validation and behavior

- Require the supported values for size (`small`, `medium`), class (`economy`, `luxury`), fuel type (`gas`, `electric`, `hybrid`), and `lifecycle_status` (`incoming`, `active`, `maintenance`, `decommissioning`, `sold`). Store this current status once as `lifecycle_status`; status-history values use the same domain.
- Validate purchase date and insurance expiration as dates; validate purchase cost as a non-negative monetary amount with two decimal places; validate odometer in kilometers as a non-negative whole-number value; validate manufacturing year and seat count as positive whole numbers. Apply these checks when the corresponding values are provided.
- Require attributes only as specified by the approved field-requirements matrix; do not make additional attributes mandatory before that matrix is finalized. Trim surrounding whitespace before validation and persistence.
- Prevent duplicate non-empty VIN values among non-deleted vehicle records, treating VINs case-insensitively. Do not impose global license-plate uniqueness until registration jurisdiction is defined; enforce uniqueness within the confirmed scope.
- Require `home_location_id` to reference an existing, non-deleted location. The location entity is part of the shared fleet inventory design; location assignment and transfer workflows remain outside this TRD.
- Reject unsupported fields, invalid enum values, malformed identifiers, and invalid field combinations without partially persisting the request.
- Apply updates atomically. Do not permit clients to change ownership, vehicle identifier, audit fields, or deletion metadata through the vehicle details endpoint.
- On a status change, persist the vehicle’s new status and a corresponding history record atomically. Do not create a history record when the requested status is already current.
- Do not infer or automate retirement, disposal, or status changes from age, mileage, or depreciation.
- Return `400 Bad Request` for invalid input, `401 Unauthorized` for unauthenticated requests, `403 Forbidden` for authenticated users without the required permission, and `404 Not Found` for an unknown or deleted vehicle or location. Return `409 Conflict` for duplicate VIN or a license-plate conflict within the confirmed uniqueness scope.

#### Authorization and audit

- A fleet manager may create and edit vehicle details and initiate lifecycle changes. An operational user may change lifecycle status only when granted that permission.
- Record the authenticated actor and timestamps for record creation, updates, and every lifecycle status change.

## Security Requirement

- Require authenticated access for all vehicle endpoints and enforce authorization on the server for every operation.
- Restrict vehicle creation and detail updates to fleet managers. Restrict status changes to users authorized for lifecycle operations; do not assume that every authenticated user has these permissions.
- Validate and normalize all client-supplied values on the server. Use parameterized data access so user input is treated as data, not executable query content.
- Protect vehicle and insurance information in transit and at rest using the deployment environment’s approved security controls. Do not include insurance details or other vehicle data in errors or application logs unless explicitly required and access-controlled.
- Maintain an audit trail for record creation, changes, and status transitions; audit entries must identify the actor and time and must not be editable through these APIs.

## Non-Functional Requirements

## AI usage disclaimer

*This document was generated with the assistance of artificial intelligence and should be reviewed by a human for accuracy and completeness.*
