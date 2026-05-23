# AXA Global Healthcare — Outpatient Claim Submission

Automation workspace for submitting outpatient claims on the [AXA Global Healthcare Customer Online portal](https://customer.axaglobalhealthcare.com) using Claude + Playwright MCP.

---

## Folder layout

```
AXA/
├── README.md
├── PROJECT.md                              ← playbook for Claude
├── invoices/
│   ├── receipts_inbox/                     ← drop receipts here before submitting
│   └── submitted_claims/                   ← receipts moved here after submission
├── reference/
│   ├── AXA_Claim_Submission_Guide.md       ← step-by-step portal walkthrough
│   ├── family_profiles.json                ← dependant config (placeholders)
│   └── secrets.json                        ← real credentials (gitignored, local only)
├── skills/
│   └── axa-claim/
│       └── SKILL.md                        ← Claude auto-trigger instructions
└── AXA_Claims_Log.json                     ← canonical log (gitignored)
```

---

## How it works

1. Drop a receipt image into `invoices/receipts_inbox/`
2. Tell Claude: "Submit this AXA claim" (or just drop it — the skill auto-triggers)
3. Claude reads `reference/secrets.json`, fills the portal form via Playwright, and logs the result

---

## Setup

1. Copy `reference/secrets.json.example` → `reference/secrets.json` and fill in your values
2. Install Playwright MCP: `npx @playwright/mcp@latest`
3. Open this folder in Claude Code

---

## Claim submission guide

See [reference/AXA_Claim_Submission_Guide.md](reference/AXA_Claim_Submission_Guide.md) for the full step-by-step portal walkthrough.

---

## Security

`secrets.json`, `AXA_Claims_Log.json`, and all receipt images are gitignored and never leave your local machine.
