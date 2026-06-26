# Vacation Request Management Automation

This project automates the ingestion, validation, logging, and follow-up tracking of employee vacation requests using UiPath Studio and integration with Gmail and Excel.

## Prerequisites

* UiPath Studio with the following package dependencies:
  * UiPath.Mail.Activities
  * UiPath.Excel.Activities
  * UiPath.Testing.Activities (for environment runtimes)
* A Google Gmail account with OAuth authentication scopes configured for reading, composing, and sending emails.
* Microsoft Excel (or access to .xlsx files locally).

## Project Architecture

The automation is modularized into four primary workflow components invoked by `Main.xaml`:

1. **ExtractEmail.xaml**: Polls the inbox, matches target unread request emails, and parses text payloads.
2. **ConflictCheck.xaml**: Centralized data validation engine verifying string formats and date chronological alignment.
3. **ManagerReview.xaml**: Formats data strings and logs fresh requests as Pending to the database workbook.
4. **SendMail.xaml**: Periodic review engine monitoring row resolution status, employee messaging, and manager escalation intervals.

---

## Configuration and Setup Parameters

To run this project, the following variables and arguments must be configured inside UiPath Studio.

### Global Variables (Main.xaml)

These variables coordinate data movement between individual sub-workflows across the main orchestration loop:

| Variable Name | Type | Description / Default Value |
| :--- | :--- | :--- |
| `main_EmployeeID` | String | Stores parsed employee identification strings. |
| `main_StartDate` | DateTime | Stores processed vacation start timeline. |
| `main_EndDate` | DateTime | Stores processed vacation end timeline. |
| `main_IsConflict` | Boolean | True if input criteria or dates fail centralized validation rules. |

### Sub-Workflow Arguments

#### 1. ExtractEmail.xaml Arguments
* `out_EmployeeID` (Direction: Out, Type: String): Passes parsed ID to Main.
* `out_StartDate` (Direction: Out, Type: DateTime): Passes converted start date to Main.
* `out_EndDate` (Direction: Out, Type: DateTime): Passes converted end date to Main.

#### 2. ConflictCheck.xaml Arguments
* `in_EmployeeID` (Direction: In, Type: String): Binds to `main_EmployeeID`.
* `in_StartDate` (Direction: In, Type: DateTime): Binds to `main_StartDate`.
* `in_EndDate` (Direction: In, Type: DateTime): Binds to `main_EndDate`.
* `out_IsConflict` (Direction: Out, Type: Boolean): Returns verification state to `main_IsConflict`.

#### 3. ManagerReview.xaml Arguments
* `in_EmployeeID` (Direction: In, Type: String)
* `in_StartDate` (Direction: In, Type: DateTime)
* `in_EndDate` (Direction: In, Type: DateTime)

---

## Local Environmental Parameters

### Local Workbook Storage
* **File Name**: `VacationDatabase.xlsx`
* **Target Worksheet**: `MasterLog`
* **Required Column Schema**:
  * Column A: `EmployeeID`
  * Column B: `StartDate`
  * Column C: `EndDate`
  * Column D: `Status`
  * Column E: `Timestamp`
  * Column F: `ManagerComments`
  * Column G: `Resolved`

*Note: The Excel file must reside in the root directory of the project folder (the same directory containing project.json) for relative paths to resolve correctly.*

### Local Email Activity Settings
* **Account Integration**: Connected via Google Workspace / Gmail integration matching the workflow account profile.
* **Inbound Filter**: The filter criteria inside `ExtractEmail.xaml` must be set to look for unread subjects containing `"vacation request"`.
* **Activity Properties**: Ensure `Mark As Read` and `Unread Only` properties are checked True within the email loop.

---

## Deployment and Execution

1. Close the local `VacationDatabase.xlsx` file on your desktop machine to avoid database execution write blocks.
2. Open the project root folder inside UiPath Studio.
3. Open `Main.xaml`.
4. Click Run or Debug to initiate the execution processing chain.
