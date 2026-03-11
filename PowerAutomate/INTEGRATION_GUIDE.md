# Power Automate + UiPath Integration Guide

## 📌 Overview

This guide explains how to use both Power Automate AND UiPath together for the loan automation system. This hybrid approach combines the best of both platforms.

---

## 🤔 Why Use Both?

### UiPath Strengths (Desktop RPA)
- ✅ **Web Automation**: Superior browser control with reliable selectors
- ✅ **PDF Processing**: Native PDF reading without OCR
- ✅ **Complex Logic**: Better handling of conditional flows
- ✅ **Error Recovery**: Advanced exception handling

### Power Automate Strengths (Cloud)
- ✅ **Cloud Scheduling**: Run without dedicated machine
- ✅ **SharePoint Integration**: Native file access
- ✅ **Teams Notifications**: Easy alerts
- ✅ **No Infrastructure**: Runs in Microsoft cloud

### Hybrid Architecture
```
┌────────────────────────────────────────────────────────────┐
│                    POWER AUTOMATE (Cloud)                   │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐ │
│  │ Schedule │──▶│SharePoint│──▶│Trigger   │──▶│  Teams   │ │
│  │ Trigger  │   │Get Files │   │UiPath Job│   │ Notify   │ │
│  └──────────┘   └──────────┘   └────┬─────┘   └──────────┘ │
└──────────────────────────────────────│─────────────────────┘
                                       │
                                       ▼
┌────────────────────────────────────────────────────────────┐
│                    UIPATH (Desktop)                         │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐ │
│  │ Extract  │──▶│ Convert  │──▶│ Submit   │──▶│ Report   │ │
│  │ PDF Data │   │ Currency │   │ to UiBank│   │ to Excel │ │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘ │
└────────────────────────────────────────────────────────────┘
```

---

## 🔧 Setup Instructions

### Prerequisites
1. **Microsoft 365** with Power Automate license
2. **UiPath Orchestrator** account
3. **UiPath Robot** on a machine
4. **SharePoint** site for files

### Step 1: Publish UiPath to Orchestrator

1. Open the project in UiPath Studio
2. Click "Publish" in the ribbon
3. Select your Orchestrator tenant
4. Note the **Release Key** after publishing

### Step 2: Create UiPath API User

1. Go to UiPath Orchestrator
2. Settings → Users → Add User
3. Create an API user for Power Automate
4. Note the credentials

### Step 3: Import Power Automate Flow

1. Go to https://flow.microsoft.com
2. Click "Import" → "Import Package"
3. Upload `PowerAutomate/LoanProcessing_Flow.json`
4. Configure connections during import

### Step 4: Configure Connections

#### SharePoint Connection
- Site URL: Your SharePoint site
- Library: LoanPDFs

#### UiPath Connection
- Orchestrator URL: https://cloud.uipath.com/{org}/{tenant}
- API Key or OAuth credentials

#### Excel Online Connection
- Default drive or specific OneDrive

### Step 5: Update Parameters

In the flow editor, update:
```
UiPathReleaseKey: [Your release key from Step 1]
UiPathRobotId: [Your unattended robot ID]
SharePointSite: [Your SharePoint URL]
```

---

## 📊 Flow Details

### Trigger: Scheduled Recurrence
- Default: Daily at 9:00 AM
- Can change to manual, webhook, or file-based trigger

### Actions Breakdown

#### 1. Get PDF Files from SharePoint
```json
{
  "action": "GetItems",
  "site": "https://company.sharepoint.com/sites/LoanProcessing",
  "list": "LoanPDFs"
}
```

#### 2. For Each PDF Loop
- Sequential processing (1 at a time)
- Prevents overwhelming UiBank

#### 3. Trigger UiPath Job
```json
{
  "startInfo": {
    "ReleaseKey": "loan-automation-release-key",
    "Strategy": "Specific",
    "RobotIds": ["robot-id"],
    "InputArguments": {
      "in_PDFPath": "@{items('For_Each')?['FileRef']}"
    }
  }
}
```

#### 4. Wait for Job Completion
- Polls job status every 30 seconds
- Timeout: 1 hour per PDF

#### 5. Collect Results
- Appends job output to results array
- Handles both success and failure

#### 6. Create Excel Report
- Writes all results to Excel Online
- Updates existing table

#### 7. Send Teams Notification
```
"Loan Processing Complete! 
Processed: 100 applications
Results available in SharePoint."
```

---

## 🚨 Error Handling

### UiPath Job Failure
```json
{
  "condition": "@equals(body('Get_Job_Status')?['State'], 'Faulted')",
  "actions": {
    "Log_Error": {
      "type": "Compose",
      "inputs": "Job failed: @{body('Get_Job_Status')?['Info']}"
    },
    "Continue_Loop": {
      "type": "Continue"
    }
  }
}
```

### API Failures
- Built-in retry policy: 3 attempts
- Exponential backoff

### SharePoint Connection Lost
- Reconnect action
- Alert via Teams

---

## 📁 File: LoanProcessing_Flow.json

The provided JSON file contains the complete Power Automate flow definition. Import this directly into Power Automate.

### Required Connections (configure during import):
1. `sharepointonline` - SharePoint Online
2. `uipath` - UiPath Orchestrator
3. `excelonline` - Excel Online (Business)
4. `teams` - Microsoft Teams

---

## ⚖️ UiPath Only vs. Hybrid Comparison

| Aspect | UiPath Only | Hybrid (PA + UiPath) |
|--------|-------------|---------------------|
| **Scheduling** | Orchestrator required | PA built-in |
| **File Storage** | Local/Network | SharePoint (cloud) |
| **Notifications** | Custom setup | Teams native |
| **Cost** | UiPath license | UiPath + M365 |
| **Reliability** | Single point | Distributed |
| **Setup Complexity** | Simpler | More components |

### Recommendation
- **Small scale (< 50 PDFs)**: UiPath only
- **Enterprise (100+ PDFs)**: Hybrid approach
- **Cloud-first organization**: Hybrid approach
- **Budget constrained**: UiPath only with free scheduler

---

## 🔐 Security Considerations

1. **API Keys**: Store in Azure Key Vault
2. **Robot Credentials**: Use Orchestrator Assets
3. **SharePoint**: Apply appropriate permissions
4. **Data in Transit**: HTTPS only
5. **Audit Logs**: Enable in both platforms

---

## 📞 Support Resources

- **UiPath Forum**: https://forum.uipath.com
- **Power Automate Docs**: https://docs.microsoft.com/power-automate
- **UiPath Academy**: https://academy.uipath.com

---

*Document Version: 1.0*
*Last Updated: January 2025*
