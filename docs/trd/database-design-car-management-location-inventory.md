# Database Design - Car Management - Location & Inventory Management

## Table of Contents

- [Entity Relationship Diagram](#entity-relationship-diagram)
- [locations](#locations)
- [vehicles](#vehicles)
- [vehicle_location_history](#vehicle_location_history)
- [transfer_costs](#transfer_costs)
- [Common Record and Deletion Rules](#common-record-and-deletion-rules)

## Entity Relationship Diagram

```mermaid
erDiagram
    locations ||--o{ vehicles : "current home"
    vehicles ||--o{ vehicle_location_history : "has"
    locations ||--o{ vehicle_location_history : "source or destination"
    vehicle_location_history ||--o| transfer_costs : "incurs"
```

## locations

| Field name | Data type | Index | Database constraints | Description |
|---|---|---|---|---|
| location_id | UUID | Primary key | Not null | Stable identifier for a fleet location. |
| display_name | TEXT | Non-unique index | Not null | Staff-facing name. Additional address fields require product confirmation. |
| created_at | TIMESTAMP WITH TIME ZONE | — | Not null | Creation timestamp. |
| updated_at | TIMESTAMP WITH TIME ZONE | — | Not null | Last update timestamp. |
| deleted_at | TIMESTAMP WITH TIME ZONE | — | Nullable | Soft-deletion timestamp. |
| created_by | TEXT | — | Not null | Authenticated actor that created the record. |
| updated_by | TEXT | — | Not null | Authenticated actor that last updated the record. |
| deleted | BOOLEAN | — | Not null; default `false` | Soft-deletion flag. |

## vehicles

| Field name | Data type | Index | Database constraints | Description |
|---|---|---|---|---|
| vehicle_id | UUID | Primary key | Not null | Stable identifier for the vehicle record. |
| vin | TEXT | Unique index | Not null | Vehicle identification number. |
| license_plate | TEXT | Index; uniqueness scope pending jurisdiction confirmation | Not null | Vehicle registration plate. |
| purchase_date | DATE | — | Not null | Date the company purchased the vehicle. |
| purchase_cost | DECIMAL(15,2) | — | Not null; non-negative | Vehicle purchase cost. Currency policy is not specified in the PRD. |
| insurance_details | TEXT | — | Not null | Insurance details as supplied by the business; structured subfields require confirmation. |
| insurance_expires_on | DATE | — | Not null | Insurance expiration date. |
| odometer_km | BIGINT | — | Not null; non-negative | Odometer reading in kilometers, matching the PRD's maintenance-distance unit. |
| brand | TEXT | — | Not null | Vehicle brand. |
| model | TEXT | — | Not null | Vehicle model. |
| manufacturing_year | SMALLINT | — | Not null | Vehicle manufacturing year. |
| size | TEXT | — | Not null; one of `small`, `medium` | Vehicle size category. |
| vehicle_class | TEXT | — | Not null; one of `economy`, `luxury` | Rental class. |
| seat_count | INTEGER | — | Not null; greater than zero | Number of seats. |
| fuel_type | TEXT | — | Not null; one of `gas`, `electric`, `hybrid` | Vehicle fuel type. |
| ownership | TEXT | — | Not null; fixed to `company` | Only company-owned vehicles are in scope. |
| lifecycle_status | TEXT | Index | Not null; one of `incoming`, `active`, `maintenance`, `decommissioning`, `sold` | Current lifecycle status. |
| home_location_id | UUID | Index; foreign key to `locations.location_id` | Not null | Vehicle's current home location. Must reference a non-deleted location. |
| created_at | TIMESTAMP WITH TIME ZONE | — | Not null | Creation timestamp. |
| updated_at | TIMESTAMP WITH TIME ZONE | — | Not null | Last update timestamp. |
| deleted_at | TIMESTAMP WITH TIME ZONE | — | Nullable | Soft-deletion timestamp. |
| created_by | TEXT | — | Not null | Authenticated actor that created the record. |
| updated_by | TEXT | — | Not null | Authenticated actor that last updated the record. |
| deleted | BOOLEAN | — | Not null; default `false` | Soft-deletion flag. |

## vehicle_location_history

| Field name | Data type | Index | Database constraints | Description |
|---|---|---|---|---|
| vehicle_location_history_id | UUID | Primary key | Not null | Stable identifier for a vehicle's location assignment or transfer. |
| vehicle_id | UUID | Index; foreign key to `vehicles.vehicle_id` | Not null | Vehicle associated with this location event. |
| from_location_id | UUID | Foreign key to `locations.location_id` | Nullable for initial assignment | Prior home location; null for initial onboarding. |
| to_location_id | UUID | Index; foreign key to `locations.location_id` | Not null | New home location. |
| effective_at | TIMESTAMP WITH TIME ZONE | Index | Not null | Time the assignment or transfer takes effect. |
| created_at | TIMESTAMP WITH TIME ZONE | — | Not null | Time the history record was created. |
| updated_at | TIMESTAMP WITH TIME ZONE | — | Not null | Time the history record was last updated. |
| deleted_at | TIMESTAMP WITH TIME ZONE | — | Nullable | Soft-deletion timestamp. |
| created_by | TEXT | — | Not null | Authenticated actor that recorded the assignment or transfer. |
| updated_by | TEXT | — | Not null | Authenticated actor that last updated the record. |
| deleted | BOOLEAN | — | Not null; default `false` | Soft-deletion flag. |

## transfer_costs

| Field name | Data type | Index | Database constraints | Description |
|---|---|---|---|---|
| transfer_cost_id | UUID | Primary key | Not null | Stable identifier for an incurred transfer cost. |
| vehicle_location_history_id | UUID | Unique index; foreign key to `vehicle_location_history.vehicle_location_history_id` | Not null | Transfer associated with this aggregate cost. |
| amount | DECIMAL(15,2) | — | Not null; non-negative | Cost amount recorded for the transfer. |
| currency_code | TEXT | — | Not null | Currency code supplied by the caller under the configured business currency policy. |
| allocation_target | TEXT | — | Not null; fixed to `company_pool` | Required company-pool cost allocation. |
| created_at | TIMESTAMP WITH TIME ZONE | — | Not null | Creation timestamp. |
| updated_at | TIMESTAMP WITH TIME ZONE | — | Not null | Last update timestamp. |
| deleted_at | TIMESTAMP WITH TIME ZONE | — | Nullable | Soft-deletion timestamp. |
| created_by | TEXT | — | Not null | Authenticated actor that recorded the cost. |
| updated_by | TEXT | — | Not null | Authenticated actor that last updated the record. |
| deleted | BOOLEAN | — | Not null; default `false` | Soft-deletion flag. |

## Common Record and Deletion Rules

- Every table includes `created_at`, `updated_at`, and nullable `deleted_at` as timestamps with time zone; `created_by` and `updated_by` as text; and `deleted` as a boolean defaulting to `false`.
- Keep `deleted` and `deleted_at` consistent. Do not physically remove inventory, locations, transfers, or costs through ordinary application operations.
- A location referenced by a current vehicle or retained location history cannot be soft-deleted. A vehicle cannot reference a deleted location as its current home.
- Vehicle creation must create its initial location-history record in the same transaction. Each transfer must create one history record and update `vehicles.home_location_id` in the same transaction.
- Retain historical `from_location_id` and `to_location_id` references even when a location is no longer used as a current home location.
- Use decimal precision 15 and scale 2 for monetary values. Currency conversion and accounting-ledger posting are outside this design.
