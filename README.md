# Assignment 1: Securing and Monitoring an Authenticated Flask App

[Video Demo](https://youtu.be/khCvhA9taXY)

## Overview

This assignment combines:

1. Auth0 SSO integration from Lab 1.
2. Azure App Service and Azure Monitor from Lab 2.

Goals:

1. Deploy an Auth0-authenticated Flask app to Azure App Service.
2. Add structured application logging for security-relevant user activity.
3. Use Azure Monitor and KQL to detect suspicious behavior.
4. Configure an Azure alert with email notifications.

## Scenario

This app is part of a SaaS platform managed by a DevSecOps team. The app uses Auth0 for authentication and is hosted on Azure.

Security monitoring objectives:

1. Monitor authenticated user activity.
2. Detect excessive access to sensitive routes such as /protected.
3. Alert operators when suspicious access patterns are detected.

## Repository Structure

1. app.py
2. auth.py
3. requirements.txt
4. README.md
5. PART2_MONITORING_DETECTION.md
6. templates/
7. static/

## Part 1: App Enhancements and Deployment

### 1) App Enhancements

The Flask app keeps Auth0 authentication and adds structured security logging.

Logged events:

1. login_success: includes user_id, email, and timestamp.
2. login_failure: includes timestamp and error context.
3. protected_access: includes user_id, email, route, and timestamp.
4. unauthorized_access: includes route, reason, and timestamp.

Implementation notes:

1. Logs are emitted with app.logger.info and app.logger.warning.
2. Logs are structured JSON prefixed with SECLOG to simplify KQL filtering.

### 2) Deploy to Azure App Service

Deploy the app to Azure App Service and enable diagnostics to Log Analytics.

Minimum required Azure configuration:

1. App Service is running the latest deployed app code.
2. Diagnostic setting is configured to send AppServiceConsoleLogs to the target Log Analytics workspace.
3. App is restarted after configuration updates.

Auth0 cloud settings checklist:

1. Allowed Callback URLs includes:
	https://<your-app-name>.azurewebsites.net/callback
2. Allowed Logout URLs includes:
	https://<your-app-name>.azurewebsites.net/

## Part 2: Monitoring and Detection

Detailed step-by-step instructions are in PART2_MONITORING_DETECTION.md.

### KQL Detection Query

This query detects users accessing /protected more than 10 times in 15 minutes.

```kusto
AppServiceConsoleLogs
| where TimeGenerated >= ago(15m)
| where ResultDescription has "SECLOG"
| extend jsonText = extract(@"SECLOG\s+(\{.*\})", 1, ResultDescription)
| where isnotempty(jsonText)
| extend log = parse_json(jsonText)
| where tostring(log.event) == "protected_access"
| where tostring(log.route) == "/protected"
| summarize access_count = count(), timestamp = max(TimeGenerated) by user_id = tostring(log.user_id)
| where access_count > 10
| project user_id, timestamp, access_count
| order by access_count desc
```

### Alert Rule Logic

Use a Scheduled Query alert with the following values:

1. Query type: Log.
2. Measurement: Table rows.
3. Operator: Greater than.
4. Threshold: 0.
5. Evaluation frequency: 5 minutes.
6. Lookback window: 15 minutes.
7. Severity: 3 (Low).
8. Action Group: Email notification enabled.

## Part 3: GitHub Repo and Demo

### Required for Submission

1. Source code in this repository.
2. requirements.txt.
3. .env.example without secrets.
4. README.md with setup, logging, KQL, and alert logic.
5. test-app.http for valid and invalid request simulation.

### Suggested Demo Outline (10 minutes max)

1. Show app running on Azure with Auth0 login.
2. Show protected route access and unauthorized attempt behavior.
3. Show logs arriving in AppServiceConsoleLogs.
4. Run KQL query and explain results.
5. Show alert rule and action group email configuration.
6. Reflection on what worked and what to improve.

## Local Setup

### 1) Create and activate virtual environment

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### 2) Install dependencies

```powershell
pip install -r requirements.txt
```

### 3) Configure environment variables

Create .env locally with values from your Auth0 tenant:

```ini
AUTH0_DOMAIN=YOUR_AUTH0_DOMAIN
AUTH0_CLIENT_ID=YOUR_CLIENT_ID
AUTH0_CLIENT_SECRET=YOUR_CLIENT_SECRET
AUTH0_SECRET=YOUR_GENERATED_SECRET
AUTH0_REDIRECT_URI=http://localhost:5000/callback
```

For Azure deployment, use App Service application settings for production URLs and secrets.

### 4) Run locally

```powershell
python app.py
```
