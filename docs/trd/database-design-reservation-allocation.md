# Database Design - Reservation Allocation

## Table of Contents

- [Entity Relationship Diagram](#entity-relationship-diagram)
- [vehicles](#vehicles)
- [Tables](#tables)
  - [reservations](#reservations)
  - [reservation_allocations](#reservation_allocations)
  - [vehicle_unavailability_blocks](#vehicle_unavailability_blocks)

## Entity Relationship Diagram

```mermaid
erDiagram
    locations ||--o{ vehicles : "current home"
    reservations ||--o{ reservation_allocations : has
    vehicles ||--o{ reservation_allocations : assigned
    vehicles ||--o{ vehicle_unavailability_blocks : blocked
```

`locations` is defined in the separate [Car Management Location & Inventory design](https://github.com/jjchase41/ai-sdlc-car-rental/pull/12). The `vehicles` table below is the consolidated vehicle master for onboarding, inventory, and reservation allocation. It resolves overlapping names from the [Vehicle Onboarding](https://github.com/jjchase41/ai-sdlc-car-rental/pull/10) and location/inventory designs to one field per concept: `vehicle_class`, `lifecycle_status`, and `odometer_km`.

## Tables

Each table includes the audit and soft-deletion fields required by the TRD database-design standard. `deleted` defaults to `false`; when a record is soft-deleted, set `deleted_at` and retain the record for audit and historical allocation checks.

### vehicles

| Field name | Data type | Index | Database constraints | Description |
|---|---|---|---|---|
| vehicle_id | UUID | Primary key | Not null | Stable, server-assigned vehicle identifier. |
| vin | TEXT | Unique among non-deleted records when provided | Nullable; unique when provided | Vehicle identification number. |
| license_plate | TEXT | Unique among non-deleted records within the confirmed registration scope | Nullable; uniqueness scope must be confirmed | Vehicle registration plate. |
| purchase_date | DATE | No | Nullable until required fields are confirmed | Date the company purchased the vehicle. |
| purchase_cost | DECIMAL(15,2) | No | Nullable; non-negative when provided | Company purchase cost. |
| insurance_details | TEXT | No | Nullable until required fields are confirmed | Insurance details captured for the vehicle. |
| insurance_expires_on | DATE | No | Nullable until required fields are confirmed | Insurance expiration date. |
| odometer_km | DECIMAL | No | Nullable; non-negative when provided | Current recorded odometer reading in kilometers. |
| brand | TEXT | No | Nullable until required fields are confirmed | Vehicle brand. |
| model | TEXT | No | Nullable until required fields are confirmed | Vehicle model. |
| manufacturing_year | INTEGER | No | Nullable; positive when provided | Vehicle manufacturing year. |
| size | TEXT | No | Nullable until required fields are confirmed; when provided, one of `small`, `medium` | Vehicle size category. |
| vehicle_class | TEXT | No | Nullable until required fields are confirmed; when provided, one of `economy`, `luxury` | Rental vehicle class; canonical name for the field also called `class`. |
| seat_count | INTEGER | No | Nullable; positive when provided | Number of seats. |
| fuel_type | TEXT | No | Nullable until required fields are confirmed; when provided, one of `gas`, `electric`, `hybrid` | Vehicle fuel type. |
| ownership | TEXT | No | Not null; fixed to `company` and set by the service | Ownership category; only company-owned vehicles are in scope. |
| lifecycle_status | TEXT | Yes | Not null; default `incoming`; one of `incoming`, `active`, `maintenance`, `decommissioning`, `sold` | Current vehicle lifecycle status; canonical name for the field also called `status`. |
| home_location_id | UUID | Yes; foreign key to `locations.location_id` | Not null; references a non-deleted location | Vehicle's current home location. |
| created_at | TIMESTAMP WITH TIME ZONE | No | Not null | Record creation time. |
| updated_at | TIMESTAMP WITH TIME ZONE | No | Not null | Most recent record update time. |
| deleted_at | TIMESTAMP WITH TIME ZONE | No | Nullable | Time the record was soft-deleted, if applicable. |
| created_by | TEXT | No | Not null | Actor that created the record. |
| updated_by | TEXT | No | Not null | Actor that last updated the record. |
| deleted | BOOL | Yes | Not null; default `false` | Soft-deletion flag. |

The PRD leaves the minimum required vehicle fields for confirmation after UI design. Until that decision is made, keep the captured vehicle attributes nullable as indicated; ownership, lifecycle status, home location, identifiers, and audit fields retain the constraints shown above. A vehicle lifecycle status must use the same five-value set in the current record and any status-history record.

### reservations

| Field name | Data type | Index | Database constraints | Description |
|---|---|---|---|---|
| id | UUID | Primary key | Not null | Reservation identifier. |
| customer_id | UUID | Yes | Not null; references the customer identity maintained by the reservation capability | Customer reference; customer details are not duplicated here. |
| requested_start_at | TIMESTAMP WITH TIME ZONE | Yes, with `requested_end_at` | Not null | Start instant of the requested rental period. |
| requested_end_at | TIMESTAMP WITH TIME ZONE | Yes, with `requested_start_at` | Not null; later than `requested_start_at` | End instant of the requested rental period. |
| created_at | TIMESTAMP WITH TIME ZONE | Yes | Not null | Original reservation receipt time; source for first-come, first-served ordering and tie-breaking input. |
| updated_at | TIMESTAMP WITH TIME ZONE | No | Not null | Last update time. |
| deleted_at | TIMESTAMP WITH TIME ZONE | No | Nullable | Soft-deletion time; null unless deleted. |
| created_by | TEXT | No | Not null | Actor that created the reservation record. |
| updated_by | TEXT | No | Not null | Actor that last updated the reservation record. |
| deleted | BOOL | Yes | Not null; default `false` | Soft-deletion flag. |

### reservation_allocations

| Field name | Data type | Index | Database constraints | Description |
|---|---|---|---|---|
| id | UUID | Primary key | Not null | Allocation record identifier. |
| reservation_id | UUID | Yes | Not null; references `reservations.id` | Reservation receiving the vehicle. |
| vehicle_id | UUID | Yes, with `allocation_status` | Not null; references `vehicles.vehicle_id` | Vehicle assigned to the reservation. |
| allocation_status | TEXT | Yes, with `vehicle_id` | Not null; allowed values: `active`, `released` | Current or historical allocation state. |
| allocated_at | TIMESTAMP WITH TIME ZONE | Yes | Not null | Time the allocation was committed. |
| released_at | TIMESTAMP WITH TIME ZONE | No | Nullable; required when status is `released` | Time the allocation ceased to apply. |
| created_at | TIMESTAMP WITH TIME ZONE | No | Not null | Record creation time. |
| updated_at | TIMESTAMP WITH TIME ZONE | No | Not null | Last update time. |
| deleted_at | TIMESTAMP WITH TIME ZONE | No | Nullable | Soft-deletion time; null unless deleted. |
| created_by | TEXT | No | Not null | Actor that created the allocation record. |
| updated_by | TEXT | No | Not null | Actor that last updated the allocation record. |
| deleted | BOOL | Yes | Not null; default `false` | Soft-deletion flag. |

Maintain at most one active allocation per reservation. For any vehicle, active allocations must not overlap the associated reservation periods. Enforce both rules atomically during allocation; the overlap rule is a logical invariant independent of database-specific range-constraint support. Keep released records for allocation history.

### vehicle_unavailability_blocks

| Field name | Data type | Index | Database constraints | Description |
|---|---|---|---|---|
| id | UUID | Primary key | Not null | Unavailability-block identifier. |
| vehicle_id | UUID | Yes, with `block_status`, `starts_at`, and `ends_at` | Not null; references `vehicles.vehicle_id` | Vehicle made unavailable for a period. |
| block_type | TEXT | Yes | Not null; allowed values: `maintenance`, `turnaround` | Source category of the unavailability. |
| block_status | TEXT | Yes, with `vehicle_id` | Not null; allowed values: `active`, `cancelled` | Whether the block currently affects availability. |
| starts_at | TIMESTAMP WITH TIME ZONE | Yes, with `vehicle_id` and `ends_at` | Not null; earlier than `ends_at` | Start instant of the block. |
| ends_at | TIMESTAMP WITH TIME ZONE | Yes, with `vehicle_id` and `starts_at` | Not null; later than `starts_at` | End instant of the block. |
| source_reference | TEXT | No | Nullable | Reference to the originating maintenance schedule or return event. |
| created_at | TIMESTAMP WITH TIME ZONE | No | Not null | Record creation time. |
| updated_at | TIMESTAMP WITH TIME ZONE | No | Not null | Last update time. |
| deleted_at | TIMESTAMP WITH TIME ZONE | No | Nullable | Soft-deletion time; null unless deleted. |
| created_by | TEXT | No | Not null | Actor that created the block. |
| updated_by | TEXT | No | Not null | Actor that last updated the block. |
| deleted | BOOL | Yes | Not null; default `false` | Soft-deletion flag. |

An active block overlapping any part of the requested rental period makes the vehicle unavailable. Represent a scheduled maintenance day using the applicable operating location's local-day boundaries, converted to instants. Represent turnaround from the recorded return through one local calendar day after the return; this accommodates daylight-saving changes without changing the business duration.
