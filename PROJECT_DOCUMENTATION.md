# Smart Incident Escalation & Problem Correlation System

## 1. Project Overview

**Platform:** ServiceNow Personal Developer Instance (PDI)\
**Primary Focus:** Incident Management + Problem Management +
Automation + SLA

This project automates P1/P2 incident escalation, notification, SLA
tracking, and recurring-incident correlation in ServiceNow.

The core correlation rule is:

-   Same Configuration Item (CI)
-   Same Category
-   Created within the previous 60 minutes
-   At least 3 matching incidents

When the threshold is reached, the system checks for an existing active
Problem. If one exists, qualifying incidents are associated with it;
otherwise, a new Problem is created and the relevant incidents are
linked.

## 2. Business Problem

High-priority incidents may require rapid escalation, notification, and
SLA monitoring. Repeated incidents affecting the same CI and category
can remain separate even when they may share an underlying cause.

The project combines Incident Management, SLA tracking, workflow
automation, and Problem Management correlation into one ServiceNow
implementation.

## 3. Architecture

``` text
Incident Created / Updated
          |
          v
   Check Priority
      /           P1/P2     Other
      |         |
 Escalation   Normal
      |
 Notification + SLA
      |
 Correlation Check
      |
 Same CI + Category
 + within 60 minutes
      |
 GlideAggregate Count
      |
   Count >= 3?
    /         No         Yes
  |           |
Stop      Check Active Problem
              |
        +-----+-----+
        |           |
     Exists       None
        |           |
     Reuse       Create
        \           /
         \         /
          Link Incidents
               |
         Reports/Dashboard
```

## 4. ServiceNow Components

  Component            Purpose
  -------------------- ----------------------------------------
  Incident             Primary operational record
  Problem              Underlying recurring issue
  Configuration Item   Correlation key
  Business Rules       Server-side escalation and correlation
  Script Include       Reusable correlation logic
  GlideRecord          Retrieve/update records
  GlideAggregate       Count matching incidents
  Flow Designer        P1/P2 notification workflow
  SLA Definitions      P1/P2 resolution tracking
  Notifications        Escalation alerts
  UI Action            Declare Major Incident
  Reports              Operational monitoring
  Dashboard            Combined visibility

## 5. Incident and Escalation

The Incident table uses Number, Caller, Priority, Category,
Configuration Item, State, Short Description, Work Notes, Escalation
Flag, and Problem.

A custom Incident reference field **Problem (`u_problem`)** associates
incidents with the standard Problem table.

### P1/P2 Business Rule

**Name:** `Auto Escalate P1 P2 Incidents`\
**Table:** Incident\
**Timing:** Before\
**Operations:** Insert and Update\
**Condition:** Priority P1 or P2

The rule sets the Escalation Flag and adds an automatic work note.

P1, P2, and P3 cases were tested. P1/P2 escalated; P3 did not.

## 6. Flow Designer and Notifications

**Flow:** `Smart Incident P1 P2 Notification`

The flow triggers for P1/P2 Incident creation or update and sends an
escalation email.

**Notification:** `P1 P2 Incident Escalation Notification`

The notification is conditioned on P1/P2 plus the Escalation Flag and
was verified with a P1 test.

## 7. SLA Management

Project-specific definitions were created:

-   `Smart P1 Resolution SLA`
-   `Smart P2 Resolution SLA`

Task SLA records were inspected to verify attachment and lifecycle
behavior.

Verified behaviors:

-   SLA attachment
-   In progress state
-   Completion
-   Breach visibility
-   Cancellation

Short project-specific durations were used during PDI testing to make
lifecycle tests practical. These are test configurations, not production
SLA commitments.

## 8. Correlation Engine

**Script Include:** `IncidentCorrelationUtil`

Main methods:

-   `getMatchingIncidentIds()`
-   `findMatchingIncidents()`
-   `checkOrCreateProblem()`
-   `linkIncidentsToProblem()`

### GlideAggregate

Counts incidents matching the same CI and Category within the last 60
minutes.

### GlideRecord

Retrieves matching incidents, searches for active Problems, creates
Problems, and associates incidents.

### Threshold

``` text
3 matching incidents / 60 minutes
```

The correlation Business Rule stops when the count is below 3.

## 9. Problem Creation, Reuse, and Duplicate Prevention

The engine first searches for an active Problem matching the same CI and
Category.

-   Existing active Problem → reuse it.
-   No matching active Problem → create a new Problem.
-   Qualifying incidents → link to the Problem.
-   Existing incident Problem references are not overwritten
    unnecessarily.

