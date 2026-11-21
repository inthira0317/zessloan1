# zessloan1
25-09-2026 - 31-10-2026 - Ess Loan details 

# SAP ABAP – Employee Loan Balance and Interest Calculation

## 1. Project Overview

**Project Name:** Employee Loan Balance and Interest Calculation  
**Program Name:** `ZESSLOAN1`  
**Module:** SAP HCM – Human Capital Management  
**Object Type:** Executable Report  
**Transaction Code:** `ZESSLOAN1`  
**Development Language:** SAP ABAP  
**Output Technology:** Object-Oriented ALV (`CL_SALV_TABLE`)  
**Primary Infotype:** 0045 – Loans

### Project Description

The Employee Loan Balance and Interest Calculation report is an SAP HCM-based custom ABAP report developed to retrieve, process, calculate, and display employee loan information.

Employees may receive different types of loans from the organization, such as House Building Advance (HBA), Car Loan, Scooter Loan, and Multipurpose Loan.

Over time, employees repay their loans through payroll deductions. Therefore, the HR and Payroll teams need to identify the outstanding principal amount, accrued interest, opening interest, and total outstanding liability for each employee.

This report automates the process by retrieving loan information from SAP HCM infotypes, calculating outstanding balances using standard SAP function modules, processing repayment schedules, and displaying the final results through an ALV report.

The report also handles an additional business scenario involving intercompany employee transfers, where previous company loan interest must also be considered.

---

## 2. Business Requirement

### Business Scenario

Consider an organization that provides loans to its employees.

An employee receives a loan of ₹10,00,000 under the House Building Advance category.

Every month, a certain amount is deducted from the employee's salary towards loan repayment.

The HR department needs to identify:

1. What was the original loan amount?
2. When was the loan approved?
3. How much principal amount has been repaid?
4. What is the remaining loan balance?
5. How much interest has been accrued?
6. Is there any opening interest from previous records?
7. What is the total outstanding amount?

Manually collecting these details from multiple SAP infotypes and calculating the amounts is time-consuming.

To solve this problem, a custom ABAP report was implemented.

### Business Objectives

- Retrieve employee loan details from SAP HCM.
- Identify active loan records based on employee number and loan subtype.
- Retrieve outstanding principal balances.
- Calculate accrued interest based on loan repayment schedules.
- Retrieve opening interest from additional payment records.
- Consider previous company interest in applicable intercompany transfer cases.
- Calculate the final outstanding loan amount.
- Display all information in a structured ALV report.

---

## 3. Loan Categories and Subtype Mapping

The report supports different loan categories maintained in SAP HCM Infotype 0045.

| Subtype | Loan Description |
|---|---|
| 5300 | HBA Non-Executive Loan |
| 5301 | HBA Non-Executive Loan Interest |
| 5302 | HBA Executive Loan |
| 5303 | HBA Executive Loan Interest |
| 5304 | Car Loan |
| 5305 | Car Loan Interest |
| 5306 | Scooter Loan |
| 5307 | Scooter Loan Interest |
| 5308 | Multipurpose Loan |
| 5309 | Multipurpose Loan Interest |
| 5310 | Festival Loan Interest |

### Subtype Processing Logic

The program retrieves related loan and interest subtypes depending on the selected loan category.

For example:

- Selecting 5300 retrieves 5300, 5301, 5302, and 5303.
- Selecting 5304 retrieves 5304 and 5305.
- Selecting 5306 retrieves 5306 and 5307.
- Selecting 5308 retrieves 5308 and 5309.
- Selecting 5310 retrieves 5310.

This allows the report to process the relevant loan records together.

---

## 4. SAP Technical Architecture

### SAP Tables and Infotypes

The report uses multiple SAP HCM infotypes to retrieve employee loan and repayment information.

| Infotype / Table | Purpose |
|---|---|
| PA0045 | Employee loan information |
| PA0000 | Employee action information |
| PA0078 | Loan repayment information |
| PA0015 | Additional payments and opening interest |

### Important Fields

**PA0045 – Loan Information**

| Field | Description |
|---|---|
| PERNR | Personnel Number |
| SUBTY | Loan Subtype |
| OBJPS | Object Identification / Sequence |
| DATBW | Loan Approval Date |
| DARBT | Loan Amount Granted |
| BEGDA | Validity Start Date |
| ENDDA | Validity End Date |

