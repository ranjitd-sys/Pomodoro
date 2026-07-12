### DeepEcom Proccurement manegement

## Current Problems

- PR creation — still manually entered, but through a proper form instead of a spreadsheet. No external integration needed -- u two thins need .

- Bank reconciliation — this one likely does need a file upload (bank statement export) since matching hundreds of transactions by hand is exactly the pain point - - - being solved. But it can start as "upload CSV, system matches" rather than a live bank API.
- SAP PO/GRPO — this is the one place true automation happens: instead of a person manually typing PO/GRPO entries into SAP, the system pushes it automatically once - QC passes. This needs the SAP Service Layer/API — the one integration that's non-negotiable for the "automation" promise to hold.


> **Note:** Every step in the above workflow is performed manually, requiring human intervention. This results in delays, repetitive work, and a higher chance of errors.


## flow
PR → Price Validation → Approval → Order → Payment → Delivery/Pickup → QC → Reconciliation → SAP (PO/GRPO)

### Build 
| Phase | Features | Why This Order? |
|-------|----------|-----------------|
| **0** | Data Model, Authentication, PR (Purchase Request) Module | Establishes the core foundation. All subsequent modules depend on a consistent data model, user authentication, and purchase request workflow. |
| **1** | Price Validation, Approval Workflow, Order Management, Procurement Dashboard | Implements the primary procurement workflow. Once PRs exist, orders can be created, validated, approved, and tracked through dashboards. |
| **2** | Payment Tracking, Delivery/Pickup Tracking, Bank Reconciliation | Requires completed purchase orders. Payments, deliveries, and financial reconciliation can only occur after orders are placed. |
| **3** | Quality Control (QC), Refund/Cancellation, Reconciliation Engine | Depends on delivery and payment information. Products must be received and payments recorded before QC, refunds, or reconciliation can be processed. |
| **4** | SAP PO/GRPO Automation, Remaining Dashboards, Exception Management | Final automation phase. SAP integration, advanced reporting, and exception handling rely on QC-approved and financially reconciled transactions to ensure accurate ERP synchronization. |



# Queustion
1. how do we get data
2. How do we validate OTP and need more clarity on OTP
4. How can we get bank satement someone will upload or any api intrgration 
5. how do we also get Delivery data 
6. Order tracking, delivery, pickup — status still updated manually by whoever's doing it (ops/warehouse), just through the app instead of a shared sheet.

---

#### Module 1 – Purchase Request Management
Objective
Digitize and centralize procurement requirements.

Features
Capture:
◦ Mobile Model
◦ Quantity
◦ Target Purchase Price
◦ Marketplace
◦ Delivery Location
◦ Requested By
◦ Purchase Date
◦ OTP capturing by business owner for locking the order using WhatsApp
◦ CN Amount if any

sol : 
upload or have form to raise pr and have schema to validate and give them a PTN number


#### Module 2 – Price Validation
Objective
Ensure procurement occurs only at approved pricing.
> Doubt: After creating the PR, are we going to show them:
`Enter marketplace price, target price, and current marketplace price.`

price mismatch -> reject notify BO 
price match -> approval

#### Module 3 – Approval Workflow
Objective
Automate procurement approvals.
generate approvel sheet and then send whatsapp otp and then verify and then 
Approval Actions
        ◦ Review
        ◦ Approve
        ◦ Reject
        ◦ Hold
> Doubt : on the basis of what we are going to perform this action

#### Module 4 – Order Management System
-> not clered how we are going to get data

#### Module 5 – Payment Management
Objective
Track all prepaid transactions.

sol:
uplaod transaction details or maulay enter

#### Module 6 – Bank & Card Reconciliation
Objective
Automate payment reconciliation.
sol:
create a map of Marketplace Order Amount map with the Card/Bank Statement Amount

Identify
◦ Missing Transactions
◦ Duplicate Transactions
◦ Short Deductions
◦ Excess Deductions
◦ Unmatched Payments

* Create Report
◦ Reconciliation Report
◦ Payment Exception Report
◦ Card-wise Spend Report
◦ Bank-wise Spend Report

sol: 
    once i get the date will find all the Transactions and gruop them and then put in to the csv file or any pdf report 
> Note : i am not sure about this 


#### Module 7 – Delivery Management
Objective
Provide complete visibility of deliveries.

not sure where to get data 

#### Module 8 – Pickup Management
Objective
Manage self-pickup operations.
not sure where to get data

#### Module 9 – Refund & Cancellation Management
Business Rule
not sure 
how do i know is order is delivered


#### Module 10 – Quality Control Management
Objective

Digitize QC operations. 
not sure 

#### Module 11 – Procurement Reconciliation Engine
Objective
Provide end-to-end reconciliation.
not sure how this flow works