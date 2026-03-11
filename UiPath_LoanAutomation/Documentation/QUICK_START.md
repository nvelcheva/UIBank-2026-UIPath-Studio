# Quick Start Guide - Loan Automation

## 🚀 5-Minute Setup

### Step 1: Download the Project
Copy the entire `UiPath_LoanAutomation` folder to your local machine.

### Step 2: Open in UiPath Studio
1. Launch UiPath Studio 2023.10+
2. Click "Open Project"
3. Navigate to the folder and select `project.json`

### Step 3: Install Dependencies
When prompted, click "Manage Packages" and install:
- UiPath.PDF.Activities
- UiPath.Excel.Activities
- UiPath.WebAPI.Activities

### Step 4: Configure Paths
Edit `Data/Config.json`:
```json
{
  "Assets": {
    "PDFFolderPath": "C:\\YourPath\\PDFs",
    "OutputExcelPath": "C:\\YourPath\\Output\\Results.xlsx"
  }
}
```

### Step 5: Place Your PDFs
Copy your 100 loan PDF files to the configured `PDFFolderPath`

### Step 6: Run!
1. Open `Main.xaml`
2. Press F5 or click "Run"
3. Watch the magic happen! ✨

---

## 📋 PDF Format Expected

Your PDFs should contain these labeled fields:
```
Email: applicant@email.com
Request Amount: 50000 EUR
Loan Term: 36 months
Yearly Income: 75000 EUR
Age: 35
Estimated Interest Rate: 5.5%
Country: Germany
```

---

## ⚠️ Important Notes

1. **NO EMAILS** - The automation will NOT send any emails to clients
2. **Chrome Required** - Make sure Chrome browser is installed
3. **Internet Needed** - For currency conversion API
4. **UiBank Access** - Automation will create a test account automatically

---

## 🎯 Expected Output

After completion, you'll find:
- `LoanResults.xlsx` with 4 sheets:
  - All Results
  - Approved
  - Rejected
  - Interest Rate Analysis

---

## 🆘 Quick Troubleshooting

| Issue | Solution |
|-------|----------|
| "Element not found" | Check Chrome is open, increase timeout |
| "PDF extraction failed" | Verify PDF format matches template |
| "Currency error" | Check API access or use fallback rates |

For detailed help, see `Documentation/README.md`
