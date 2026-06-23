# Employee Vacation Request Automation

## Project Overview
This automation serves as an end-to-end management system for incoming employee time-off requests. It processes emails, references an Excel database to ensure scheduling availability, updates statuses, and coordinates with management via interactive email notification streams.

### Core Workflow Engine Execution Flow
1. **Extract Email:** Parses incoming email metadata to gather requests (`EmployeeID`, `StartDate`, `EndDate`).
2. **Conflict Check:** Validates against Excel spreadsheet which acts as database. 
3. **Manager Review & Log:** Appends a `Pending` transactional record to Excel, dispatches an approval request email, and polls the file for an active human-in-the-loop decision every 10 seconds.
4. **Final Notification:** Emails confirmation or rejection alerts back to the requesting employee based on the manager's live update.

## Prerequisites & System Requirements

To execute or modify this project successfully, ensure the following environments and dependencies are configured:

### 1. Software & Package Dependencies
* **UiPath Studio 2024.x / 2026.x** (Windows Compatibility Profile)
* **UiPath.Excel.Activities** (To process local database sheets)
* **UiPath.GSuite.Activities** (To orchestrate Gmail integrations via Integration Service)
* **UiPath.Testing.Activities** (To run BDD verification test cases)

### 2. Local File Assets
* **`VacationDatabase.xlsx`**: Must be located directly inside the project root folder directory. It requires a worksheet named **`MasterLog`** with the following structural schema header columns:

| Column A | Column B | Column C | Column D | Column E |
| :--- | :--- | :--- | :--- | :--- |
| `EmployeeID` | `StartDate` | `EndDate` | `Status` | `Timestamp` |

## Variables & Arguments Reference Model

### Global / Main Orchestration Variables (`Main.xaml`)
These variables maintain state across individual workflow components called within the primary execution chain:

* **`main_EmployeeID`** `(String)`: Holds the unique alpha-numeric text identifying the employee.
* **`main_StartDate`** `(DateTime)`: Tracks the scheduled start date boundary of the request.
* **`main_EndDate`** `(DateTime)`: Tracks the scheduled end date boundary of the request.
* **`main_IsConflict`** `(Boolean)`: Flags whether a timeline intersection error was uncovered.

### Component Architecture Specs

#### 1. `ExtractEmail.xaml`
* **Purpose:** Simulates or connects to email clients to harvest application data bounds.
* **Arguments:**
  * `out_EmployeeID` `(OutArgument<String>)` → Maps back to `main_EmployeeID`.
  * `out_StartDate` `(OutArgument<DateTime>)` → Maps back to `main_StartDate`.
  * `out_EndDate` `(OutArgument<DateTime>)` → Maps back to `main_EndDate`.

#### 2. `ConflictCheck.xaml`
* **Purpose:** Queries database history to spot concurrent scheduling collisions.
* **Arguments:**
  * `in_EmployeeID` `(InArgument<String>)`
  * `in_StartDate` `(InArgument<DateTime>)`
  * `in_EndDate` `(InArgument<DateTime>)`
  * `out_IsConflict` `(OutArgument<Boolean>)`
* **Local Variables:**
  * `dt_DataTable` `(DataTable)`: Temporarily buffers structural Excel logs inside system memory.

#### 3. `ManagerReview.xaml`
* **Purpose:** Writes row updates, handles email distribution alerts, and continuously checks workbook status fields for explicit supervisor inputs.
* **Arguments:**
  * `in_EmployeeID` `(InArgument<String>)`
  * `in_StartDate` `(InArgument<DateTime>)`
  * `in_EndDate` `(InArgument<DateTime>)`
  * `in_IsConflict` `(InArgument<Boolean>)`
* **Local Variables:**
  * `dt_Log` `(DataTable)`: Computes active sheet boundaries to predict row offsets.
  * `int_RowIndex` `(Int32)`: Points accurately to the next open row coordinate (`Rows.Count + 2`).
  * `str_CurrentStatus` `(String)`: Polling state tracking token. *Default value:* `"Pending"`.
  * `dt_StatusCheck` `(DataTable)`: Reads single cells dynamically on loops to intercept modifications.

