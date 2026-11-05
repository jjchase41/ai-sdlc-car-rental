# PRD - Car Rental Accounting

## Document Information
- Product / Feature Name: Car Rental Accounting
- Author: Copilot
- Date:
- Version:

## Table of Contents
- [Overview](#overview)
  - [Background](#background)
  - [Objective](#objective)
  - [Goals](#goals)
- [Problem Statement](#problem-statement)
- [Functional Requirements](#functional-requirements)
  - [FR-1: Rental Chart of Accounts and Ledger](#fr-1-rental-chart-of-accounts-and-ledger)
  - [FR-2: Revenue Recognition and Rental Adjustments](#fr-2-revenue-recognition-and-rental-adjustments)
  - [FR-3: Cost Accounting and Vehicle Profitability](#fr-3-cost-accounting-and-vehicle-profitability)
  - [FR-4: Invoices and Tax](#fr-4-invoices-and-tax)
  - [FR-5: Payment Reconciliation and Receivables](#fr-5-payment-reconciliation-and-receivables)
  - [FR-6: Manual Adjustments, Refunds, and Chargebacks](#fr-6-manual-adjustments-refunds-and-chargebacks)
  - [FR-7: Financial Reporting and Period Close](#fr-7-financial-reporting-and-period-close)
  - [FR-8: Financial Approvals and Audit](#fr-8-financial-approvals-and-audit)
  - [FR-9: Oracle General Ledger Integration](#fr-9-oracle-general-ledger-integration)
- [Non-Functional Requirements](#non-functional-requirements)
- [Dependency & Constraints](#dependency--constraints)
- [Success Metrics](#success-metrics)

## Overview

### Background
The business is expanding from vehicle sales into car rental and needs rental-specific accounting. Rental transactions, fleet costs, reconciliation, and reporting must be tracked separately from the existing sales accounting while posting to the company's Oracle general ledger.

### Objective
Provide a reliable accounting process for rental income and costs that supports daily cash-flow visibility, per-vehicle profitability, financial controls, and timely period close.

### Goals
- Maintain a separate rental chart of accounts (CoA) and an immutable, auditable ledger.
- Record revenue when the related payment is reconciled and track rental costs consistently.
- Reconcile payments against booking records and surface discrepancies promptly.
- Report rental cash flow and profit/loss by vehicle and revenue by location.
- Meet the specified transaction reflection and month-end close targets.

## Problem Statement
Without dedicated rental accounting, the business cannot reliably distinguish rental income and costs from vehicle sales, verify that payments match bookings, or assess profitability per vehicle. Accounting staff need a controlled ledger and clear processes for reconciliation, corrections, approvals, and close. Marketing approvers and operational teams providing booking, return, vehicle, and payment data are also affected.

This is important before rental operations scale: financial records and reconciliation must be complete, sufficiently timely, and retained for audit.

## Functional Requirements

### FR-1: Rental Chart of Accounts and Ledger

**User story title:** Record rental activity in a separate ledger

**Statement:** **As an** accounting user, **I want** rental financial activity recorded against a separate rental CoA, **so that** rental results can be distinguished from vehicle sales and reported consistently.

**Requirement detail:** The initial CoA must support base rental income, late-charge income, separate accounts for penalty and fee types (including damage and cleaning fees), workshop/maintenance cost, tax, fuel cost, vehicle installment, vehicle insurance, depreciation, promotion cost, rental cancellation cost, customer retention cost, chargebacks, and manual adjustments. The CoA must be extensible. Revenue reporting must be segmentable by location. Unspecified mappings, such as delivery/drop-off charges, must be configurable or agreed during accounting setup.

**Acceptance criteria:**
- **Given** a rental financial transaction is ready to post, **when** it is recorded, **then** it is assigned to an appropriate rental CoA account and retains its booking or vehicle reference where applicable.
- **Given** an authorized accounting user adds an account, **when** it is made available, **then** it can be used for subsequent rental entries without changing existing ledger entries.
- **Given** rental revenue has been recorded for multiple locations, **when** an accounting user runs a revenue report, **then** the report can group the revenue by location.

### FR-2: Revenue Recognition and Rental Adjustments

**User story title:** Recognize rental income on payment reconciliation

**Statement:** **As an** accounting user, **I want** rental income recognized when payment is reconciled, **so that** reported income reflects received and verified payments.

**Requirement detail:** Recognize base rental revenue after booking payment reconciliation. Recognize a late charge after vehicle return when payment for that charge is reconciled. Do not accrue revenue for rentals spanning reporting periods. Early returns generally do not result in a refund; an approved exception may be recorded as customer retention cost. A no-show is recorded as rental cancellation cost. Deposits are not held as liabilities under the initial policy. Late, damage, cleaning, and other penalty/fee types use separate CoA accounts.

**Acceptance criteria:**
- **Given** a booking payment is not reconciled, **when** the booking is included in accounting reports, **then** its base rental revenue is not recognized as reconciled rental income.
- **Given** the booking payment is reconciled, **when** the ledger is updated, **then** base rental income is recorded against the booking.
- **Given** a customer has returned the vehicle and paid a late charge, **when** that charge is reconciled, **then** late-charge income is recorded separately from base rental income.
- **Given** a rental crosses a reporting-period boundary, **when** the period is closed, **then** no in-progress-rental revenue accrual is created.
- **Given** an early return receives no refund or an approved exception is handled as retention cost, **when** the transaction is recorded, **then** the default no-refund policy is respected and any exception is classified as customer retention cost.
- **Given** a booking is a no-show, **when** its financial outcome is recorded, **then** the amount is classified as rental cancellation cost.

### FR-3: Cost Accounting and Vehicle Profitability

**User story title:** Track fleet costs and per-vehicle results

**Statement:** **As an** accounting user, **I want** vehicle costs and depreciation associated with the relevant vehicle, **so that** I can evaluate profit or loss per vehicle.

**Requirement detail:** Use straight-line depreciation over 60 months, calculated as vehicle purchase price divided by 60, with the same schedule for tax and management reporting. Revalue or assess impairment annually. Record other costs on demand and on a cash basis; allocate overhead monthly. Vehicle insurance is an annual cost per vehicle. Do not accrue for expected damage or insurance claims.

**Acceptance criteria:**
- **Given** a vehicle's purchase price and depreciation schedule are recorded, **when** monthly depreciation is calculated, **then** the monthly amount is purchase price divided by 60 and associated with that vehicle.
- **Given** workshop, fuel, insurance, installment, or other applicable costs are recorded, **when** a per-vehicle profit/loss report is run, **then** the report includes costs linked to the vehicle.
- **Given** monthly overhead is recorded, **when** monthly financial reporting is prepared, **then** the overhead is included as a monthly cost and is not allocated per rental.
- **Given** the annual asset review is due, **when** the review is completed, **then** revaluation or impairment results can be recorded against the affected vehicle.

### FR-4: Invoices and Tax

**User story title:** Issue simplified USD invoices with configurable VAT

**Statement:** **As an** accounting user, **I want** invoices to use a consistent simplified format and configurable VAT rules, **so that** customers are billed in the supported currency under the applicable tax policy.

**Requirement detail:** Support simplified B2C invoices, USD only, and VAT only for the initial scope. Tax rates and rules must be configurable by Accounting. Location-specific rate differences and exemptions are not required. Invoices do not itemize taxes, surcharges, or environmental fees. Delivery and drop-off charges, either or both, may be added to the base rental charge for cross-location arrangements. Versioned invoice history is not required.

**Acceptance criteria:**
- **Given** a customer invoice is issued, **when** it is presented, **then** it uses the simplified B2C format, is denominated in USD, and does not itemize tax or surcharges.
- **Given** Accounting configures a VAT rule, **when** a new taxable transaction is processed, **then** the configured rule is applied consistently without requiring a location-specific exception.
- **Given** delivery or drop-off is charged, **when** the invoice is prepared, **then** the charge is added to the base rental amount.

### FR-5: Payment Reconciliation and Receivables

**User story title:** Reconcile payments and identify exceptions

**Statement:** **As an** accounting user, **I want** online payments and bank CSV reports reconciled against rental bookings, **so that** mismatches and delayed payments are detected in time for resolution.

**Requirement detail:** Reconcile online payments in real time and bank report CSV transactions daily, excluding holidays. Send email alerts for amount mismatches or delays. Customers pay before rental; delayed-payment accounts and aging reports are out of scope. An accumulated balance such as a late charge may be referred to a third-party debt collector.

**Acceptance criteria:**
- **Given** an online payment is received, **when** the matching booking is identified, **then** reconciliation occurs in real time and the result is reflected in the ledger in real time.
- **Given** a bank report CSV is available on a business day, **when** the daily reconciliation is run, **then** its payments are matched against booking records and exceptions are identified.
- **Given** a payment amount mismatches or is delayed, **when** the exception is detected, **then** an email alert is sent to the designated accounting recipient.
- **Given** an unpaid rental booking is considered for fulfillment, **when** payment status is checked, **then** the booking must be paid before rental; accumulated post-rental balances may be referred to a third-party collector.

### FR-6: Manual Adjustments, Refunds, and Chargebacks

**User story title:** Correct financial records with traceable entries

**Statement:** **As an** accounting user, **I want** corrections and exceptional financial events captured as traceable ledger entries, **so that** the audit history remains complete.

**Requirement detail:** Ledger journals are immutable. Corrections must create a new entry that identifies and corrects or reverses the original; manual adjustments use debit/credit manual-adjustment accounts and include notes. Record chargebacks in a chargeback account with notes; do not generate automated journal entries for chargeback fees. Refunds above defined thresholds require approval from the Marketing supervisor, department head, or director according to the approval hierarchy. The requirements do not define refund thresholds.

**Acceptance criteria:**
- **Given** a posted journal requires correction, **when** an accounting user corrects it, **then** the original remains unchanged and a new correcting or reversing entry references it.
- **Given** a manual adjustment is posted, **when** the entry is saved, **then** it uses the manual-adjustment CoA and includes notes.
- **Given** a chargeback is recorded, **when** it is posted, **then** it uses the chargeback account and contains notes, and no automated journal for the chargeback fee is created.
- **Given** a refund exceeds its approval threshold, **when** it is submitted, **then** it requires approval at the applicable Marketing hierarchy level before processing.

### FR-7: Financial Reporting and Period Close

**User story title:** Review rental financial performance and close periods

**Statement:** **As an** accounting user, **I want** rental cash flow and vehicle-level profit/loss reports, **so that** I can monitor daily income and complete period close promptly.

**Requirement detail:** Provide cash flow/revenue per day and profit/loss per vehicle. Financial revenue reports must support location segmentation. Do not require fleet-utilization dashboards, forecasts, scenario modeling, split billing, or rental line-item aging reports in the initial scope. Target month-end close is T+2 days. Zero-friction close means all accountable transactions are in the ledger, payment reconciliation is complete with no unresolved mismatches, accruals and depreciation are automatically calculated, and manual adjustment/correction entries are less than 1% of total ledger entries.

**Acceptance criteria:**
- **Given** rental financial data is available, **when** an accounting user runs the daily report, **then** revenue/cash flow per day is shown.
- **Given** vehicle costs and revenue are recorded, **when** an accounting user runs profitability reporting, **then** profit or loss can be reviewed per vehicle.
- **Given** a month-end period is ready to close, **when** the close process is completed, **then** all accountable transactions are present, reconciliation mismatches are resolved, and the period closes by T+2 days.
- **Given** a period is closed, **when** close performance is measured, **then** the proportion of manual adjustment/correction entries can be compared against the target of less than 1% of ledger entries.

### FR-8: Financial Approvals and Audit

**User story title:** Apply hierarchical approvals and retain audit history

**Statement:** **As an** accounting manager, **I want** financial approvals and changes traceable to their responsible users, **so that** financial activity is governed and auditable.

**Requirement detail:** Financial approvals follow thresholds through supervisor, department head, and director levels. The approver is the poster's direct reporting line. Record creation date and the person responsible for each update. Retain financial records for 10 years. Automated control checks for negative revenue or duplicate invoices are not required.

**Acceptance criteria:**
- **Given** a financial action requires approval, **when** it is submitted, **then** it is routed to the poster's direct reporting line and escalated according to the configured threshold hierarchy.
- **Given** a financial record is created or updated, **when** its audit history is reviewed, **then** the creation date and responsible person are available.
- **Given** a financial record is retained, **when** it reaches its retention period, **then** it has been retained for at least 10 years.

### FR-9: Oracle General Ledger Integration

**User story title:** Transfer rental ledger data to Oracle

**Statement:** **As an** accounting user, **I want** rental ledger activity exchanged with Oracle General Ledger using CSV, **so that** rental financial data is reflected in the company's general ledger.

**Requirement detail:** Oracle General Ledger is the target system. Use CSV file exchange only; API-based integration is out of scope. Reconciled payment transactions must be reflected in the ledger in real time. Other transaction reflection may take up to T+2 days. File exchange and processing must support the daily bank CSV reconciliation process.

**Acceptance criteria:**
- **Given** rental ledger data is ready for Oracle, **when** it is exported, **then** it is provided in CSV format suitable for the agreed Oracle General Ledger import.
- **Given** a payment is reconciled, **when** the corresponding accounting entry is created, **then** the ledger reflects it in real time.
- **Given** a non-payment accounting transaction is recorded, **when** it is transferred to Oracle, **then** it is reflected within T+2 days.

## Non-Functional Requirements

## Dependency & Constraints
- The accounting solution depends on accurate booking, vehicle, return, payment, location, and cost data from rental operations.
- Oracle General Ledger is the external accounting system; integration is limited to CSV file exchange.
- The initial scope supports USD and VAT only, with one tax rate policy across locations; Accounting must be able to configure tax rules.
- Refund approval thresholds and the detailed Oracle CSV layout must be agreed before implementation.
- Delivery/drop-off charge account mapping and any additional account mappings not specified above must be finalized during accounting setup.
- Deposits are not treated as liabilities under the stated initial policy.
- Accruals for in-progress rentals, expected damage, and insurance claims are not required. The close target mentions automatic accrual calculation; the accrual types and policies remain to be clarified before implementation.
- No dual reporting under IFRS and local GAAP is required. Regulatory compliance checklists and audits are handled manually for now.
- Split billing, mid-rental vehicle swaps, refunds for unused prepaid packages, rolling forecasts, scenario modeling, and aging reports are outside the initial scope.
- Invoices are simplified B2C invoices without tax or surcharge itemization or versioned invoice history.
- Financial records must be retained for 10 years.

## Success Metrics
- Daily cash flow/revenue reporting is available and supports monitoring daily income.
- Profit/loss is reportable for each vehicle and revenue can be analyzed by location.
- Reconciliation of online payments is real time; bank CSV reconciliation runs daily excluding holidays; detected mismatches or delays generate email alerts.
- Reconciled payment activity is reflected in the ledger in real time and other transactions within T+2 days.
- Month-end close is completed by T+2 days.
- At close, all accountable transactions are in the ledger and no payment reconciliation mismatches remain unresolved.
- Depreciation and any applicable accrual calculations are automated; manual adjustment/correction entries are less than 1% of all ledger entries.
