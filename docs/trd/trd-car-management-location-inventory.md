# TRD - Car Management - Location & Inventory Management

## Document Information

- Feature Name: Car Management - Location & Inventory Management
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

This document defines technical requirements for the fleet onboarding and lifecycle and home locations and transfers functional requirements in [PRD - Car Management](../prd/prd-car-management.md), specifically sections 1 and 2. It covers maintaining company-owned vehicle inventory and each vehicle's current home location, preserving location transfer history, and recording transfer costs against the company pool.

## In Scope

- Create, view, and maintain inventory records for company-owned rental vehicles and their required attributes.
- Represent fleet locations and require every vehicle to have a current home location.
- Change a vehicle's lifecycle status using the statuses defined in the PRD.
- Transfer a vehicle to another home location while retaining its prior location and transfer history.
- Record any incurred transfer cost as a company-pool cost.
- Provide staff with inventory views filterable by location and lifecycle status.
- Preserve auditable attribution and timestamps for inventory, location, and transfer-cost changes.

## Constraints

- Only company-owned vehicles are covered; peer-to-peer inventory is excluded.
- Reservation allocation, availability calculations, maintenance scheduling, rental operations, GPS/geofencing, and utilization reporting are outside this TRD.
- No programming language, web framework, database product, API gateway, or external fleet system is mandated.
- No accounting or payment integration is required. Transfer costs must be recorded as company-pool costs; calculation, currency policy, and downstream posting are not specified by the PRD.
- Location address fields, status-transition rules, the precise set of required vehicle fields, and jurisdiction-specific identifier rules must be confirmed during product/design review. Do not invent validation rules for these unspecified items.
- Retirement and disposal are manually initiated. The system must not automatically retire vehicles based on age, mileage, or depreciation.
- Role-specific permissions and a detailed retention policy are not specified.

## Technical Requirements

### Database Design

Use the [Location & Inventory database design](database-design-car-management-location-inventory.md) for the logical entities and relationships. Persist a vehicle's current home location on its inventory record and append a location-history record for each completed transfer. Vehicle creation and initial-location history, and each transfer and its new current location, must be committed atomically so the current inventory state cannot disagree with its history.

### Frontend

- Provide fleet staff with searchable and filterable inventory by home location and lifecycle status. Display enough vehicle identifiers and descriptive attributes to distinguish inventory records.
- Provide an onboarding form for the vehicle attributes listed in the PRD, with the initial home location required.
- Provide a vehicle detail view showing its current home location, lifecycle status, captured vehicle and insurance details, and chronological location history.
- Provide a transfer form that identifies the source and destination locations and allows an incurred transfer cost to be recorded as charged to the company pool.
- Provide an explicit status-change action using only the lifecycle statuses in the PRD. Do not infer or automatically change a status from vehicle age, mileage, or depreciation.
- Validate required inputs before submission and display field-level errors beside the affected inputs. Preserve entered values when a validation or save operation fails.
- Clearly distinguish a transfer's current location from its historical locations and show transfer effective time and recorded cost, when present.

### Backend

Expose resource-oriented REST endpoints. The API must validate requests server-side; frontend validation is not authoritative. Identifiers in paths are opaque resource identifiers. Requests and responses use JSON unless otherwise negotiated.

#### API specification

| Method and URL | Parameters / request body | Successful response |
|---|---|---|
| `GET /locations` | Optional pagination parameters | `200` with location identifiers and display names |
| `POST /locations` | Required location display name; other address/location fields await product confirmation | `201` with the created location |
| `GET /vehicles` | Optional `location_id`, `status`, `vin`, `license_plate`, and pagination query parameters | `200` with matching vehicle records and pagination metadata |
| `POST /vehicles` | Vehicle attributes, company ownership, and required initial `home_location_id` | `201` with the created vehicle |
| `GET /vehicles/{vehicle_id}` | `vehicle_id` path parameter | `200` with vehicle details and location history |
| `PATCH /vehicles/{vehicle_id}` | `vehicle_id` path parameter; mutable vehicle descriptive, purchase, insurance, and odometer fields. Excludes ownership, lifecycle status, and home location. | `200` with the updated vehicle |
| `POST /vehicles/{vehicle_id}/status` | `vehicle_id` path parameter; body contains the requested `status` | `200` with the updated vehicle |
| `POST /vehicles/{vehicle_id}/transfers` | `vehicle_id` path parameter; body contains required `destination_location_id`, optional `effective_at`, and optional aggregate incurred `cost` with amount and currency | `201` with the transfer, updated vehicle location, and any cost record |

