---
name: powergrid-routing-skill
version: 1.0
description: >
  Routing disambiguation skill for the Power Grid Support Routing Agent.
  Built from empirical analysis of 40,574 historical tickets across 11 teams.
  Covers every known case where a ticket sounds like it belongs to one team
  but actually belongs to another. Read this BEFORE applying routing rules.
---

# Power Grid Routing Agent — Disambiguation SKILL

> This skill exists for one purpose: to stop the agent from routing a ticket
> to the wrong team because the surface language of the ticket was misleading.
> Every pattern documented here is extracted from real ticket data.
> Read each section fully before routing any ticket in that category.

---

## SKILL 1 — THE CUSTOMER CODE TRAP
### "Customer" does NOT always mean SD_TEAM

This is the single most dangerous confusion in the entire dataset.
**Both SD_TEAM and FICO_TEAM handle customer-related tickets.**
The split is determined by what action is being performed, not just the word "customer."

### Route to SD_TEAM when:
- Creating a new customer code / customer ID
- Customer code for scrap sale, scrap vendor
- Billing document creation for customer (VF01)
- Scrap invoice, IRN, e-invoice for customer
- Tax calculation error in a sales transaction
- GSTIN needed for e-invoicing / GST compliance in SD
- VA01 authorization issues (sales order)
- Customer name correction (if it's a transactional fix)
- Post goods issue not allowed
- Item category not defined for cube testing / sale order type error

**Real SD ticket examples:**
```
"Creation of customer code"
"GSTIN UPDATE" ← sounds like FICO but SD owns this when it's for e-invoicing
"Creation of Customer ID for Scrap Vendor"
"Customer code for Vendor - ABCD086119" ← customer for scrap = SD
"Unable to transact in VA01 while having..."
"Cancellation of Draft Invoice"
"Post good issue not allowed"
"Tax code ZOIG is not showing in Scrap order"
"Addition of GST TAX code" ← in scrap sale context = SD
```

### Route to FICO_TEAM when:
- **Extending** a customer code to a company code (1101, 1120, 1130, 1131, etc.)
- Updating **PAN** of a customer AND/OR extending to company codes
- Updating **GSTIN** in customer master (FI view, not SD transaction)
- Adding / updating **bank account** in customer code (FI side)
- Customer code not defined in a company code (financial org issue)
- Mapping customer to a company code
- TCS tax code mapping on customer master
- Creating a Business Place for a customer (nuclear/power entity)
- Customer address / PIN code correction in master data
- One-time customer code mapping (299000xxxx)

**Real FICO ticket examples:**
```
"Extension of Customer code 2110620 to Co code 1131"
"Extend Customer from 1136 to various company codes"
"UPDATION OF PAN NUMBER IN CUSTOMER CODE" ← NOT SD
"Updation of bank details in customer code" ← NOT SD
"Adding Bank Ac to Customer ID 2170914" ← NOT SD
"UPDATION OF GSTN IN CUSTOMER CODE 2110760" ← NOT SD
"Customer Mater TCS Tax code to be mapped"
"Extend customer ABCD839 from 1136 to 1140"
"Customer not defined in Company Code 1140"
"Mapping of Customer 2990000005 in company code 1141"
"Update Address in Customer" ← address in FI master = FICO
```

### THE RULE:
```
Customer code + CREATION / TRANSACTION / BILLING    → SD_TEAM
Customer code + EXTENSION TO COMPANY CODE / FI MASTER DATA UPDATE → FICO_TEAM
```

---

## SKILL 2 — THE VENDOR CODE TRAP
### "Vendor" appears in MM, FICO, BASIS, and SRM tickets

Four different teams handle vendor-related issues. Read carefully.

### Route to MM_TEAM when:
- Extending a vendor to a **plant** or **storage location** (procurement org)
- Vendor code **mapping** in a PO context
- Bank account not mapped in PO (invoice party mismatch)
- Invoice party and bank account mismatch in a PO
- Vendor not defined at plant level
- Vendor code extension for purchase organization

**Real MM ticket examples:**
```
"Extend vendor code ABCD089671 to plant 2304"
"Vendor code mapping"
"Invoice party and bank account is not mapped"
"BANK ACCOUNT NOT MAPPED IN PO"
"in PO no. 3000111090 vendor bank details..." ← PO context = MM
"Regarding mismatch in bank details as per PO"
"Extend the vendor id ABCD089842 to Purchase Org"
"Extension of Vendor for 1101" ← if plant/purchase org context
```

### Route to FICO_TEAM when:
- Updating vendor **bank account / IFSC / bank details** (master data update)
- Vendor code extension to a **company code**
- Vendor code **locked / blocked for payment**
- Vendor **GST / PAN / TDS** mapping on vendor master
- **Deleting** old bank account from vendor master
- MSE vendor master updation
- FI person responsible change on vendor
- Vendor name + bank details update together

**Real FICO ticket examples:**
```
"BANK DETAILS UPDATION - ABCD051901"
"Change of IFSC Code"
"Extend vendor code ABCD090015 to company code"
"Vendor code locked"
"Deletion of Bank Account" ← standalone from vendor master = FICO
"BANK DELETION VENDOR ABCD035294"
"NAME CHANGE IN VENDOR ABCD082894" ← when combined with bank/GST = FICO
"MSE VENDOR MASTER UPDATION"
```

### Route to BASIS_TEAM when:
- Vendor **name change** (standalone, no bank/GST combined)
- Vendor **address / contact details / email / mobile** update
- Vendor **password reset** / login credentials
- BTS portal password / BTS role assignment
- SRM password reset for vendor user
- E-tender login validity extension
- Assigning BTS role to vendor (ABCD code)
- VMS vendor login reset

**Real BASIS ticket examples:**
```
"Vendor password reset"
"Name Change in Vendor Code ABCD09040"
"Addition of contact details in vendor code"
"Modification of Email address and Mobile"
"REQUEST FOR CHANGE IN VENDOR NAME & ADDRESS" ← standalone name/address = BASIS
"Reset BTS password"
"RESET VENDOR BTS PORTAL PASSWORD"
"reset the VMS Vendor login password"
"SRM Password reset of Vendor SRM User ID"
"Login Credentials for BTS for vendor"
"Extension of BTS validity"
```

### Route to SRM_TEAM when:
- Vendor-related issues **inside the SRM/Pranit portal**
- Transferring vendor to SRM/PRANIT portal
- Vendor code not working during award in SRM
- Vendor code and name mismatch in award utility
- Changing vendor ID in award/RFx context

**Real SRM ticket examples:**
```
"Transfer of Vendor to SRM/PRANIT portal"
"Vendor code not working during Award"
"Vendor code and Name mismatch error" ← in SRM/award context
"Change of vendor code in award details"
"Problem while transferring new Vendor" ← to PRANIT portal
```

### THE RULE:
```
Vendor + plant/PO/mapping                 → MM_TEAM
Vendor + bank/IFSC/GST/PAN/company code   → FICO_TEAM
Vendor + name/address/contact/password    → BASIS_TEAM
Vendor + SRM/Pranit/award/BTS portal      → SRM_TEAM
```

---

## SKILL 3 — THE INSPECTION LOT / QUALITY TRAP
### "Inspection lot" appears in MM, QM, and PM tickets

### Route to QM_TEAM when:
- Usage decision (UD) on inspection lot
- Reversal of usage decision
- QA32 errors (quality inspection t-code)
- Inspection lot not getting cleared / QI not cleared
- Opening an inspection lot for a decision
- Quality inspection error (standalone QM process)
- Inspection lot reset (in QM context, for re-inspection)

**Real QM ticket examples:**
```
"Unable to give usage decision"
"Inspection Lot UD Reversal"
"Error during QA32"
"QI is not getting cleared"
"Opening of inspection lot"
"Reg Quality inspection"
"inspection lot reset" ← if it's for re-inspection / UD purpose
```

### Route to MM_TEAM when:
- Inspection lot is **blocking a GR, MRC, or PO**
- Quality inspection error is **preventing a procurement step**
- "Quality Inspection for PO XXXXXXXXX"
- Inspection lot number not available for MRC
- "Missing inspection lot in MRC No."
- Quality clearance blocking goods receipt
- Not able to process GR due to quality check

**Real MM ticket examples:**
```
"Quality Inspection for PO 6900012524" ← PO context = MM
"Missing inspection lot in MRC No. 50017"
"Error in Inspection Lot clearance" ← blocking MRC/GR = MM
"Reset inspection lots" ← for MRC processing = MM
"Not able to reverse quality inspection" ← in GR/MRC context
"Inspection lot number not available" ← for goods receipt = MM
"issue in quality clearance" ← blocking procurement = MM
```

### Route to PM_TEAM when:
- Inspection lot **could not be generated** in context of a maintenance order
- Inspection lot blocking a **work order or permit**

**Real PM ticket examples:**
```
"Inspection lot could not be generated" ← maintenance work order context
"INSPECTION LOT HAS NOT BEEN GENERATED" ← in PM order context
```

### THE RULE:
```
Inspection lot + usage decision / QA32 / QI clearing   → QM_TEAM
Inspection lot + PO / MRC / GR / goods receipt         → MM_TEAM
Inspection lot + work order / permit / maintenance     → PM_TEAM
```

---

## SKILL 4 — THE WORKFLOW TRAP
### "Workflow" appears in MM, PM, BASIS, and SRM tickets

### Route to MM_TEAM when:
- Workflow for **SAP PO** (purchase order approval chain)
- Missing workflow / unreleased SAP PO
- Workflow issue for PO number (51xxxxxxxx, 52xxxxxxxx)
- Change of workflow for PO
- Workflow not triggering for PO

**Real MM ticket examples:**
```
"SAP PO workflow"
"Missing workflow / Un-released SAP PO 5200054925"
"Workflow issue for SAP PO-5100051249"
"Change of workflow for PO No. 5200054925"
"Request for PO workflow creation"
```

### Route to PM_TEAM when:
- Workflow for a **permit / PTW**
- Workflow for a **maintenance order**
- Not triggering of workflow for order
- Changing / deleting workflow of permit
- Retrigger the workflow of permit
- Workflow not showing after order return

**Real PM ticket examples:**
```
"Not trigger of workflow" ← for permit/order
"For changing of workflow of permit order"
"PERMIT NOT IN WORKFLOW"
"Retrigger the workflow of permit"
"Workflow Not Triggered for Order 101000393..."
"Deletion of Workflow" ← permit context
"Reg. re-triggering of ZG01 order in WF"
```

### Route to SRM_TEAM when:
- Workflow for an **SRM award / RFx**
- Approval workflow on PRANIT portal

### Route to BASIS_TEAM when:
- Workflow for **SAP user roles / authorizations**
- Approval workflow for roles (GRC)

**Real BASIS ticket example:**
```
"Approval Workflow for Roles - 121977"
```

### THE RULE:
```
Workflow + SAP PO number                  → MM_TEAM
Workflow + permit / PTW / order number    → PM_TEAM
Workflow + roles / GRC / authorization   → BASIS_TEAM
Workflow + award / RFx / SRM portal      → SRM_TEAM
```

---

## SKILL 5 — THE ESS TRAP
### "ESS" appears in HCM, BASIS, and INFRASTRUCTURE tickets

### Route to HCM_TEAM when:
- ESS functional issues: claims, tours, TA, leave, loan, PF
- Unable to track TA/TTA claim in ESS
- Tour tab not opening in ESS (functional)
- Error while submitting cafeteria benefits
- Unable to download PF statement in ESS
- ESS probation confirmation / deletion
- Reporting officer in ESS (org structure)
- Transfer order updation in ESS

**Real HCM ticket examples:**
```
"Unable to track TA and TTA claim in ESS"
"Unable to download PF statement in ESS"
"Error while Opening Tour Tab in ESS"
"Transfer order updation in ESS"
"E-medical Card is not created in ESS"
"Confirmation for probation ESS deletion"
```

### Route to BASIS_TEAM when:
- ESS **portal not loading / not opening** (technical)
- ESS tabs not visible (access/role issue)
- Wrong ESS screen displayed
- Unable to create claims in ESS (role/authorization issue)
- FIORI portal access
- Vendor BTS portal on ESS not opening
- ESS login access request

**Real BASIS ticket examples:**
```
"ESS portal not working"
"ESS Tabs are not visible"
"Wrong ESS Screen Displayed"
"ess portal properly opening problems"
"Assess to FIORI Portal"
"Unable to open Vendor BTS portal on ESS"
```

### Route to INFRASTRUCTURE_TEAM when:
- "SAP ESS issue" where SAP itself is not opening on the PC/browser
- "Sap .Ess issue" where the problem is device/browser level
- ESS tab not loading due to a network or browser issue

**Real INFRA ticket examples:**
```
"ESS tab not fully opening" ← device/browser = INFRA
"Sap .Ess issue" ← SAP on device = INFRA
```

### THE RULE:
```
ESS + claim / tour / TA / PF / leave / loan        → HCM_TEAM
ESS + portal not opening / tabs missing / access   → BASIS_TEAM
ESS + SAP not opening on laptop / browser issue    → INFRASTRUCTURE_TEAM
```

---

## SKILL 6 — THE SAP ON DEVICE TRAP
### "SAP not working" can mean INFRASTRUCTURE or BASIS

### Route to INFRASTRUCTURE_TEAM when:
- SAP is not **opening on a PC / laptop** (installation / device issue)
- SAP installation request
- SAP configuration on desktop / floor
- SAP install and configuration
- "Kindly upload SAP in my Laptop"
- SAP is running slow on PC (device performance)
- SAP patch file issue (IT update)
- Installing Checkpoint VPN + SAP (IT setup)

**Real INFRA ticket examples:**
```
"SAP is not opening in my PC"
"Kindly upload SAP in my Laptop"
"SAP Installation"
"SAP install and Configuration"
"SAP Configuration at -1 establishment room"
"Sap patch file issue"
"Installation of Checkpoint VPN & SAP"
"PC install and Configuration and also configure SAP"
```

### Route to BASIS_TEAM when:
- SAP **login / password** issue (credentials)
- SAP user ID locked
- SAP rights / T-code authorization
- User not able to perform a specific SAP action (authorization)
- ABAP runtime error
- Output device not defined in user master

**Real BASIS ticket examples:**
```
"SAP password reset"
"ABAP runtime error"
"Define an output device in your user master"
"Authorization required for closing SAP PO"
"Rights removed from backend"
```

### THE RULE:
```
SAP + install / configure / open on PC / slow device   → INFRASTRUCTURE_TEAM
SAP + login / password / rights / authorization        → BASIS_TEAM
```

---

## SKILL 7 — THE MIGO TRAP
### "MIGO" appears in both MM and PS tickets

### Route to MM_TEAM when:
- MIGO error in general procurement context
- Error during GR (goods receipt) for standard PO
- MIGO for standard purchase orders (51xx, 52xx, 68xx series)

### Route to PS_TEAM when:
- MIGO error for a **project-linked PO** (48xx series)
- "Error while processing MIGO in SAP PO 6900xxxxx" in project context
- GI (Goods Issue) from Project Stores
- Error during MIGO for WBS-linked items

**Real PS ticket examples:**
```
"MIGO error" ← when project PO is in context
"Error during Migo" ← in PS/WBS context
"Error while processing migo in SAP" ← project context
"Issue in GI from Project Stores"
"Regarding error in MIGO entry in PO 6900012524"
```

### THE RULE:
```
MIGO + standard procurement PO (51xx/52xx)   → MM_TEAM
MIGO + project PO (48xx) / WBS / stores      → PS_TEAM
```

---

## SKILL 8 — THE SRM vs MM PO TRAP
### Both teams deal with "SAP PO" — but differently

### Route to SRM_TEAM when:
- PO is **not visible / not fetching in PRANIT/SRM portal**
- GeM PO deletion from SAP PO
- Award utility linked to SAP PO
- SAP PO to Award number linking
- PO created from PRANIT
- Mismatch in sequence of items in SAP PO (SRM side)
- Changing vendor ID in PO through SRM portal
- Error while floating tender on SRM portal
- Roles for publishing tenders in CPP portal

**Real SRM ticket examples:**
```
"fetching of SAP PO from SRM issue"
"ERROR WHILE FETCHING PO FROM SRM"
"Deletion of GeM PO number from SAP PO"
"SAP PO 4100006542 created from PRANIT"
"Regarding Award Utility & SAP PO linking"
"SAP PO NOT VISIBLE" ← in SRM/Pranit context
"MISMATCH IN SEQUENCE OF ITEMS IN SAP PO" ← SRM-originated PO
"Error while floating tender on SRM portal"
```

### Route to MM_TEAM when:
- PO itself has errors: cannot be edited, wrong GL, tax code issues
- Invoice parking against the PO
- PO release / approval workflow
- Material code issues in PO
- GR / MRC against PO
- Error during PO creation (ME21N, ME22N)
- PO line item errors
- Budget exceeded on PO

### THE RULE:
```
PO + SRM/Pranit portal visibility/fetching/award   → SRM_TEAM
PO + creation/editing/parking/GR/errors            → MM_TEAM
```

---

## SKILL 9 — THE "BANK DELETION" TRAP
### Deleting a bank account can be MM or FICO

### Route to FICO_TEAM when:
- Deleting / removing old bank account from **vendor master**
- "Delete the old & add new bank data in VC (vendor code)"
- Deletion of bank details from vendor code

**Real FICO ticket examples:**
```
"DELETION OF BANK ACCOUNT" ← vendor master
"BANK DELETION VENDOR ABCD035294"
"ADDITION OF NEW BANK & DELETION OF OLD"
"Deletion of bank account details" ← vendor master context
```

### Route to MM_TEAM when:
- "Unable to delete bank account A001 in **PO**"
- "DELETION OF OLD BANK AC A000 MAPPED WITH PO"
- Bank account deletion that is blocking a PO transaction
- Bank account / invoice party mismatch in PO

**Real MM ticket examples:**
```
"Unable to delete bank account A001 in PO"
"DELETION OF OLD BANK AC A000 MAPPED WITH PO"
"Bank account deletion" ← in PO/invoice context
"Error in deletion of bank account in Bank..." ← PO context
```

### THE RULE:
```
Bank deletion + vendor code (ABCD)     → FICO_TEAM
Bank deletion + PO number              → MM_TEAM
```

---

## SKILL 10 — THE "EXTENSION" TRAP
### Extending codes can go to MM, FICO, or SD

### Extension of VENDOR CODE:
```
Extend vendor to PLANT / Purchase Org / storage location   → MM_TEAM
Extend vendor to COMPANY CODE                              → FICO_TEAM
```

**Key identifiers:**
- "plant 2304", "plant 3200", "purchase org" = MM
- "company code 1101", "co code 2201", "CC 1131" = FICO

### Extension of MATERIAL CODE:
```
Extending material code to plant   → MM_TEAM (always)
```

### Extension of CUSTOMER CODE:
```
Extend customer to COMPANY CODE                           → FICO_TEAM
Extend customer from 1136 to various company codes        → FICO_TEAM
Customer code creation (first time)                       → SD_TEAM
```

### Extension of SAP ROLES / USER validity:
```
Extension of role validity of user    → BASIS_TEAM
Extension of BTS validity             → BASIS_TEAM
Extension of e-tender login validity  → BASIS_TEAM
```

**Real examples:**
```
"Extension of role validity of user 23003..."   → BASIS_TEAM
"Extension of BTS validity"                     → BASIS_TEAM
"Extension of E-tender login validity"          → BASIS_TEAM
"Material code Extension to plant 2304"         → MM_TEAM
"Extension of vendor code ABCD090015 to co..."  → FICO_TEAM
"Extension of Customer code 2110620 to Co..."   → FICO_TEAM
```

---

## SKILL 11 — THE "ORDER CLOSING" TRAP
### "Order" and "order closing" can be MM, PM, or PS

### Route to PM_TEAM when:
- Maintenance order closing / completing
- "Order closing" standalone → it almost always means maintenance
- Z3 order not closed
- Not able to close order (with order number 103000xxxxxxx or 101000xxxxxxx)
- TECO of maintenance order
- Order number format: 10100xxxxxxx or 10300xxxxxxx

**Real PM ticket examples:**
```
"order closing"
"PM order closing"
"Not able to close Z3 order 103000519707"
"error during order closing"
"Reg. re-triggering of ZG01 order in WF"
"SAP Order No. 101000396322 Not closed"
```

### Route to PS_TEAM when:
- Closing a **project order** / WBS-linked order
- CJ order in project systems context

### Route to MM_TEAM when:
- "Unable to close SAP POs" (purchase orders, not maintenance orders)
- PO close/reversal issue

**Real BASIS ticket (edge case):**
```
"Authorization required for closing SAP PO"   → BASIS_TEAM
"Unable to close SAP Pos" ← if it's an authorization issue = BASIS
```

### THE RULE:
```
"Order closing" standalone or with 103xxx/101xxx   → PM_TEAM
"Close SAP PO" with authorization issue           → BASIS_TEAM
"Close SAP PO" with procurement error             → MM_TEAM
```

---

## SKILL 12 — THE "SCRAP" TRAP
### "Scrap vendor" sounds like it should go to MM, but it's SD

Scrap sales are handled as outbound sales transactions in SAP SD.
A "scrap vendor" is actually a **customer** in SAP for scrap buying.

### Route to SD_TEAM when:
- Scrap invoice
- Scrap sales order
- "Creation of Customer ID for Scrap Vendor"
- "Customer code for Vendor - ABCD" (in scrap context)
- CGST code in scrap order

**Real SD ticket examples:**
```
"Creation of New Customer for Scrap Sale"
"Creation of Customer ID for Scrap Vendor"
"Customer code for Vendor - ABCD086119" ← scrap vendor = customer in SD
"Issue in scrap invoice"
"CREATION OF CUSTOMER CODE FOR SCRAP INVOICE"
"Add remarks or other text in Scrap Invoice"
"Scrap Creation Invoice"
"CGST code needed in scrap order 47008284"
"Scrap sales order"
```

### THE RULE:
```
Scrap + invoice / customer / sale / order   → SD_TEAM (always)
Never MM for scrap
```

---

## SKILL 13 — THE "PR" TRAP
### "PR" (Purchase Requisition) can go to MM or PS

### Route to MM_TEAM when:
- PR release for standard procurement
- PR creation error
- PR line item issues in standard PO

### Route to PS_TEAM when:
- PR release for a **project-linked requisition**
- "PR number is not coming for a line item" in project context
- Error in PR release for WBS / project
- PR 7000xxxxxx number series (project PR)

**Real PS ticket examples:**
```
"Error in PR release" ← in project context
"PR number is not coming for a line item"
"PR 7000029589 for line item 10, 20 error"
```

### THE RULE:
```
PR + standard purchase   → MM_TEAM
PR + project / WBS       → PS_TEAM
```

---

## SKILL 14 — THE FICO "PARKING" EXCEPTION
### "Parking" almost always means MM, but there is one FICO exception

### Route to MM_TEAM: (99% of parking cases)
- "Park invoice", "parking error", "invoice parking" against a PO
- Credit note parking, credit memo parking
- Subsequent debit / credit parking

### Route to FICO_TEAM: (rare, 1% of cases)
- "Error while parking **customer** invoice under company code"
- This is an FI document posting for a customer, not a procurement PO

**Real FICO ticket example:**
```
"Error while parking customer invoice under company code"  → FICO_TEAM
"Error while parking customer invoice (Re...)"            → FICO_TEAM
```

### THE RULE:
```
Parking + PO / vendor invoice / credit note   → MM_TEAM
Parking + customer invoice / FI document      → FICO_TEAM
```

---

## SKILL 15 — THE "VENDOR NAME CHANGE" TRAP
### Name changes go to BASIS or FICO depending on context

### Route to BASIS_TEAM when:
- Vendor name change **standalone** (no bank or financial data involved)
- Change of address in vendor code
- Contact details, email, mobile number update
- "REQUEST FOR CHANGE IN VENDOR NAME & ADDRESS" ← standalone = BASIS
- Change name in SAP PO (name display issue, not master data)

**Real BASIS ticket examples:**
```
"Name Change in Vendor Code ABCD09040"
"REQUEST FOR CHANGE IN VENDOR NAME & ADDRESS"
"UPDATION OF VENDOR NAME IN SAP" ← standalone name = BASIS
"change in name of Vendor"
"Addition of contact details in vendor code"
"Modification of Email address and Mobile"
```

### Route to FICO_TEAM when:
- Vendor name change **combined with** bank/GST/PAN update
- Name + bank details together
- "PAN, GSTN, Bank details & Name update"
- Name correction combined with financial master data

**Real FICO ticket examples:**
```
"CHANGE OF VENDOR NAME & BANK DETAILS"   → FICO (combined with bank)
"PAN, GSTN, Bank details & Name update"  → FICO (combined with FI data)
"Correction in the name of Vendor: ABCD051..." + bank = FICO
```

### THE RULE:
```
Vendor name/address change only        → BASIS_TEAM
Vendor name + bank / GST / PAN         → FICO_TEAM
```

---

## SKILL 16 — THE HCM SALARY / DEDUCTION TRAP
### Financial-sounding HR issues are HCM, not FICO

These ticket descriptions sound like finance but are entirely HR:

### Route to HCM_TEAM when:
- Salary deduction issues
- TA/DA/conveyance deductions
- Conflict with absences/late deduction
- NPS (National Pension System) issues
- Interest rate on conveyance advance
- Wrong deductions of conveyance reimbursement
- Tour expense, tour claims, cafeteria benefits
- Medical claim status, medical reimbursement
- Unable to apply for multipurpose / conveyance advance
- PF statement (Provident Fund)
- Retired employee issues in ESS
- Digital service claim

**Real HCM ticket examples:**
```
"Salary Deduction against Staff advance"
"DA deduction"
"CONFLICT WITH ABSENCES/LATE DEDUCTION"
"Wrong Deductions of Conveyance Reimbursement"
"NPS"
"Interest Rate on Conveyance Advance"
"Error while submitting cafeteria benefits"
"REGARDING TOUR EXPENSE"
"Medical claim status"
"Unable to download PF statement in ESS"
"RETIRED EMPLOYEE"
"digital service claim regarding"
```

### THE RULE:
```
Any employee claim / deduction / reimbursement / PF / NPS / tour → HCM_TEAM
Never FICO for employee-facing financial issues
```

---

## SKILL 17 — THE BASIS EDGE CASES
### Unexpected things that go to BASIS

These are not obvious and will trip up the agent:

```
"ABAP runtime error"                          → BASIS_TEAM
"Define an output device in your user master" → BASIS_TEAM
"Unable to extract excel sheet from SAP"      → BASIS_TEAM
"unable to login erpapps.powergrid.in"        → BASIS_TEAM
"Change name in SAP PO of M/s Reva..."        → BASIS_TEAM (PO name display = user config)
"Reg - Group head release of SAP PO"          → BASIS_TEAM (authorization for release)
"Authorization required for closing SAP PO"   → BASIS_TEAM
"REGARDING REMOVAL OF ALL RIGHTS OF 2300..."  → BASIS_TEAM
"Clearing of Inspection LOT no: 1000..."      → BASIS_TEAM (when it's an authorization issue)
"Error while deleting service entry sheet"    → BASIS_TEAM (when it's auth-related)
"Regarding the storage location of Neemuc..." → BASIS_TEAM (org config, not MM mapping)
```

### THE RULE:
```
If the core issue is "not authorized / not permitted / rights" 
even inside another module's process → BASIS_TEAM
```

---

## SKILL 18 — THE PM EDGE CASES
### Unexpected things that go to PM_TEAM

```
"Non-confirming of operation in I2P software"  → PM_TEAM
"718 Line bay CVT not showing in CVT section"  → PM_TEAM (equipment/CVT reporting)
"PFTL plant code 3190 issue"                   → PM_TEAM (plant maintenance context)
"Error in CVT reporting in ZMM_reports"        → PM_TEAM (when CVT/maintenance context)
"BTS - Break-to-Start issues"                  → PM_TEAM
"PTW RETURN INITIATION DISCREPANCY"            → PM_TEAM
"Workflow Not Showing After Order Returning"   → PM_TEAM
```

---

## SKILL 19 — THE QM EDGE CASES
### When a PS/MM description mentions quality but should be QM

```
"Reset inspection lots" + MRC blocking    → MM_TEAM
"Reset inspection lots" + usage decision  → QM_TEAM
"Notification" (SAP QM notification)      → QM_TEAM
"Inspection lot could not be generated" in maintenance order → PM_TEAM
```

---

## SKILL 20 — THE SRM EDGE CASES
### Non-obvious SRM tickets

```
"User Id locked - PRANIT Portal"                   → SRM_TEAM (not BASIS)
"User IDs creation on PRANIT Portal"               → SRM_TEAM (not BASIS)
"Delivery date updated in PR but not getting..."   → SRM_TEAM (Pranit PR sync)
"roles for publishing tenders in CPP portal"       → SRM_TEAM
"Awarding of GEM contract in Pranit portal"        → SRM_TEAM
"GeM PO deletion from SAP PO"                      → SRM_TEAM (GeM = Govt e-Marketplace)
"Purchase group change in SRM"                     → SRM_TEAM
"Update Contact Details" in SRM portal context     → SRM_TEAM
"Award no. approval issue"                         → SRM_TEAM
```

Note: "User locked on PRANIT" goes to SRM, not BASIS.
BASIS handles SAP backend user IDs; SRM handles PRANIT/SRM portal user IDs.

---

## SKILL 21 — IT INFRASTRUCTURE EDGE CASES
### IT tickets that could be confused with SAP functional teams

```
"AutoCAD not working"       → INFRASTRUCTURE_TEAM (not BASIS, not MM)
"PSSE not working"          → INFRASTRUCTURE_TEAM (engineering software)
"PSSE installation access"  → INFRASTRUCTURE_TEAM
"ZTNA software install"     → INFRASTRUCTURE_TEAM (network/VPN)
"One Drive sync issue"      → INFRASTRUCTURE_TEAM
"One Drive Access Update"   → INFRASTRUCTURE_TEAM (not BASIS)
"Outlook not working"       → INFRASTRUCTURE_TEAM
"Scan to mail issue"        → INFRASTRUCTURE_TEAM
"Printer connection"        → INFRASTRUCTURE_TEAM
"Meeting room AV / projector" → INFRASTRUCTURE_TEAM
"DSC e-signer install"      → INFRASTRUCTURE_TEAM (device install, not SRM DSC migration)
"Adobe reader install"      → INFRASTRUCTURE_TEAM
"Google Earth install"      → INFRASTRUCTURE_TEAM
"pdf utility not working in Office Application" → INFRASTRUCTURE_TEAM
```

Note: DSC on a device = INFRASTRUCTURE; DSC migration in SRM/Pranit portal = SRM_TEAM.

---

## QUICK REFERENCE — THE 21 CONFUSING PATTERNS SUMMARY

| # | What You Hear | You Might Think | Correct Team | Key Differentiator |
|---|---|---|---|---|
| 1 | "Customer code extension" | SD | **FICO** | Extension to company code = FICO |
| 2 | "Customer bank details" | SD | **FICO** | FI-side bank update = FICO |
| 3 | "Customer GST/PAN update" | SD | **FICO or SD** | FI master = FICO; SD transaction = SD |
| 4 | "Vendor name change" | FICO | **BASIS** | Standalone name/address = BASIS |
| 5 | "Vendor password" | FICO | **BASIS** | All vendor portal access = BASIS |
| 6 | "BTS role / BTS password" | PM | **BASIS** | BTS portal access = BASIS |
| 7 | "Vendor name + bank" | BASIS | **FICO** | Combined with FI data = FICO |
| 8 | "Invoice parking" | FICO | **MM** | Parking in PO context = MM always |
| 9 | "Credit note/credit memo" | FICO | **MM** | Procurement reversal = MM |
| 10 | "Down payment" | FICO | **MM** | ME2DP/downpayment in PO = MM |
| 11 | "GL account" | FICO | **MM or FICO** | GL in PO = MM; GL in co. code = FICO |
| 12 | "Bank deletion" | FICO | **FICO or MM** | Vendor master = FICO; PO-mapped = MM |
| 13 | "Scrap vendor" | MM | **SD** | Scrap is a sales transaction = SD |
| 14 | "Inspection lot" | QM | **MM or PM** | Blocking GR/MRC = MM; maintenance = PM |
| 15 | "Workflow" | BASIS | **MM or PM** | PO workflow = MM; permit workflow = PM |
| 16 | "ESS not opening" | HCM | **BASIS or INFRA** | Access issue = BASIS; device = INFRA |
| 17 | "SAP not working" | BASIS | **INFRA** | Install/device issue = INFRA |
| 18 | "Order closing" | MM | **PM** | Maintenance order always PM |
| 19 | "MIGO error" | MM | **PS** | Project PO context = PS |
| 20 | "SAP PO in SRM" | MM | **SRM** | PO visibility in PRANIT = SRM |
| 21 | "User locked on PRANIT" | BASIS | **SRM** | Portal user = SRM; SAP user = BASIS |

---

*SKILL built from 40,574 tickets — 37,366 SAP ERP (Sample_Tickets.xlsx) + 3,208 IT (Sample_IT_related_tickets.xlsx). Every pattern is empirically verified from actual team assignments in the source data.*
