# PRD - Marketing

## Document Information

- Product / Feature Name: Marketing
- Author: @copilot
- Date:
- Version:

## Table of Contents

- [Overview](#overview)
  - [Background](#background)
  - [Objective](#objective)
  - [Goals](#goals)
- [Problem Statement](#problem-statement)
- [Functional Requirements](#functional-requirements)
  - [Rental Demand and Customer Segments](#rental-demand-and-customer-segments)
  - [Booking Funnel Measurement](#booking-funnel-measurement)
  - [Rental Pricing](#rental-pricing)
  - [Rental Approval Authority](#rental-approval-authority)
  - [Marketing Reporting](#marketing-reporting)
  - [Identity Validation Consent](#identity-validation-consent)
  - [Marketing Data Retention](#marketing-data-retention)
- [Non-Functional Requirements](#non-functional-requirements)
- [Dependency & Constraints](#dependency--constraints)
- [Success Metrics](#success-metrics)

## Overview

### Background

The company is expanding from car sales into car rentals and has no prior rental experience or dedicated rental systems. Marketing needs a basic way to support awareness and customer acquisition, present rental pricing, understand the booking funnel, and report on rental performance.

### Objective

Define launch requirements for marketing capabilities in the car rental system, keeping the initial scope focused on customer acquisition, basic pricing, rental activity reporting, and the specified approval and consent needs.

### Goals

- Support brand awareness and customer acquisition through web and marketplace aggregator discovery channels.
- Provide clear static rental rates for the launch vehicle categories and rental durations.
- Track the key booking funnel and rental performance.
- Establish manual customer segments and role-based rental approval limits.
- Keep launch capabilities deliberately limited; defer promotions, loyalty, add-ons, personalization, and advanced experimentation.

## Problem Statement

Marketing is entering a new business line without dedicated rental systems or established rental processes. Without basic rental pricing, funnel visibility, and performance reporting, the team cannot assess whether it is acquiring customers or meeting the target fleet rental rate. The business is particularly concerned about customers who rent or extend a rental without paying on time.

Marketing staff, supervisors, the marketing department head, and the marketing director need a clear initial set of capabilities and approval boundaries. The rental business also needs to recognize local, tourist, and insurance-replacement customers, while obtaining consent for valid ID card validation.

These needs are important at launch because marketing’s first-year objectives are brand awareness and customer acquisition, while the rental operation needs a measurable baseline and controls from the start.

## Functional Requirements

### Rental Demand and Customer Segments

#### User Story: Understand launch customer segments

- **Statement:** **As a** marketing staff member, **I want** to classify customers as local, tourist, or insurance replacement, **so that** marketing can understand the customer groups using the rental service.
- **Requirement detail:** Segments are defined and reassessed manually. The system does not need demographic or behavioral segmentation attributes, automated segment rules, or machine-learning-driven segmentation at launch. No segment-specific pricing tiers or bundles are required.
- **Acceptance criteria:**
  - **Given** a customer record, **when** an authorized user assigns a segment, **then** the customer can be classified as local, tourist, or insurance replacement.
  - **Given** an existing customer segment, **when** an authorized user manually reassesses the customer, **then** the segment can be updated without an automated segmentation process.

### Booking Funnel Measurement

#### User Story: Measure rental booking activity

- **Statement:** **As a** marketing staff member, **I want** to measure the main booking steps across web and marketplace aggregator channels, **so that** I can understand customer activity from discovery through payment.
- **Requirement detail:** The steps to measure are search, vehicle selection, confirmation, and payment, in that order. Abandoned-search recovery, incomplete-booking recovery, and on-site personalization are not required at launch.
- **Acceptance criteria:**
  - **Given** a customer uses a supported discovery channel, **when** the customer searches, selects a vehicle, confirms, or pays, **then** the corresponding funnel step and channel are recorded.
  - **Given** a booking does not proceed to the next funnel step, **when** the customer leaves or stops, **then** the system is not required to initiate recovery or personalization.

### Rental Pricing

#### User Story: Present launch rental rates

- **Statement:** **As a** marketing staff member, **I want** static rates for each launch vehicle category and rental duration, **so that** customers can see the applicable rental price.
- **Requirement detail:** Vehicle categories are small regular car, medium regular car, and medium luxury car. Rates support daily, weekly, and monthly rentals; the daily rate is the most expensive, the weekly daily-equivalent rate is cheaper, and the monthly daily-equivalent rate is the cheapest. Pricing is static, with a seasonal price overlay for peak holiday season. Competitor prices are monitored regularly as an input; automated competitor feeds are not required.
- **Acceptance criteria:**
  - **Given** a vehicle in a supported category, **when** a customer views its rental price, **then** the applicable daily, weekly, or monthly rate is shown.
  - **Given** rates for the same vehicle category, **when** daily-equivalent prices are compared, **then** the daily rate is highest, the weekly rate is lower, and the monthly rate is lowest.
  - **Given** a proposed pricing change, **when** an authorized user reviews the proposed change on the confirmation page, **then** the system provides a price scenario preview before activation.
  - **Given** an authorized user changes a price tier, **when** the change is saved, **then** the audit history records who changed the tier.
  - **Given** peak holiday season, **when** seasonal pricing applies, **then** the applicable seasonal price overlay can be reflected in the presented rate.

### Rental Approval Authority

#### User Story: Apply rental approval limits by role

- **Statement:** **As a** marketing department head, **I want** configurable approval limits for staff, supervisors, and the department head, **so that** rental approvals follow the organization’s assigned authority levels.
- **Requirement detail:** Staff, supervisors, and the marketing department head have configurable maximum approval levels X, Y, and Z respectively. The marketing director has an unlimited approval level. A request above the acting user’s configured limit must not be approved by that user.
- **Acceptance criteria:**
  - **Given** an approval request and a configured threshold, **when** a staff member, supervisor, or marketing department head reviews it, **then** approval is allowed only up to that role’s configured maximum (X, Y, or Z).
  - **Given** an approval request above a user’s configured maximum, **when** that user attempts to approve it, **then** the approval is not permitted at that level.
  - **Given** an approval request, **when** the marketing director reviews it, **then** the director can approve it without a configured maximum limit.

### Marketing Reporting

#### User Story: Review rental performance reports

- **Statement:** **As a** marketing staff member, **I want** daily, weekly, and monthly rental performance reports, **so that** I can monitor fleet rental activity, income, and customer acquisition and retention.
- **Requirement detail:** Reports include the percentage of cars rented, rental income, new customers within a month, and customers with repeat orders. For each reporting period, the rented-car percentage is the number of cars rented during that period divided by the number of cars in the fleet; the weekly target is at least 75%. Daily, weekly, and monthly reporting views are required for rented-car percentage and income. No other rental-specific KPIs are required for now.
- **Acceptance criteria:**
  - **Given** rental and fleet data for a reporting period, **when** an authorized user requests a daily, weekly, or monthly report, **then** the rented-car percentage and rental income are available for that period.
  - **Given** rental and fleet counts for a reporting period, **when** the rented-car percentage is calculated, **then** the result is the number of cars rented during that period divided by the number of cars in the fleet.
  - **Given** customer booking history, **when** the monthly report is requested, **then** it includes new customers within the month and customers with repeat orders.

### Identity Validation Consent

#### User Story: Record consent for ID card validation

- **Statement:** **As a** customer, **I want** to provide consent for valid ID card validation, **so that** my identity document can be validated for the rental process with my consent.
- **Requirement detail:** The system must capture customer consent for valid ID card validation. No broader marketing consent or preference-management behavior is specified for launch.
- **Acceptance criteria:**
  - **Given** that ID card validation is required, **when** the customer is asked for consent, **then** the customer’s consent decision is captured.
  - **Given** that consent has not been provided, **when** validation is requested, **then** the system does not represent that validation consent was obtained.

### Marketing Data Retention

#### User Story: Retain marketing performance history

- **Statement:** **As a** marketing staff member, **I want** marketing performance data retained for three years, **so that** I can review historical performance.
- **Requirement detail:** Marketing performance data must be retained for three years.
- **Acceptance criteria:**
  - **Given** recorded marketing performance data, **when** it is less than three years old, **then** it remains available for historical review.
  - **Given** marketing performance data reaches the three-year retention period, **when** retention is applied, **then** it is no longer required to be retained as marketing performance history.

## Non-Functional Requirements

## Dependency & Constraints

- The initial supported discovery channels are web and marketplace aggregators.
- English is the only required language at launch.
- Customer segments are managed manually.
- Pricing uses static rates by vehicle category and rental duration, with a peak-holiday seasonal overlay. Competitor price monitoring is regular, but no automated feed is required.
- Promotions, promotion scheduling and eligibility rules, campaign attribution/ROI, A/B testing, loyalty, add-ons, upsells, customer recovery, personalization, and advanced analytics are out of scope for now.
- If promotion capabilities are introduced later, the marketing department head is the stated approver. Promotion management itself is not a launch requirement.
- Marketing preference changes, unsubscribe propagation, and geographic advertising compliance are not required for now.
- Rental approval thresholds X, Y, and Z must be configurable. The marketing director’s approval level is unlimited.
- The main business risk is rental or extension without timely payment. No specific payment enforcement workflow or payment KPI was provided in the marketing requirements.

## Success Metrics

- Achieve a weekly rented-car percentage of at least 75%, calculated as cars rented during the week divided by the fleet size.
- Make rented-car percentage and rental income reports available daily, weekly, and monthly.
- Make monthly new-customer and repeat-order customer reporting available.
- Support the first-year marketing objectives of brand awareness and customer acquisition. No numeric targets for these objectives were provided.