Use standard HTTP status semantics: `400` for malformed or invalid input, `401` for missing or invalid authentication, `403` for an authenticated caller without permission, `404` for a missing resource, `409` for conflicting or duplicate state, and `5xx` for unexpected service failures. Error responses must provide a stable machine-readable error identifier and a safe, human-readable message; field validation errors should identify the relevant field. Do not expose internal exception details.

#### Validation and consistency

- Enforce required vehicle attributes as established by product/design review and the enumerated PRD domains: size is small or medium; class is economy or luxury; fuel type is gas, electric, or hybrid; ownership is company; lifecycle status is incoming, active, maintenance, decommissioning, or sold.
- Require a valid existing home location on vehicle creation and a valid destination location on transfer. Reject a transfer to the vehicle's current location as a no-op conflict.
- Prevent duplicate vehicle identifiers according to confirmed identifier scope. VIN uniqueness should be enforced; license-plate uniqueness must account for the registration jurisdiction once that scope is confirmed.
- Reject unsupported lifecycle status values. Since the PRD does not define a transition matrix, do not reject a recognized status solely because it is a particular transition; apply authorization and any subsequently approved business rules.
- Treat omitted patch fields as unchanged. Do not allow patch operations to change the company-only ownership, lifecycle status, or home location; use the dedicated operations for status changes and transfers.
- Validate cost amounts as non-negative decimal amounts with at most two fractional digits and a precision sufficient for the database design. Require a currency code whenever a cost is supplied. Do not calculate costs or silently select a currency.
- Record every transfer cost with company-pool attribution. Do not accept a caller-supplied allocation target that could route this cost elsewhere.
- Use a transaction for onboarding and for transfers. On transfer, validate the destination and current vehicle state, append history, update the current home location, and record any supplied cost as one atomic operation. If any step fails, none of those changes may persist.
- Detect concurrent updates to the same vehicle and return `409` rather than silently overwriting a transfer or vehicle change. A conditional request mechanism such as an ETag may be used.
- Retain soft-deleted records and prevent deleting a location that is referenced as a current vehicle location or by retained transfer history. No public deletion endpoint is required by this TRD.

#### Operation sequences

Vehicle onboarding:

1. Authenticate and authorize the caller.
2. Validate the vehicle attributes, company ownership, unique VIN, and referenced initial location.
3. In one transaction, create the vehicle in the requested permitted initial lifecycle status and record its initial location assignment.
4. Commit and return the vehicle; on any validation or persistence error, return an appropriate error without partial creation.

Vehicle transfer:

1. Authenticate and authorize the caller.
2. Load the vehicle and destination location, and verify that the vehicle's current state has not changed since it was read.
3. Validate the destination and any supplied transfer cost and currency.
4. In one transaction, append a transfer history record, update the vehicle's current home location, and record any incurred cost with company-pool attribution.
5. Commit and return the updated vehicle and transfer record; on conflict or failure, preserve the pre-transfer state.

## Security Requirement

- Require authenticated access to all endpoints. Enforce authorization on the server for vehicle creation, updates, status changes, location transfers, and cost recording. Until a role matrix is defined, use the PRD actor expectations: fleet managers onboard vehicles and initiate transfers; authorized operational users may change lifecycle status.
- Deny access by default and do not infer permissions from client-supplied role or ownership fields.
- Enforce company-only ownership on the server; clients cannot create or change an inventory record to peer-owned.
- Validate and normalize all request fields before persistence. Use parameterized data access and avoid rendering user-supplied values as trusted markup.
- Protect API traffic using encrypted transport. Protect stored vehicle, insurance, cost, and audit data with access controls and encryption appropriate to the deployment.
- Record the authenticated actor and timestamps for creation and updates, including status changes and transfers. Do not accept audit actor identifiers from the request body.
- Return only data authorized for the caller and avoid leaking whether inaccessible records exist through detailed error messages.

## Non-Functional Requirements

## AI usage disclaimer

*This document was generated with the assistance of artificial intelligence and should be reviewed by a human for accuracy and completeness.*
