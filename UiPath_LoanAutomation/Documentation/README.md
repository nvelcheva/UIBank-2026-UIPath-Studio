# Loan Automation System - Complete Documentation

## 📋 Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [UiPath Project Structure](#uipath-project-structure)
4. [Workflow Descriptions](#workflow-descriptions)
5. [Power Automate Integration](#power-automate-integration)
6. [Setup Instructions](#setup-instructions)
7. [Configuration Guide](#configuration-guide)
8. [Hidden Traps & Solutions](#hidden-traps--solutions)
9. [Testing Guide](#testing-guide)
10. [Troubleshooting](#troubleshooting)

---

## 🎯 Project Overview

### Business Objective
Automate the processing of 100 loan application PDFs through the UiBank web platform, including:
- PDF data extraction
- Currency conversion to USD
- Loan submission to UiBank
- Approval/rejection handling
- Account creation for new applicants
- Excel reporting with interest rate analysis

### Key Features
- ✅ Automated PDF parsing with regex patterns
- ✅ Real-time currency conversion via API
- ✅ Web automation for UiBank form submission
- ✅ Intelligent result handling (approved/rejected)
- ✅ New account creation detection
- ✅ Comprehensive Excel reporting
- ✅ Interest rate comparison analysis
- ⛔ NO client emails sent (per requirement)

---

## 🏗️ Architecture

### System Flow
```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   PDF Files     │────▶│   UiPath RPA    │────▶│   UiBank Web    │
│   (100 docs)    │     │   Automation    │     │   Platform      │
└─────────────────┘     └────────┬────────┘     └────────┬────────┘
                                 │                       │
                                 ▼                       ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Exchange Rate  │◀────│  Currency API   │     │  Approval /     │
│  Conversion     │     │  (REST)         │     │  Rejection      │
└─────────────────┘     └─────────────────┘     └────────┬────────┘
                                                         │
                                 ┌───────────────────────┘
                                 ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  New Account    │     │  Excel Report   │     │  Interest Rate  │
│  Creation       │────▶│  Generation     │────▶│  Analysis       │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Technology Stack
| Component | Technology | Purpose |
|-----------|------------|---------|
| RPA Platform | UiPath Studio 2023.10 | Core automation |
| Web Browser | Chrome | UiBank interaction |
| PDF Processing | UiPath.PDF.Activities | Data extraction |
| Currency API | exchangerate-api.com | Exchange rates |
| Country API | restcountries.com | Country info |
| Reporting | Excel | Results output |
| Orchestration | Power Automate (optional) | Cloud scheduling |

---

## 📁 UiPath Project Structure

```
LoanAutomation_UiBank/
├── project.json                    # Project configuration
├── Main.xaml                       # Main orchestrator workflow
├── Data/
│   ├── Config.json                 # Configuration settings
│   └── SamplePDF_Template.txt      # PDF format reference
├── Workflows/
│   ├── InitializeConfig.xaml       # Load configuration
│   ├── GetPDFFiles.xaml            # Scan PDF folder
│   ├── ExtractPDFData.xaml         # Parse PDF content
│   ├── ConvertCurrency.xaml        # USD conversion
│   ├── CreateUiBankAccount.xaml    # Bot account creation
│   ├── SubmitLoanApplication.xaml  # Form submission
│   ├── ProcessResult.xaml          # Handle outcomes
│   ├── CreateApplicantAccount.xaml # New applicant accounts
│   ├── GenerateExcelReport.xaml    # Excel output
│   └── InterestRateAnalysis.xaml   # Rate comparison
├── Documentation/
│   └── (this file)
├── Exceptions/                     # Custom exception handling
└── Framework/                      # REFramework templates
```

---

## 🔧 Workflow Descriptions

### 1. Main.xaml (Orchestrator)
**Purpose:** Controls the entire automation flow
**Input Arguments:**
- `in_ConfigPath` (String): Path to Config.json
- `in_PDFFolderPath` (String): Location of loan PDFs
- `in_OutputExcelPath` (String): Output Excel file path

**Flow:**
1. Initialize configuration
2. Get all PDF files
3. Create bot UiBank account
4. Loop through each PDF:
   - Extract data
   - Convert currency
   - Submit to UiBank
   - Process result
5. Generate Excel report
6. Generate analysis report

### 2. ExtractPDFData.xaml
**Purpose:** Extract loan information from PDF
**Regex Patterns Used:**
```regex
Email:       Email[:\s]*([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})
Amount:      Request\s*Amount[:\s]*([0-9,.]+)\s*([A-Z]{3})
Income:      Yearly\s*Income[:\s]*([0-9,.]+)\s*([A-Z]{3})?
Age:         Age[:\s]*([0-9]+)
Interest:    (?:Estimated\s*)?Interest\s*Rate[:\s]*([0-9.]+)\s*%?
Country:     Country[:\s]*([A-Za-z\s]+)
```

### 3. ConvertCurrency.xaml
**Purpose:** Convert all monetary values to USD
**API Used:** `https://api.exchangerate-api.com/v4/latest/USD`
**Fallback Rates:** Configured in Config.json for offline operation

### 4. SubmitLoanApplication.xaml
**Purpose:** Fill and submit UiBank loan form
**Selectors Used:**
```xml
<webctrl tag='INPUT' id='email' />
<webctrl tag='INPUT' id='loanAmount' />
<webctrl tag='INPUT' id='yearlyIncome' />
<webctrl tag='INPUT' id='age' />
<webctrl tag='BUTTON' type='submit' />
```

### 5. ProcessResult.xaml
**Purpose:** Handle approval/rejection outcomes
**Decision Logic:**
- If APPROVED:
  - Extract Loan ID and APR
  - Check if "New Application"
  - Compare rates (PDF vs UiBank)
  - Create account if needed
- If REJECTED:
  - Log rejection
  - Move to next applicant

### 6. InterestRateAnalysis.xaml
**Purpose:** Business intelligence report
**Metrics Calculated:**
- Average UiBank APR
- Average PDF Estimated Rate
- Count: APR better / PDF better / Equal
- Recommendation based on lower rates

---

## 🔗 Power Automate Integration

### Why Use Both?

| Capability | UiPath | Power Automate |
|------------|--------|----------------|
| Web Automation | ✅ Excellent | ❌ Limited |
| PDF Extraction | ✅ Native | ⚠️ Needs AI Builder |
| Cloud Scheduling | ⚠️ Needs Orchestrator | ✅ Built-in |
| SharePoint Integration | ⚠️ Requires setup | ✅ Native |
| Excel Online | ⚠️ Desktop only | ✅ Native |
| Cost | 💰 License required | 💰 M365 included |

### Hybrid Architecture
```
Power Automate (Cloud)          UiPath (Desktop)
┌───────────────────┐          ┌───────────────────┐
│ • Schedule trigger│          │ • Web automation  │
│ • SharePoint read │──────────▶ • Form filling   │
│ • AI Builder OCR  │          │ • Account creation│
│ • Teams notify    │◀──────────│ • Result handling│
└───────────────────┘          └───────────────────┘
```

### Power Automate Flow Setup
1. Import `LoanProcessing_Flow.json`
2. Configure connections:
   - SharePoint Online
   - AI Builder
   - UiPath Orchestrator
   - Excel Online
3. Update parameters:
   - UiPathReleaseKey
   - UiPathRobotId
   - SharePoint site URL

---

## ⚙️ Setup Instructions

### Prerequisites
- UiPath Studio 2023.10+
- Chrome browser
- Microsoft 365 (for Power Automate)
- Network access to UiBank

### Step 1: Install UiPath Dependencies
```
UiPath.Excel.Activities [2.23.0]
UiPath.PDF.Activities [3.18.0]
UiPath.System.Activities [23.10.0]
UiPath.UIAutomation.Activities [23.10.0]
UiPath.WebAPI.Activities [1.18.0]
```

### Step 2: Configure Paths
Edit `Data/Config.json`:
```json
{
  "Assets": {
    "PDFFolderPath": "YOUR_PDF_FOLDER_PATH",
    "OutputExcelPath": "YOUR_OUTPUT_PATH\\LoanResults.xlsx"
  }
}
```

### Step 3: Prepare PDF Folder
- Place all 100 PDF files in the configured folder
- Ensure PDFs follow the labeled form structure

### Step 4: Run the Automation
1. Open `Main.xaml` in UiPath Studio
2. Click "Run" or press F5
3. Monitor the Output panel for progress

---

## ⚠️ Hidden Traps & Solutions

### Trap 1: PDF Format Variations
**Problem:** PDFs may have inconsistent formatting
**Solution:** 
- Use flexible regex patterns
- Implement fallback extraction methods
- Log extraction failures for manual review

### Trap 2: Currency Parsing Edge Cases
**Problem:** Amount formats vary (1,000.00 vs 1.000,00)
**Solution:**
```vb
' Remove all non-numeric except decimal
CleanAmount = Regex.Replace(RawAmount, "[^0-9.]", "")
```

### Trap 3: Website Timeout/Loading
**Problem:** UiBank may load slowly
**Solution:**
- Set appropriate timeouts (30s page load)
- Use ElementExists before interactions
- Implement retry logic

### Trap 4: Duplicate Applications
**Problem:** Same email submitted multiple times
**Solution:**
- Track processed emails in a HashSet
- Check before submission
- Log duplicates to separate sheet

### Trap 5: Session Management
**Problem:** Browser session expires
**Solution:**
- Create fresh session per batch
- Handle login if required
- Close browser after each submission

### Trap 6: Exchange Rate API Downtime
**Problem:** API unavailable
**Solution:**
- Implement fallback rates in Config.json
- Cache last known rates
- Log when fallback used

### Trap 7: New Application Detection
**Problem:** Loan ID format may vary
**Solution:**
- Use Contains("new application") check
- Handle both scenarios gracefully

### Trap 8: Email Temptation
**Problem:** Natural inclination to send confirmations
**Solution:**
- **EXPLICIT RULE: NO EMAILS SENT**
- Removed all email activities
- Added comments as reminders

---

## 🧪 Testing Guide

### Test Case 1: Single PDF Processing
1. Place one test PDF in input folder
2. Run automation
3. Verify:
   - Data extracted correctly
   - Currency converted
   - Form submitted
   - Result recorded

### Test Case 2: Multiple Currencies
Test PDFs with: EUR, GBP, BGN, JPY, USD
Verify correct conversion for each

### Test Case 3: Approval Flow
Use data that triggers approval
Verify:
- Loan ID captured
- APR captured
- Account created if "New Application"

### Test Case 4: Rejection Flow
Use data that triggers rejection
Verify:
- Logged as rejected
- No account created
- Continues to next

### Test Case 5: API Failure
1. Disconnect network
2. Run automation
3. Verify fallback rates used

### Test Case 6: Full Batch (100 PDFs)
Run complete batch and verify:
- All 100 processed
- Excel has all results
- Analysis report generated

---

## 🔍 Troubleshooting

### Error: "Element not found"
**Cause:** Selector changed on UiBank
**Fix:** Update selector in SubmitLoanApplication.xaml

### Error: "PDF text empty"
**Cause:** Scanned PDF without OCR
**Fix:** Use OCR activity or pre-process PDFs

### Error: "Currency not supported"
**Cause:** Unknown currency code
**Fix:** Add to fallback rates in Config.json

### Error: "API timeout"
**Cause:** Network issues
**Fix:** Increase timeout or use fallback

### Error: "Browser not starting"
**Cause:** Chrome not installed or updated
**Fix:** Install Chrome or update BrowserType in config

---

## 📊 Output Files

### LoanResults.xlsx Structure

**Sheet 1: All Results**
| Column | Description |
|--------|-------------|
| Email | Applicant email |
| RequestAmountUSD | Converted loan amount |
| YearlyIncomeUSD | Converted income |
| Age | Applicant age |
| Country | Origin country |
| Status | APPROVED/REJECTED |
| LoanID | UiBank loan ID |
| APRRate | UiBank APR |
| EstimatedInterestRate | PDF rate |
| RateComparison | Which is better |
| ProcessedAt | Timestamp |

**Sheet 2: Approved**
Only approved applications

**Sheet 3: Rejected**
Only rejected applications

**Sheet 4: Interest Rate Analysis**
| Metric | Value |
|--------|-------|
| Total Approved | Count |
| Average UiBank APR | X% |
| Average PDF Estimated | Y% |
| APR Better Count | N |
| PDF Better Count | M |
| RECOMMENDATION | Use lower rate method |

---

## 📞 Support

For issues or questions:
1. Check this documentation
2. Review UiPath logs
3. Contact development team

---

*Document Version: 1.0*
*Last Updated: January 2025*
*Author: Loan Automation Team*