**PA0078 – Repayment Information**

The program reads repayment information using the employee number, loan subtype, and loan sequence number.

**PA0000 – Employee Actions**

The program checks the action type `ZX` to identify intercompany transfers.

**PA0015 – Additional Payments**

The program retrieves relevant interest wage type records and accumulates their `BETRG` values.

---

## 5. Selection Screen

The report contains two mandatory input parameters.

| Parameter | Description |
|---|---|
| P_PERNR | Employee Personnel Number |
| P_SUBTY | Loan Subtype |

### Execution Example

Suppose the user enters:

- Employee Number: 10001234
- Loan Subtype: 5300

The program retrieves the corresponding active HBA loan records and their related interest subtypes.

The report then performs the balance and interest calculations for each retrieved loan record.

---

## 6. Detailed Technical Implementation

### Step 1: Retrieve Active Loan Records

**FORM:** `GET_LOAN_DETAILS`

The first step is to retrieve employee loan information from PA0045.

The program uses the following conditions:

- Employee number matches the entered PERNR.
- Loan record validity begins on or before the current date.
- Loan record validity ends on or after the current date.
- Loan subtype matches the selected loan category or its associated subtypes.

The retrieved records are stored in the internal table:

`GT_PA0045`

The report processes each loan record using:

`LOOP AT GT_PA0045 INTO GS_PA0045.`

This ensures that multiple active loan records can be processed individually.

### Step 2: Retrieve Employee and Loan Information

For each loan record, the program maps the relevant information into the output structure.

The following fields are populated:

- PERNR
- SUBTY
- OBJPS
- DATBW
- DARBT

These fields represent employee identification, loan category, sequence number, approval date, and original loan amount.

### Step 3: Calculate Outstanding Loan Balance

**Function Module:** `HR_GET_LOAN_BAL_PAID_AMT`

The program uses the standard SAP function module to retrieve the outstanding loan balance.

The function module receives:

| Parameter | Value |
|---|---|
| PERNR | Employee Number |
| LOAN_TYPE | Loan Subtype |
| SEQ_NR | Loan Sequence Number |
| BEGDA | Loan Start Date |
| ENDDA | Loan End Date |

The returned value is stored in:

`GV_BALANCE`

The value is assigned to:

`GS_OUTPUT-OPELO`

This represents the outstanding principal balance returned by the standard function module.

### Step 4: Calculate Accrued Interest

**FORM:** `GET_ACC_INTREST`

This is one of the important business logic sections of the project.

The accrued interest calculation involves two scenarios:

**Scenario A – Current Company Interest**

The program first calculates the accrued interest for the current loan record by calling:

`PERFORM CALCULATE_LOAN_INTEREST`

The calculated interest is stored in `GV_INTEREST`.

**Scenario B – Intercompany Transfer**

An employee may move from one company to another within the same organization.

In this scenario, the report checks whether the employee has an intercompany transfer action.

The program reads PA0000 using:

- PERNR
- MASSN = ZX
- BEGDA less than or equal to the current date

If a transfer action exists:

1. The action records are sorted by date in descending order.
2. The latest action is identified.
3. The previous loan sequence number is derived.
4. The previous loan record is retrieved from PA0045.
5. The previous company's accrued interest is calculated.
6. The previous interest is added to the current accrued interest.

This ensures that the report considers the previous loan record in the intercompany transfer scenario.

### Step 5: Detailed Loan Interest Calculation

**FORM:** `CALCULATE_LOAN_INTEREST`

This routine performs the main repayment-plan-based interest calculation.

The process is divided into several stages.

**Stage 1 – Retrieve Current Loan Balance**

The function module `HR_GET_LOAN_BAL_PAID_AMT` is called again to obtain the loan balance for the loan record being processed.

**Stage 2 – Retrieve Repayment Information**

The program retrieves repayment information from PA0078.

The selection conditions are:

- PERNR = Employee Number
- DLART = Loan Subtype
- OBJID = Loan Sequence Number

The records are stored in `LT_PA0078`.

**Stage 3 – Convert Loan Structure**

The PA0045 record is moved into the P0045 structure using:

`MOVE-CORRESPONDING IS_LOAN TO LS_P0045.`

This prepares the loan information for the repayment-plan function module.

