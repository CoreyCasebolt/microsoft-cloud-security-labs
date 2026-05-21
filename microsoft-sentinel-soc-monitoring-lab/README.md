# Microsoft Sentinel SOC Monitoring Lab

## Summary

Built a Microsoft Sentinel SOC monitoring lab to practice cloud log ingestion, KQL-based log validation, analytics rule creation, alert and incident review, rule tuning, and workbook dashboarding.

This lab used Microsoft Sentinel with Microsoft Entra ID and Azure Activity logs to simulate a basic SOC workflow. The project demonstrated how cloud identity and subscription activity can be collected, queried, detected, and visualized inside Microsoft Sentinel.

Sensitive user, IP address, subscription, tenant, and object ID values were redacted or partially redacted in screenshots before publishing.

## Objective

The goal of this project was to:

- Create a Microsoft Sentinel SIEM lab using a Log Analytics workspace
- Ingest Azure Activity and Microsoft Entra ID logs into Sentinel
- Validate log ingestion using beginner-level KQL queries
- Generate safe test identity activity for investigation
- Create a scheduled analytics rule for repeated failed sign-ins
- Validate alert and incident creation from the analytics rule
- Tune the analytics rule after observing duplicate alerts and incidents
- Build a Sentinel workbook dashboard to visualize identity and Azure activity

## Tools & Technologies

- Microsoft Azure
- Microsoft Sentinel
- Microsoft Defender portal
- Microsoft Entra ID
- Log Analytics workspace
- Azure Activity logs
- Entra sign-in logs
- Entra audit logs
- Kusto Query Language, or KQL
- Sentinel analytics rules
- Sentinel workbooks

## Environment

| Component | Details |
|---|---|
| Cloud Platform | Microsoft Azure |
| SIEM | Microsoft Sentinel |
| Workspace | `law-sentinel-soc-lab` |
| Resource Group | `cloud-security-lab` |
| Region | West US |
| Identity Platform | Microsoft Entra ID |
| Log Sources | Azure Activity, Entra Sign-in Logs, Entra Audit Logs |
| Test Activity | Failed sign-ins, successful sign-ins, test user creation/update |
| Dashboard | Sentinel workbook built from Sentinel log data |

## What I Configured

- Created a dedicated Log Analytics workspace named `law-sentinel-soc-lab`
- Enabled Microsoft Sentinel on the Log Analytics workspace
- Installed and configured the Azure Activity connector
- Verified Azure Activity ingestion using the `AzureActivity` table
- Installed and configured the Microsoft Entra ID connector
- Enabled Microsoft Entra sign-in logs and audit logs
- Verified identity log ingestion using the `SigninLogs` and `AuditLogs` tables
- Generated safe test identity activity for validation
- Created beginner-level KQL queries for common SOC review scenarios
- Created a scheduled analytics rule for repeated failed sign-ins from the same source IP
- Validated alert and incident generation from the analytics rule
- Tuned the analytics rule schedule after duplicate alerts were observed
- Built a Sentinel workbook dashboard to visualize failed sign-ins, source IPs, audit events, and Azure activity

## KQL Queries Used

The following KQL queries were saved in the `kql/` folder.

### Recent Azure Activity Events

```kql
AzureActivity
| project TimeGenerated, OperationNameValue, ActivityStatusValue, ResourceGroup, ResourceProviderValue
| sort by TimeGenerated desc
| take 20
```

### Recent Sign-in Events

```kql
SigninLogs
| sort by TimeGenerated desc
| take 10
```

### Failed Sign-ins

```kql
SigninLogs
| where ResultType != 0
| project TimeGenerated, UserPrincipalName, IPAddress, AppDisplayName, ResultType, ResultDescription, Location
| sort by TimeGenerated desc
```

### Successful Sign-ins

```kql
SigninLogs
| where ResultType == 0
| project TimeGenerated, UserPrincipalName, IPAddress, AppDisplayName, Location
| sort by TimeGenerated desc
```

### Top Failed Sign-in Source IPs

```kql
SigninLogs
| where ResultType != 0
| summarize FailedAttempts=count() by IPAddress
| sort by FailedAttempts desc
```

### New Users Created

```kql
AuditLogs
| where OperationName has "Add user"
   or OperationName has "Create user"
| project TimeGenerated, OperationName, Result, InitiatedBy, TargetResources
| sort by TimeGenerated desc
```

### Analytics Rule: Multiple Failed Sign-ins From Same IP

```kql
SigninLogs
| where ResultType != 0
| summarize FailedAttempts=count() by UserPrincipalName, IPAddress
| where FailedAttempts >= 3
```

