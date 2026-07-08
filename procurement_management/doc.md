### DeepEcom Proccurement manegement

## Current Problems

- PR creation — still manually entered, but through a proper form instead of a spreadsheet. No external integration needed.
- Price validation — the person still checks the marketplace price themselves (opens Amazon/Flipkart, looks at it) and enters it into the system. The system's job is - just to compare entered price vs approved price and block/alert — not to auto-fetch prices. No scraping/API needed for v1.
- Approval — human still approves, just via the app instead of a printed/signed sheet.
- OTP — human still gets OTP over WhatsApp, just types it into the system to lock the order — no WhatsApp API needed.
- Order tracking, delivery, pickup — status still updated manually by whoever's doing it (ops/warehouse), just through the app instead of a shared sheet.
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