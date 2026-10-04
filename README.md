# Smart Incident Escalation & Problem Correlation System

A ServiceNow ITSM automation portfolio project for **P1/P2 incident escalation, SLA visibility, and recurring-incident correlation**.

## Project Overview

This project demonstrates an end-to-end ServiceNow workflow that:

- Detects P1/P2 incidents and automatically escalates them.
- Sends escalation notifications through Flow Designer and Notifications.
- Tracks critical incidents using SLA definitions and Task SLA records.
- Detects recurring incidents using the same Configuration Item (CI), Category, and a 60-minute time window.
- Uses `GlideAggregate` to count matching incidents.
- Creates a Problem when the recurrence threshold is reached.
- Reuses an existing active Problem instead of creating duplicates.
- Associates qualifying incidents with the Problem.
- Provides a **Declare Major Incident** UI Action with meaningful work-note validation.
- Provides operational reports and a dashboard.

> **Threshold:** 3 matching incidents for the same CI and Category within 60 minutes.

## Architecture

```text
Incident Created / Updated
          |
          v
     Check Priority
       /       \
     P1/P2     Other
       |         |
       v         v
  Escalation   Normal
       |
       v
Notification + SLA
       |
       v
Correlation Check
       |
       v
Same CI + Category
+ Within 60 Minutes
       |
       v
GlideAggregate Count
       |
       +------------------+
       |                  |
    Count < 3          Count >= 3
       |                  |
       v                  v
No Correlation      Check Active Problem
                           |
                    +------+------+
                    |             |
                  Exists        None
                    |             |
                    v             v
               Reuse Problem  Create Problem
                    |             |
                    +------+------+
                           |
                           v
                    Link Incidents
                           |
                           v
                    Reports/Dashboard
```

## Technology Stack

- ServiceNow Personal Developer Instance (PDI)
- JavaScript
- Incident Management
- Problem Management
- Business Rules
- Script Includes
- GlideRecord
- GlideAggregate
- Flow Designer
- SLA / Task SLA
- Notifications
- UI Actions
- Reports
- Dashboards

## Main Components

### 1. P1/P2 Escalation

**Business Rule:** `Auto Escalate P1 P2 Incidents`

For P1 and P2 incidents, the Business Rule:

- Sets the custom `Escalation Flag`.
- Adds an automatic work note.
- Runs on Incident insert/update when the priority condition is met.

P1, P2, and P3 scenarios were tested.

### 2. Flow Designer

**Flow:** `Smart Incident P1 P2 Notification`

The flow is triggered by Incident creation/update and checks for P1/P2 priority before sending the escalation email.

### 3. Notifications

**Notification:** `P1 P2 Incident Escalation Notification`

The notification is sent when a P1/P2 incident has the escalation flag enabled.

### 4. SLA Management

Project-specific SLA definitions:

- `Smart P1 Resolution SLA`
- `Smart P2 Resolution SLA`

Task SLA records were used to verify SLA lifecycle behavior including attachment, completion, breach visibility, and cancellation.

The short durations used during testing are **PDI test configurations**, not production SLA commitments.

### 5. Correlation Engine

**Script Include:** `IncidentCorrelationUtil`

Main methods:

```text
getMatchingIncidentIds()
findMatchingIncidents()
checkOrCreateProblem()
linkIncidentsToProblem()
```

The correlation logic uses:

- `GlideAggregate` for matching-incident counts.
- `GlideRecord` for incident retrieval and Problem operations.
- Same CI.
- Same Category.
- Last 60 minutes.
- Threshold of 3 incidents.

### 6. Problem Creation and Reuse

When the threshold is reached:

```text
Existing active matching Problem?
        |
   +----+----+
   |         |
  YES        NO
   |         |
 Reuse     Create
   |         |
   +----+----+
        |
        v
 Link qualifying incidents
```

Duplicate prevention was tested by confirming that an existing active Problem is reused rather than creating another Problem.

### 7. Major Incident UI Action

**UI Action:** `Declare Major Incident`

The action requires meaningful work notes before declaration.

Example work note:

```text
Major incident confirmed after reviewing the business impact and escalation requirements.
```

The successful test confirmed the Major Incident declaration and recorded the activity/work note.

## Custom Data Model

A custom Incident reference field was added:

```text
Label: Problem
Column: u_problem
Type: Reference
Reference: Problem [problem]
```

This connects Incident records to the standard ServiceNow Problem table.

## Reports

The project includes:

### Smart - Open P1 P2 Incidents

Shows open P1/P2 incidents.

### Smart - SLA Status

Uses the Task SLA table and groups SLA records by stage.

### Smart - Correlated Incidents

Shows incidents whose Problem reference is populated.

### Smart - Correlated Problems

Shows Problems created by the recurring-incident correlation process.

## Dashboard

**Dashboard:** `Smart Incident Escalation Dashboard`

The dashboard provides visibility into:

- P1/P2 incidents
- Open incidents
- Problems
- Active SLAs
- Correlated incidents

Dashboard values represent the PDI test environment.

## Testing

| Test Case | Expected Result | Result |
|---|---|---|
| P3 incident | No P1/P2 escalation | PASS |
| P2 incident | Escalation + notification/SLA | PASS |
| P1 incident | Escalation + notification/SLA | PASS |
| 1–2 matching incidents | No Problem correlation | PASS |
| 3 matching incidents | Correlation threshold reached | PASS |
| 4th matching incident | Existing Problem reused | PASS |
| Existing active Problem | Reuse, no duplicate | PASS |
| Different CI | No correlation | PASS |
| Different Category | No correlation | PASS |
| Outside 60-minute window | Not counted | PASS |
| SLA completion | SLA completes | PASS |
| SLA breach | Breach visible in SLA reporting | PASS |
| SLA cancellation | Cancellation represented in Task SLA | PASS |
| Major Incident without meaningful notes | Action blocked | PASS |
| Major Incident with meaningful notes | Action succeeds | PASS |
| Notification | Email generated | PASS |
| Flow Designer | Test execution successful | PASS |

## Project Evidence

Implementation evidence was captured for:

- Incident forms and P1/P2 test records
- Escalation Business Rule
- Script Include
- GlideAggregate testing
- Flow Designer
- SLA definitions and Task SLA records
- Problem records
- Related incidents
- Major Incident UI Action
- Work-note validation
- Reports
- Dashboard
- Positive and negative test cases

## Project Documentation

Detailed implementation documentation is available in:

```text
PROJECT_DOCUMENTATION.md
```

It contains the architecture, implementation details, test summary, design decisions, limitations, and verified technology stack.

## Limitations

- Developed and tested in a ServiceNow Personal Developer Instance.
- SLA durations used during testing are project-specific test configurations.
- No enterprise production deployment is claimed.
- No AI component is implemented.
- No percentage performance improvement is claimed because no controlled benchmark was performed.
- No Slack, Teams, REST, OAuth, Node.js, Python, WebSockets, or external monitoring integration is claimed as part of the core project.
- Dashboard and report values represent the PDI test environment.

## Final Outcome

The project demonstrates practical ServiceNow development across:

**Incident Management → P1/P2 Escalation → Notification → SLA Tracking → Recurring Incident Correlation → Problem Creation/Reuse → Incident Association → Reports → Dashboard**

## Author

**Lokesh Avulapati**

ServiceNow ITSM | JavaScript | Business Rules | Script Includes | GlideRecord | GlideAggregate | Flow Designer | SLA | Problem Management