## Screenshots

### Log Analytics Workspace Created

![Log Analytics Workspace Created](screenshots/sentinel-workspace-created.png)

Created the `law-sentinel-soc-lab` Log Analytics workspace used as the data store for Microsoft Sentinel.

### Microsoft Sentinel Enabled

![Microsoft Sentinel Enabled](screenshots/sentinel-enabled.png)

Enabled Microsoft Sentinel on the Log Analytics workspace to create the SIEM environment.

### Data Connectors Overview

![Data Connectors Overview](screenshots/data-connectors-overview.png)

Opened the Microsoft Sentinel data connectors page to begin configuring log sources.

### Azure Activity Diagnostic Setting

![Azure Activity Diagnostic Setting](screenshots/azure-activity-diagnostic-setting.png)

Configured the Azure subscription diagnostic setting to send Azure Activity logs to the Sentinel workspace.

### Azure Activity Connector Configured

![Azure Activity Connector Configured](screenshots/azure-activity-connector-configured.png)

Confirmed the Azure Activity connector was connected and receiving logs.

### Azure Activity Query Results

![Azure Activity Query Results](screenshots/azure-activity-query-results.png)

Verified Azure Activity ingestion by querying the `AzureActivity` table in Sentinel Logs.

### Microsoft Entra ID Content Hub

![Microsoft Entra ID Content Hub](screenshots/entra-id-content-hub.png)

Installed the Microsoft Entra ID solution from the Microsoft Sentinel Content hub.

### Microsoft Entra ID Connector Configured

![Microsoft Entra ID Connector Configured](screenshots/entra-id-connector-configured.png)

Configured the Microsoft Entra ID connector to ingest sign-in logs and audit logs.

### Sign-in Logs Query Results

![Sign-in Logs Query Results](screenshots/signinlogs-query-results.png)

Verified Microsoft Entra sign-in log ingestion by querying the `SigninLogs` table.

### Audit Logs Query Results

![Audit Logs Query Results](screenshots/auditlogs-query-results.png)

Verified Microsoft Entra audit log ingestion by querying the `AuditLogs` table.

### Successful Sign-ins Query

![Successful Sign-ins Query](screenshots/successful-signins-query.png)

Used KQL to review successful Microsoft Entra sign-in events.

### Failed Sign-ins Query

![Failed Sign-ins Query](screenshots/failed-signins-query.png)

Used KQL to review failed or interrupted sign-in events, including invalid credential attempts and authentication challenges.

### Top Failed Sign-in Source IPs

![Top Failed Sign-in Source IPs](screenshots/top-failed-signin-ips-query.png)

Grouped failed sign-ins by source IP address to identify repeated failed authentication attempts.

### New Users Created Query

![New Users Created Query](screenshots/new-users-created-query.png)

Queried Entra audit logs to identify user creation activity.

### Recent Azure Activity Events Query

![Recent Azure Activity Events Query](screenshots/azure-activity-recent-events-query.png)

Queried recent Azure subscription-level activity collected by Sentinel.

### Analytics Rule Details

![Analytics Rule Details](screenshots/analytics-rule-multiple-failed-signins.png)

Created a scheduled analytics rule named `Multiple Failed Sign-ins From Same IP` with Medium severity and the Credential Access MITRE ATT&CK tactic.

### Analytics Rule Query Logic

![Analytics Rule Query Logic](screenshots/analytics-rule-query-logic.png)

Configured the analytics rule logic to detect three or more failed sign-ins from the same user and source IP address.

### Analytics Rule Created

![Analytics Rule Created](screenshots/analytics-rule-created.png)

Confirmed the scheduled analytics rule was created and enabled.

### Alert Created From Failed Sign-ins

![Alert Created From Failed Sign-ins](screenshots/alert-created-from-failed-signins.png)

Generated failed sign-in activity and confirmed that the Sentinel analytics rule created an alert.

### Incident Created From Failed Sign-ins

![Incident Created From Failed Sign-ins](screenshots/incident-created-from-failed-signins.png)

Confirmed that the analytics rule generated an incident for investigation.

### Incident Details

![Incident Details](screenshots/incident-details-failed-signins.png)

Reviewed the incident details associated with the failed sign-in detection.

### Multiple Alerts From Rule Testing

![Multiple Alerts From Rule Testing](screenshots/multiple-alerts-from-rule-testing.png)

Observed duplicate alerts and incidents during rule testing due to the rule frequency and lookup period overlap.

### Tuned Analytics Rule Schedule

![Tuned Analytics Rule Schedule](screenshots/analytics-rule-tuned-schedule.png)

