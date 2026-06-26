# Vacation Request Management Automation

This project automates the ingestion, validation, logging, and follow-up tracking of employee vacation requests using UiPath Studio, Gmail, and Excel.

## Prerequisites

* UiPath Studio with the following package dependencies:
  * UiPath.Mail.Activities
  * UiPath.Excel.Activities
  * UiPath.GSuite.Activities
* A Google Gmail account with OAuth authentication scopes configured for reading, composing, and sending emails.
* Microsoft Excel database workbook.

## Project Architecture

The automation is split into four sub-workflows invoked by `Main.xaml`:

1. **ExtractEmail.xaml**: Polls the inbox, filters target emails, and extracts text inputs.
2. **ValidationCheck.xaml**: Evaluates extracted values against formatting and chronological constraints.
3. **ManagerReview.xaml**: Logs new pending vacation requests directly into the spreadsheet.
4. **SendMail.xaml**: Reviews row resolution status and automates employee notifications and manager escalation tracking.

---

## Format of Mail

Mail sent to the robot must be formatted following to ensure that no validation errors are thrown. 

**Subject:** Vacation Request

**EmployeeID:** EMP[XXX]

**Start Date Vacation:** 0[DD/MM/YYYY]

**End Date Vacation** [DD/MM/YYYY]

---

## Configuration and Setup Parameters

### Global Variables (Main.xaml)

These variables track and transfer data across the main sequence loop container:

| Variable Name | Type | Description |
| :--- | :--- | :--- |
| `main_EmployeeID` | String | Stores extracted employee ID string. |
| `main_StartDate` | DateTime | Stores processed vacation start date. |
| `main_EndDate` | DateTime | Stores processed vacation end date. |
| `main_IsConflict` | Boolean | Flags validation or logical date errors. |

### Workflow Arguments

#### 1. ExtractEmail.xaml Arguments
* `out_EmployeeID` (Out, String): Passes extracted employee ID to Main.
* `out_StartDate` (Out, DateTime): Passes extracted start date to Main.
* `out_EndDate` (Out, DateTime): Passes extracted end date to Main.

#### 2. ValidationCheck.xaml Arguments
* `in_EmployeeID` (In, String): Receives `main_EmployeeID`.
* `in_StartDate` (In, DateTime): Receives `main_StartDate`.
* `in_EndDate` (In, DateTime): Receives `main_EndDate`.
* `out_IsConflict` (Out, Boolean): Returns evaluation status to `main_IsConflict`.

#### 3. ManagerReview.xaml Arguments
* `in_EmployeeID` (In, String): Employee identification token.
* `in_StartDate` (In, DateTime): Confirmed leave start time.
* `in_EndDate` (In, DateTime): Confirmed leave end time.
* `in_IsConflict` (In, Boolean): Initial validation flag tracking state.
* `str_StartDate` (In, String): Unused formatting container argument.
* `str_EndDate` (In, String): Unused formatting container argument.

#### 4. SendMail.xaml Arguments
* No external parameters needed. The process reads log metrics directly from the workbook database.

---

### Local Sequence Variables

These local variables handle isolated tasks within their specific workflow files:

#### ExtractEmail.xaml Local Variables
* `str_MailBody` (String): Stores the raw body text of the current email.
* `str_EmployeeID` (String): Temporary text placeholder for regex matching.
* `str_StartDateText` (String): Extracted text matching the start date string.
* `str_EndDateText` (String): Extracted text matching the end date string.
* `dt_StartDate` (DateTime): Temporal date variable checking format parsing validation.
* `dt_EndDate` (DateTime): Temporal date variable checking format parsing validation.

#### ValidationCheck.xaml Local Variables
* `dt_DataTable` (DataTable): Structural variable data frame placeholder.

#### ManagerReview.xaml Local Variables
* `dt_dataTable` (DataTable): Unused schema tracking data frame.
* `Resolved` (String): State string tracking closed records.
* `Comment` (String): Standard commentary tracking placeholder.
* `dt_Log` (DataTable): Holds read ranges from the MasterLog worksheet database.
* `int_RowIndex` (Int32): Tracks target write coordinates inside the loop context.
* `str_CurrentStatus` (String): Tracking token representing manager selection.
* `dt_StatusCheck` (DataTable): Temporary validation table for updates.

#### SendMail.xaml Local Variables
* `dt_BatchLog` (DataTable): Houses read information from the spreadsheet database sheet.
* `int_CurrentLine` (Int32): Translates data row index into active Excel rows.

---

## Workbook Schema Parameters

* **Path Location**: Root directory folder of the project.
* **File Target**: `VacationDatabase.xlsx`.
* **Worksheet Target**: `MasterLog`.
* **Column Positions**:
  * Column A: EmployeeID
  * Column B: StartDate
  * Column C: EndDate
  * Column D: Status
  * Column E: Timestamp
  * Column F: ManagerComments
  * Column G: Resolved
