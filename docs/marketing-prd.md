# Marketing Product Requirements

## Purpose

Support marketing's first-year goals of increasing brand awareness and acquiring customers for the car rental business. The initial product should focus on a simple rental discovery and booking journey, clear rental pricing, and basic business reporting.

## Goals and success measures

- Increase brand awareness and customer acquisition.
- A successful customer completes a rental, pays on time, and returns the car on time. A rental extension is a positive outcome, provided payment is made on time.
- Keep the rented-car percentage at or above 75%.
- Provide the reports listed below so the team can monitor utilization, income, and customer growth and retention.

The rented-car percentage is currently described as the number of cars rented in a week compared with the total fleet. The precise calculation and treatment of cars unavailable for rental must be confirmed before implementation.

## Users and customer segments

### Customer segments

- Local customers
- Tourists
- Insurance replacement customers

Segments are defined and reassessed manually. No segment-specific pricing tiers or bundles are required initially, and no additional demographic or behavioral attributes have been specified.

### Marketing roles

The roles are staff, supervisor, marketing department head, and marketing director. Rental approval limits are configurable:

| Role | Approval limit |
| --- | --- |
| Staff | X |
| Supervisor | Y |
| Marketing department head | Z |
| Marketing director | Unlimited |

The department head approves promotions before they go live if promotions are introduced. The meaning, units, and escalation behavior of X, Y, and Z must be defined by the business.

## Customer journey and channels

- Support rental discovery on the web and through marketplace aggregators.
- Instrument the initial funnel in this order: search → vehicle selection → confirmation → payment.
- Provide a price scenario on the confirmation page.
- English is the only required language at launch.

Abandoned-search recovery, incomplete-booking recovery, and on-site personalization are not required initially.

## Pricing requirements

- Use static rates for these vehicle categories:
  - Small regular car
  - Medium regular car
  - Medium luxury car
- Offer daily, weekly, and monthly rental rates. The effective daily rate should be highest for daily rentals, lower for weekly rentals, and lowest for monthly rentals.
- Support a time-limited pricing adjustment for peak holiday seasons.
- Allow regular competitor-price monitoring inputs. The input method and review cadence are not yet specified.
- Show a price scenario on the confirmation page. The business must confirm the exact point in the journey at which this page appears and what-if assumptions the scenario should support.
- Record who changed a price tier and when, including the previous and new values.

Dynamic pricing, other seasonal overlays, and pricing simulations beyond the confirmation-page scenario are not currently required.

## Payment and rental risk

Late or missing payment for a rental or extension is the primary business risk. The system should make payment status visible for rentals and extensions and support monitoring whether customers pay on time. The business must define the action for an unpaid rental or extension (for example, whether approval or extension is blocked); that policy is not specified in the gathered requirements.

## Reporting and data retention

Provide daily, weekly, and monthly reporting for:

- Rented-car percentage
- Income

Also report:

- New customers within each month
- Customers with repeat orders

Retain marketing performance data for three years. No additional analytics platform, real-time delivery, cohort analysis, anomaly alerts, or campaign attribution requirements have been identified.

## Privacy and consent

Obtain customer consent for valid ID card validation. The consent flow, evidence to retain, and applicable retention period need to be defined before implementation.

## Out of scope for the initial release

- Promotions and campaign management, including promo codes, eligibility rules, scheduling, and A/B testing
- Add-ons, cross-selling, and upselling
- Loyalty program
- Personalized content
- Abandoned-search or incomplete-booking recovery
- CRM/CDP, affiliate, and aggregator pricing or inventory-feed integrations
- Multilingual support beyond English
- Sandbox/production campaign separation
- Subscription rentals, car sharing, and other future service bundles

## Open decisions

1. Define the rented-car percentage formula, including unavailable cars and the reporting period boundary.
2. Set the values, units, and approval/escalation behavior for X, Y, and Z.
3. Define how unpaid rentals and extensions affect approval, continued rental, and extension eligibility.
4. Confirm holiday-season dates and how seasonal adjustments interact with daily, weekly, and monthly rates.
5. Specify competitor-price input method and cadence.
6. Define the confirmation-page scenario's position in the booking flow and supported assumptions.
7. Define the ID-validation consent flow, recorded evidence, and retention.
