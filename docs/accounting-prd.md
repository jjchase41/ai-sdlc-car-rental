# Accounting Product Requirements

## 1. Purpose

Define the accounting capabilities required for the car rental system. The system must give the business a reliable daily view of cash income and profitability, account for rental and fleet activity in a separate chart of accounts (CoA), and provide an auditable ledger that can be reconciled with payments and Oracle General Ledger (GL).

## 2. Goals and success measures

- Record daily income and support cash-flow monitoring.
- Report revenue and profit/loss by vehicle and revenue by location.
- Ensure all accountable transactions are represented in the ledger.
- Complete payment reconciliation with no unexplained mismatches.
- Automatically calculate vehicle depreciation.
- Keep manual adjustments and corrections below 1% of total ledger entries.
- Support period close within two days of month end (T+2).

## 3. Users and permissions

- **Accounting users** manage reconciliations, review reports, and prepare period close.
- **Posters** create financial transactions and adjustment requests.
- **Approvers** are the poster’s direct reporting-line manager. Financial approvals follow the hierarchy of supervisor, department head, then director, according to transaction thresholds.
- Refunds above the applicable threshold require approval by a Marketing supervisor, Marketing department head, or Marketing director.

Approval thresholds and delegated-approval rules must be configurable. The system must enforce the configured hierarchy and prevent a poster from approving their own action.

## 4. Scope

### In scope

- Separate rental CoA with revenue reporting by location.
- Ledger entries for rental, fees, taxes, and fleet costs.
- Payment reconciliation for online transactions and bank-report CSV files.
- Reconciliation discrepancy notifications by email.
- Vehicle depreciation and per-vehicle profitability reporting.
- Manual adjustment, correction, and chargeback entries with audit details.
- Simplified B2C invoices in USD with VAT.
- CSV exchange with Oracle GL.
- Financial reports and month-end close support.

### Out of scope for now

- Multi-currency, foreign exchange, and dual IFRS/local-GAAP reporting.
- Corporate delayed payment, accounts-receivable aging, and doubtful-account provisioning.
- Accruals for in-progress rentals, expected damage, or insurance claims.
- Forecasting, scenario modeling, and fleet-utilization dashboards.
- Automated regulatory compliance checklists and regulatory audits.
- Split billing, vehicle swaps during rentals, and unused prepaid-package refunds.
- Automated chargeback-fee journals.

## 5. Functional requirements

### 5.1 Chart of accounts and classification

1. The system shall use a separate CoA for rental operations.
2. The initial CoA shall support at least:
   - Base rental income.
   - Late charge income.
   - Maintenance/workshop cost.
   - Tax.
   - Fuel cost.
   - Vehicle installments.
   - Vehicle insurance.
   - Vehicle depreciation.
   - Promotion cost.
   - Rental cancellation cost.
   - Customer retention cost.
   - Manual adjustment (debit or credit).
   - Chargeback.
3. Damage, cleaning, delivery, and drop-off fees shall use distinct CoA accounts rather than being combined with base rental income. The final account names and mappings are to be configured by Accounting.
4. Revenue shall be segmentable by location. Per-vehicle costs and revenue shall be reportable together for vehicle profitability.
5. Accounting shall be able to add accounts as the business grows.

### 5.2 Revenue recognition and rental events

1. Recognize rental revenue when payment has been reconciled after booking.
2. Recognize late-charge revenue only after the vehicle has been returned and the late-charge payment has been reconciled.
3. Early returns do not ordinarily result in a refund. An approved exception may be recorded as customer-retention cost, not as a reduction of rental revenue.
4. Record no-shows as rental cancellation cost.
5. Delivery and drop-off charges are additional to the base rental and shall be accounted for separately.
6. Do not accrue revenue for rentals spanning reporting periods.

### 5.3 Costs, assets, and depreciation

1. Record non-depreciation costs on demand and on a cash basis.
2. Allocate overhead monthly; per-rental allocation is not required.
3. Calculate straight-line depreciation for each vehicle over 60 months using purchase price divided by 60 as the monthly amount.
4. Use the same depreciation schedule for management and tax reporting.
5. Support yearly vehicle revaluation or impairment.
6. Track annual insurance for each vehicle. The expense-recognition timing for an annual premium has not been specified and must be confirmed before implementation.
7. Support per-vehicle profit/loss reporting.

### 5.4 Invoices and tax

1. Produce simplified B2C invoices, denominated in USD.
2. Support VAT only, using a rule-based tax configuration that Accounting can manage.
3. Tax rates do not vary by location according to the current requirements.
4. Do not itemize taxes, surcharges, or environmental fees on the customer invoice.
5. Keep tax accounting in the appropriate CoA even when tax is not itemized on the invoice.