Tests verified Problem creation, existing Problem reuse, fourth-incident
reuse, and duplicate prevention.

## 10. Correlation Tests

  Scenario                   Result
  -------------------------- --------
  1--2 matching incidents    PASS
  3 matching incidents       PASS
  4th matching incident      PASS
  Existing active Problem    PASS
  Different CI               PASS
  Different Category         PASS
  Outside 60-minute window   PASS

## 11. Declare Major Incident UI Action

**UI Action:** `Declare Major Incident`

The action requires meaningful work notes.

-   Empty/insufficient work notes → action blocked.
-   Meaningful work notes → action allowed.
-   Successful declaration → activity/work note recorded.

The positive test used:

> Major incident confirmed after reviewing the business impact and
> escalation requirements.

The action successfully declared the incident as a Major Incident.

## 12. Reports

The project includes:

1.  **Smart - Open P1 P2 Incidents**
2.  **Smart - SLA Status**
3.  **Smart - Correlated Incidents**
4.  **Smart - Correlated Problems**

The SLA Status report uses the Task SLA table and groups records by SLA
stage, including In progress, Completed, Cancelled, Paused, Breached,
and Achieved where applicable.

The Correlated Incidents report filters incidents whose Problem
reference is populated.

The Correlated Problems report identifies Problems generated by the
recurring-incident correlation process.

## 13. Dashboard

**Dashboard:** `Smart Incident Escalation Dashboard`

Dashboard visibility includes:

-   P1/P2 incidents
-   Open incidents
-   Problems
-   Active SLAs
-   Correlated incidents

The values represent the PDI test environment and are not production
performance metrics.

## 14. Testing Summary

### Positive

-   P1 escalation
-   P2 escalation
-   P1/P2 SLA attachment
-   SLA completion
-   SLA breach
-   SLA cancellation
-   Three-incident correlation
-   Problem creation
-   Existing Problem reuse
-   Fourth-incident association
-   Major Incident declaration
-   Notifications
-   Flow Designer
-   Reports
-   Dashboard

### Negative / Edge

-   P3 does not trigger P1/P2 escalation.
-   Different CI does not correlate.
-   Different Category does not correlate.
-   Outside the 60-minute window does not count.
-   Duplicate active Problems are prevented.
-   Major Incident action is blocked without meaningful work notes.

## 15. Evidence

Screenshots were captured for:

-   Incident test records
-   P1/P2 escalation
-   Business Rules
-   Script Include
-   GlideAggregate testing
-   Flow Designer
-   SLA definitions and Task SLA records
-   Problem records and related incidents
-   Major Incident UI Action
-   Work-note validation
-   Reports
-   Dashboard
-   Positive and negative tests

## 16. Design Decisions

-   Standard ServiceNow Incident and Problem tables were used.
-   Correlation logic is separated into a Script Include.
-   GlideAggregate performs the matching count.
-   Active Problem reuse prevents duplicate Problems.
-   Correlation is restricted to matching CI, Category, and time window.
-   Work notes provide traceability for automated/manual escalation.
-   Short SLA durations were used only for efficient PDI testing.

## 17. Limitations

-   Developed and tested in a ServiceNow PDI.
-   SLA test durations are not production SLA commitments.
-   No enterprise production deployment is claimed.
-   No AI component is implemented.
-   No percentage performance improvement is claimed because no
    controlled benchmark was performed.
-   No Slack, Teams, REST, OAuth, Node.js, Python, WebSockets, or
    external monitoring integration is claimed as part of the core
    project.
-   Dashboard/report values represent the PDI environment.

## 18. Final Outcome

The completed implementation demonstrates:

``` text
P1/P2 Incident
      ↓
Escalation
      ↓
Notification
      ↓
SLA Tracking
      ↓
Recurring Incident Detection
      ↓
Problem Creation / Reuse
      ↓
Incident Association
      ↓
Reports + Dashboard
```

### Verified Technology Stack

ServiceNow PDI, JavaScript, Business Rules, Script Includes,
GlideRecord, GlideAggregate, Flow Designer, SLA/Task SLA, Notifications,
UI Actions, Reports, Dashboards, Incident Management, and Problem
Management.

## 19. Completion Status

**Core ServiceNow implementation: COMPLETE and TESTED.**

Remaining project packaging tasks:

-   GitHub README and repository upload
-   Final resume update using only verified features
-   Interview explanation practice
