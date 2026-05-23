# AXA Global Healthcare — Outpatient Claim Submission

Automation workspace for submitting outpatient claims on the [AXA Global Healthcare Customer Online portal](https://customer.axaglobalhealthcare.com) using Claude + Playwright MCP.

---

## Folder layout

```
AXA/
├── README.md                               ← this file
├── invoices/
│   ├── receipts_inbox/                     ← drop receipts here before submitting
│   └── submitted_claims/                   ← receipts moved here after submission
├── reference/
│   ├── family_profiles.json                ← dependant config (placeholders)
│   ├── secrets.json                        ← real credentials (gitignored, local only)
│   └── secrets.json.example                ← template for secrets.json
├── skills/
│   └── axa-claim/
│       └── SKILL.md                        ← Claude automation instructions
└── claims_data/
    └── AXA_Claims_Log.json                 ← canonical log (gitignored)
```

---

## Setup

1. Copy `reference/secrets.json.example` → `reference/secrets.json` and fill in your values
2. Install Playwright MCP: `npx @playwright/mcp@latest`
3. Open this folder in Claude Code

---

## How it works

1. Drop a receipt image into `invoices/receipts_inbox/`
2. Tell Claude: "Submit this AXA claim" (or just drop it — the skill auto-triggers)
3. Claude reads `reference/secrets.json`, fills the portal form via Playwright, and logs the result

---

## Portal submission steps (manual reference)

### Step 1 — Log in
1. Go to https://customer.axaglobalhealthcare.com
2. Enter email and password
3. If "An active session already exists" appears → scroll down and click **Continue**

### Step 2 — Navigate to claim submission
1. From the dashboard, click **MAKE A CLAIM**
2. Scroll to **Out-patient treatment** → click **Submit invoice**

### Step 3 — Acknowledge the notice
Tick **"I acknowledge"** in the pop-up, then close it.

### Step 4 — Fill in the form

| Field | Value |
|---|---|
| Who is this invoice for? | Select the family member |
| Is the patient over 16? | No (for children) |
| Do you have a claim number? | No |
| Could this be caused by an accident? | No |
| Describe the symptoms | Diagnosis verbatim from receipt |
| When were you first aware of symptoms? | Visit date |
| What treatments are needed? | Consultation and medication |
| When did the treatment take place? | Visit date |
| Where is the treatment taking place? | Hong Kong |
| Have you already paid? | Yes |
| How much are you claiming? | Total HKD amount |
| Currency | Hong Kong Dollar |
| Is this your preferred payment method? | Yes |
| Upload supporting documents | Receipt photo |
| Are documents in English? | Yes |

Click **Save & review form**.

### Step 5 — Review & submit
1. Check summary against receipt
2. Tick both confirmation checkboxes
3. Click **Submit form**
4. Note the reference number

Repeat for each family member separately — never combine into one claim.

---

## Security

`secrets.json`, `claims_data/AXA_Claims_Log.json`, and all receipt images are gitignored and never leave your local machine.