**Stage 4 – Generate Repayment Plan**

The standard SAP function module:

`PCLO_BUILD_REPAYMENT_PLAN`

is used to generate the loan repayment schedule.

Important parameters:

| Parameter | Purpose |
|---|---|
| PS_P0045 | Loan information |
| PF_FDAY | Loan start date |
| PF_LDAY | Loan end date |
| PT_PAYM | Repayment information |
| PF_CURR | Currency – INR |
| PF_NODISP | Suppress display |

The generated repayment schedule is stored in:

`LT_LOAN_PLAN`

**Stage 5 – Apply Interest Start Date**

The program considers 01.04.2023 as the interest calculation start date.

Repayment-plan entries before this date are removed using:

`DELETE LT_LOAN_PLAN WHERE BEGDA < GC_OPEN_DATE.`

**Stage 6 – Match Loan Balance**

The program loops through the repayment schedule and compares:

- `LS_PLAN-LLB` – Loan balance in the repayment plan.
- `LV_BALANCE` – Current outstanding loan balance.

When both values match, the corresponding repayment-plan date is captured in `LV_MATCH_DATE`.

This date becomes the reference point for the next calculation step.

**Stage 7 – Calculate Interest Period**

The function module:

`RE_ADD_MONTH_TO_DATE`

adds one month to the matching date.

The resulting date is stored in:

`LV_INTEREST_TO`

**Stage 8 – Restrict Future Interest**

The program ensures that interest is not calculated beyond the current system date.

If the calculated interest date is greater than `SY-DATUM`, the date is restricted to the current date.

**Stage 9 – Accumulate Interest**

The repayment-plan internal table is processed again.

The program checks:

- Repayment-plan date is less than or equal to the interest calculation end date.
- Interest field `LID` is not initial.

The interest values are accumulated:

`LV_INTEREST = LV_INTEREST + LS_PLAN-LID.`

Finally, the calculated interest is added to the output interest variable.

### Step 6: Calculate Opening Interest

**FORM:** `GET_OPEN_INTEREST`

Opening interest is retrieved from PA0015.

The report maps each principal loan subtype to its corresponding interest wage type.

For example:

| Principal Loan | Interest Wage Type |
|---|---|
| 5300 | 5301 |
| 5302 | 5303 |
| 5304 | 5305 |
| 5306 | 5307 |
| 5308 | 5309 |
| 5310 | 5310 |

The program uses the standard function module:

`HR_READ_INFOTYPE`

Parameters:

- PERNR – Employee Number
- INFTY – 0015
- BEGDA – Loan Start Date
- ENDDA – Loan End Date

The retrieved records are stored in `LT_P0015`.

The program removes records that do not match the mapped interest wage types.

Finally, it accumulates the amount from the `BETRG` field.

`GV_OPEN_INTEREST = GV_OPEN_INTEREST + LS_P0015-BETRG.`

The final amount is stored in:

`GS_OUTPUT-OPEN_INT`

### Step 7: Calculate Total Outstanding Amount

After calculating the principal balance, accrued interest, and opening interest, the program calculates the final amount.

**Business Formula:**

TOTAL OUTSTANDING AMOUNT =

Outstanding Principal Balance  
+ Accrued Interest  
+ Opening Interest

**ABAP Implementation:**

`GS_OUTPUT-TOTAL = GS_OUTPUT-OPELO + GS_OUTPUT-ACCINR + GS_OUTPUT-OPEN_INT.`

The calculated output is appended to the internal table `GT_OUTPUT`.

---

## 7. ALV Report Development

**FORM:** `DISPLAY_ALV`

The report uses the Object-Oriented ALV approach through the standard SAP class:

`CL_SALV_TABLE`

The ALV displays all processed loan records in a structured tabular format.

### ALV Features

**1. Automatic Column Optimization**

The report automatically adjusts column widths.

`LO_COLUMNS->SET_OPTIMIZE( ABAP_TRUE ).`

**2. Standard ALV Toolbar**

All standard ALV functions are enabled.

`GR_ALV->GET_FUNCTIONS( )->SET_ALL( ABAP_TRUE ).`

**3. Custom Column Headings**

The report provides meaningful column descriptions.

Examples:

- Employee Number
- Loan Subtype
- Loan Sequence Number
- Approved Date
- Loan Amount
- Loan Balance
- Accrued Interest
- Opening Interest
- Total Balance Including Interest