### 5.5 Payments and reconciliation

1. Support automated online payment reconciliation in real time.
2. Support bank-statement reconciliation from CSV on a daily basis, excluding holidays.
3. Link reconciled payment records to their booking and ledger entries.
4. Identify amount mismatches and delayed settlements, and send discrepancy alerts by email.
5. No customer credit terms are supported; customers pay first. No overdue balance is expected, but accumulated balances such as late charges may be referred to a third-party debt collector.

### 5.6 Adjustments, refunds, and chargebacks

1. A manual adjustment shall create a ledger entry against the manual-adjustment CoA and include notes.
2. A posted ledger journal is immutable. A correction must create a new entry that identifies and corrects or reverses the original journal; the original must remain unchanged.
3. Record chargebacks to the chargeback account with notes. Do not automatically create journal entries for chargeback fees.
4. Route financial approvals through the configured supervisor → department head → director hierarchy and applicable thresholds.
5. Route refunds above the applicable threshold to an authorized Marketing supervisor, department head, or director.
6. Do not maintain versioned invoice history.

### 5.7 Reporting and close

The system shall provide:

- Daily cash-flow/revenue reporting (revenue per day).
- Profit/loss by vehicle.
- Revenue by location.
- Income and tax reports.
- Rental agreement/contract reports.
- Vehicle maintenance and safety reports.
- Fleet and vehicle availability reports.
- Accident/damage reports.
- Driver’s license verification reports.
- Payment reconciliation status and mismatch reporting.
- A month-end close view showing whether accountable transactions are in the ledger, payments are reconciled without mismatch, and depreciation is calculated.

The target is to reflect a transaction in accounting within T+2 days and close the period within T+2 days after month end.

### 5.8 Oracle GL integration

1. Integrate with Oracle General Ledger using CSV import/export only.
2. Send payment-reconciliation ledger updates in real time, subject to the CSV-only integration constraint.
3. Send other transaction updates on an end-of-day basis.
4. The integration design must reconcile this latency requirement with batch/file transfer and make failed or incomplete transfers visible to Accounting.

## 6. Audit, retention, and security

- Retain financial records for 10 years.
- For each record update, retain its creation date and the identity of the person responsible for the update.
- Preserve ledger journals as immutable entries; corrections are additive and reference the original entry.
- Financial approvals must be traceable to the poster, approver, decision, and applicable threshold.
- No additional field-level access restrictions or anonymized reporting are required at this time.

## 7. Operational targets

- Reconciliation: real time for online payments; daily for CSV bank reports, excluding holidays.
- Accounting reflection: no later than T+2 days.
- Month-end close: no later than T+2 days after month end.
- Email notification: alert Accounting to reconciliation amount mismatches and delays.

## 8. Acceptance criteria

1. A reconciled booking payment creates the corresponding rental revenue entry; a late charge is not recognized until return and payment reconciliation.
2. Revenue can be filtered by location, and a vehicle’s revenue and costs can be viewed together to determine profit/loss.
3. A vehicle’s monthly depreciation is calculated as purchase price / 60 and uses the same schedule for management and tax reporting.
4. The system accepts a bank-report CSV, reconciles it daily on non-holidays, and reports unmatched or mismatched transactions; online reconciliation updates in real time.
5. A mismatch or delayed settlement produces an email alert.
6. A manual adjustment requires notes and the configured approval path. Posted ledger entries cannot be edited or deleted; corrections create linked entries.
7. The system produces simplified USD B2C invoices with VAT accounting and without tax itemization.
8. Accounting can export/import ledger data through CSV for Oracle GL, with real-time handling for payment-reconciliation entries and end-of-day handling for other entries.
9. Financial records and update identity/date information remain available for 10 years.
10. Month-end close reporting identifies unreconciled payments, missing accountable ledger transactions, and uncalculated depreciation.

## 9. Decisions requiring confirmation

- Approval thresholds and whether each threshold requires one approval or sequential approvals at multiple hierarchy levels.
- Recognition and allocation treatment for annual vehicle-insurance premiums.
- The settlement and ledger treatment for deposits, since deposits are not to be held as liabilities but the replacement treatment was not specified.
- The precise CoA mappings for damage, cleaning, delivery/drop-off, and any additional fee types.
- How real-time Oracle GL synchronization will be achieved while the integration is CSV-only.
- Whether T+2 transaction reflection is measured in calendar or business days.
