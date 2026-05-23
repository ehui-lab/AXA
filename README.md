# AXA Global Healthcare — Claim Submission

Automation workspace for submitting and tracking outpatient claims on the [AXA Global Healthcare Customer Online portal](https://customer.axaglobalhealthcare.com) using Claude + Playwright MCP.

---

## Actions

| Command | What it does |
|---|---|
| `/axa-setup` | First-time setup — credentials checklist, dependencies, folder structure |
| `/axa-claim` | Submit an outpatient claim from a receipt image |
| `/axa-track` | Download fresh portal CSVs and compare against local claims log |

---

## Folder layout

```
AXA/
├── README.md
├── invoices/
│   ├── receipts_inbox/                     ← drop receipts here before submitting
│   └── submitted_claims/                   ← receipts moved here after submission
├── reference/
│   ├── family_profiles.json                ← dependant config (placeholders)
│   ├── secrets.json                        ← real credentials (gitignored, local only)
│   └── secrets.json.example                ← template — copy this to get started
├── skills/
│   ├── axa-setup/SKILL.md                  ← /axa-setup prompt
│   ├── axa-claim/SKILL.md                  ← /axa-claim prompt
│   └── axa-track/SKILL.md                  ← /axa-track prompt
└── claims_data/
    ├── AXA_Claims_Log.json                 ← submitted claims log (gitignored)
    └── AXA-All-Claims-*.csv                ← portal downloads (gitignored)
```

---

## Security

`secrets.json`, `claims_data/`, and all receipt images are gitignored and never leave your local machine.