**4. Display ALV**

The final output is displayed using:

`GR_ALV->DISPLAY( ).`

Exception handling is implemented for ALV-related errors using `CX_SALV_MSG` and `CX_SALV_NOT_FOUND`.

---

## 8. Expected ALV Output

The final report displays the following information:

| Field | Description |
|---|---|
| PERNR | Employee Number |
| SUBTY | Loan Subtype |
| OBJPS | Loan Sequence Number |
| DATBW | Approved Date |
| DARBT | Original Loan Amount |
| OPELO | Outstanding Principal |
| ACCINR | Accrued Interest |
| OPEN_INT | Opening Interest |
| TOTAL | Total Outstanding Amount |

### Illustrative Business Example

The following values are hypothetical examples to demonstrate the output structure.

| Description | Amount |
|---|---:|
| Original Loan Amount | ₹10,00,000 |
| Outstanding Principal | ₹6,00,000 |
| Accrued Interest | ₹45,000 |
| Opening Interest | ₹15,000 |
| **Total Outstanding Amount** | **₹6,60,000** |

The report displays the principal and interest components separately so that HR and Payroll users can understand the total outstanding liability.

---

## 9. End-to-End Processing Architecture

```text
             USER INPUT
                  |
                  v
        Enter PERNR and SUBTY
                  |
                  v
       Retrieve Active PA0045
                  |
                  v
       Process Each Loan Record
                  |
          +-------+-------+
          |               |
          v               v
     Loan Details    Loan Balance
                          |
                          v
                HR_GET_LOAN_BAL_PAID_AMT
                          |
                          v
                  Accrued Interest
                          |
                          v
                Repayment Plan (PA0078)
                          |
                          v
                PCLO_BUILD_REPAYMENT_PLAN
                          |
                          v
                Match Balance and Date
                          |
                          v
                    Sum LID
                          |
                          v
                 Opening Interest
                          |
                          v
                    Read PA0015
                          |
                          v
                 Calculate Total
                          |
                          v
                  Append GT_OUTPUT
                          |
                          v
                 CL_SALV_TABLE
                          |
                          v
                   FINAL ALV
```

---

## 10. ABAP Technical Concepts Demonstrated

| Technical Area | Implementation |
|---|---|
| Report Programming | Executable ABAP Report |
| Selection Screen | PARAMETERS |
| Database Retrieval | SELECT and SELECT SINGLE |
| Internal Tables | Standard Internal Tables |
| Data Processing | LOOP AT, READ TABLE |
| Field Mapping | MOVE-CORRESPONDING |
| Modular Programming | FORM and PERFORM |
| Function Modules | Standard SAP HR Function Modules |
| Date Processing | RE_ADD_MONTH_TO_DATE |
| Repayment Processing | PCLO_BUILD_REPAYMENT_PLAN |
| Infotype Processing | PA0045, PA0000, PA0078, PA0015 |
| Data Accumulation | Arithmetic Operations |
| ALV Development | CL_SALV_TABLE |
| Exception Handling | TRY, CATCH |

---

## 11. Key Learning Outcomes

Through this project, the following practical SAP ABAP concepts were implemented:

- Working with SAP HCM loan management and employee infotypes.
- Retrieving and processing multiple employee loan records.
- Integrating standard SAP function modules into custom reports.
- Understanding loan repayment schedules.
- Implementing business logic for accrued interest calculation.
- Handling intercompany transfer scenarios.
- Reading additional payment records and processing wage types.
- Working with date-based calculations.
- Performing internal table operations and data accumulation.
- Developing an Object-Oriented ALV report using `CL_SALV_TABLE`.


## 13. Project Summary

The Employee Loan Balance and Interest Calculation report demonstrates the implementation of a practical SAP HCM reporting requirement using ABAP.

The report integrates multiple SAP HR infotypes, standard function modules, repayment schedule processing, interest calculations, intercompany transfer handling, and Object-Oriented ALV reporting.

It provides a consolidated view of employee loan balances and interest components, helping HR and Payroll teams analyze outstanding employee loan liabilities.

## Author

**SAP ABAP Developer**

GitHub: https://github.com/inthira0317

LinkedIn: https://linkedin.com/in/bminthira