Adjusted the analytics rule schedule after observing duplicate alerts and incidents during testing.

### Workbook Created

![Workbook Created](screenshots/sentinel-workbook-created.png)

Created a new Sentinel workbook in the Microsoft Defender portal.

### Workbook: Failed Sign-ins Over Time

![Workbook Failed Sign-ins Over Time](screenshots/workbook-failed-signins-over-time.png)

Added a time chart to visualize failed Microsoft Entra sign-ins over time.

### Workbook: Top Failed Sign-in Source IPs

![Workbook Top Failed Sign-in Source IPs](screenshots/workbook-top-failed-signin-ips.png)

Added a workbook panel showing the top source IP addresses associated with failed sign-ins.

### Workbook: Recent Failed Sign-ins

![Workbook Recent Failed Sign-ins](screenshots/workbook-recent-failed-signins.png)

Added a workbook table showing recent failed or interrupted sign-in activity.

### Workbook: Recent Entra Audit Events

![Workbook Recent Entra Audit Events](screenshots/workbook-recent-entra-audit-events.png)

Added a workbook panel showing recent Microsoft Entra audit events.

### Workbook: Recent Azure Activity Events

![Workbook Recent Azure Activity Events](screenshots/workbook-recent-azure-activity-events.png)

Added a workbook panel showing recent Azure Activity events.

### Sentinel SOC Monitoring Dashboard

![Sentinel SOC Monitoring Dashboard](screenshots/sentinel-workbook-dashboard.png)

Built a Sentinel workbook dashboard to visualize failed sign-in trends, top failed sign-in source IPs, and recent identity activity.

## Analytics Rule Testing Observation

During testing, the scheduled analytics rule generated multiple alerts and incidents from the same failed sign-in activity. This occurred because the rule was originally configured to run every 5 minutes while looking back over the previous 15 minutes of data.

Because the same failed sign-in events remained inside the 15-minute lookup window across multiple rule runs, Sentinel matched the same activity more than once and generated duplicate alerts/incidents.

This was a useful learning point because it demonstrated how rule frequency, lookup period, grouping, and suppression settings affect alert volume.

After observing the duplicate alerts/incidents, I tuned the rule from a 5-minute run frequency with a 15-minute lookup period to a 10-minute run frequency with a 10-minute lookup period. This reduced overlap between scheduled query runs and helped demonstrate how analytics rules can be adjusted after testing.

In a production environment, this type of rule could be further tuned using:

- Matching query frequency and lookup periods
- Alert suppression
- Incident grouping
- More specific KQL logic
- Entity mapping and investigation fields
- Threshold adjustments based on expected user behavior

## Results

This project successfully demonstrated:

- Microsoft Sentinel deployment on a Log Analytics workspace
- Azure Activity log ingestion into Sentinel
- Microsoft Entra ID sign-in and audit log ingestion into Sentinel
- KQL-based validation of collected cloud and identity logs
- Basic investigation of successful sign-ins, failed sign-ins, audit events, and Azure activity
- Creation of a scheduled analytics rule from a KQL query
- Alert and incident generation from repeated failed sign-in activity
- Rule tuning based on duplicate alert behavior
- Workbook dashboard creation for SOC-style monitoring

## Key Takeaways

- Microsoft Sentinel uses Log Analytics workspaces to store and query collected security data
- Data connectors are required before Sentinel can investigate Azure and Entra activity
- KQL can be used to validate ingestion and investigate common identity events
- Failed sign-in patterns can be turned into scheduled analytics rules
- Analytics rule frequency and lookup period settings directly affect duplicate alert generation
- Sentinel alerts and incidents help turn raw log activity into an investigation workflow
- Workbooks provide a dashboard-style view of security activity for analysts and administrators
- Screenshots should be reviewed and redacted before publishing to avoid exposing usernames, IP addresses, subscription IDs, tenant IDs, or object IDs

## Real-World Relevance

This project relates to real security and IT operations by demonstrating:

- How a SOC team can collect identity and Azure activity logs into a SIEM
- How analysts can use KQL to validate log ingestion and investigate sign-in activity
- How failed authentication activity can be converted into a detection rule
- How alerts and incidents support investigation workflows
- How duplicate alerts can occur if analytics rules are not tuned properly
- How dashboards can help analysts quickly review identity and cloud activity
- How Microsoft Sentinel and Microsoft Defender are used together for modern cloud security monitoring

This lab is relevant for SOC analyst, cloud security intern, Microsoft 365 security, and managed services roles because it demonstrates a full monitoring workflow:

```text
Log ingestion → KQL validation → Analytics rule → Alert/incident → Rule tuning → Workbook dashboard
```
