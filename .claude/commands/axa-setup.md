---
name: axa-setup
description: Run this when setting up the AXA claims workspace for the first time. Guides through creating secrets.json and verifying all dependencies are in place.
---

# axa-setup — First-time setup

## What you need

### 1. Credentials — create `reference/secrets.json`

Copy `reference/secrets.json.example` to `reference/secrets.json` and fill in:

| Field | Where to find it |
|---|---|
| `portal_email` | Email you use to log in to https://customer.axaglobalhealthcare.com |
| `portal_password` | Your AXA portal password |
| `policy_number` | Shown on your AXA policy documents |
| `reimbursement_bank` | Bank where AXA sends reimbursements |
| `reimbursement_account` | Account number for reimbursements |
| `policy_holder.name` | Policyholder full name as shown on AXA portal |
| `policy_holder.member_id` | 8-digit member ID from the AXA portal URL when viewing your claims |
| `dependants[].full_name` | Each dependant's full name as shown on AXA portal |
| `dependants[].member_id` | Each dependant's member ID from the AXA portal |
| `dependants[].match_keywords` | Name variations used to match receipts to the right person |

To find member IDs: log in to the portal → go to Claims → switch between family members — the ID appears in the URL (`/ClaimsInformation/XXXXXXXX`).

### 2. Playwright MCP

Required for browser automation (claim submission + CSV download).

```
npx @playwright/mcp@latest
```

Confirm it's active: the Playwright tools (`browser_navigate`, `browser_run_code_unsafe`, etc.) should appear in your Claude tool list.

### 3. Python (for CSV processing)

Required for claims tracking. Python 3.x with no extra packages needed — only the standard library (`csv`, `json`).

Check: `python --version`

### 4. Folder structure check

Verify these exist locally:
- `invoices/receipts_inbox/` — drop zone for new receipts
- `invoices/submitted_claims/` — archive of filed receipts
- `claims_data/` — CSV downloads and claims log

If missing, create them:
```
mkdir invoices\receipts_inbox invoices\submitted_claims claims_data
```

## Done

Once `secrets.json` is filled and Playwright MCP is active, you're ready to use `/axa-claim` and `/axa-track`.
