# AXA Global Healthcare — Claim Submission Project

This folder is a reusable workspace for submitting outpatient claims on the
AXA Global Healthcare Customer Online portal.

**Policy:** BZE00071341 (Global Health Plan Enterprise HK USD)
**Account email:** acc.hkt@gmail.com
**Portal:** https://customer.axaglobalhealthcare.com
**Reimbursement bank:** HSBC (454518887833)

---

## How to use this project

When Eric drops a receipt image (or PDF) into the Cowork chat, Claude should:

1. **Detect** that it's an AXA-relevant medical receipt (Union Hospital / Hong Kong
   outpatient / matches one of the family members below).
2. **Auto-load** the `axa-claim` skill at `.claude/skills/axa-claim/SKILL.md`.
3. **Extract** the following from the receipt:
   - Patient name
   - Visit date and time
   - Doctor name
   - Diagnosis
   - Invoice number
   - Specialist fee (HKD)
   - Pharmaceutical charges (HKD)
   - Total amount (HKD)
   - Payment method
4. **Match** the patient against `reference/family_profiles.json`.
5. **Open** the AXA portal in Claude in Chrome, log in, and walk through the
   submission flow exactly as documented in
   `reference/AXA_Claim_Submission_Guide.md`.
6. **Stop at the review/summary page.** Do NOT click final Submit.
   Eric reviews visually and clicks Submit himself.
7. After Eric reports the reference number, **append** the claim to
   `AXA_Claims_Log.json` and **move** the receipt image from
   `receipts_inbox/` to `submitted_claims/<reference_number>_<patient>.jpg`.

---

## Folder layout

```
C:\Users\Eric\Github\AXA\
├── PROJECT.md                            ← this file (read first)
├── AXA_Claims_Log.json                   ← canonical log of all submitted claims
├── receipts_inbox\                       ← drop receipts here (working area)
├── submitted_claims\                     ← receipts filed after submission
├── reference\
│   ├── AXA_Claim_Submission_Guide.md     ← step-by-step portal walkthrough
│   ├── AXA_Claim_Submission_Guide.html   ← same, HTML version
│   └── family_profiles.json              ← dependants + matching rules
└── .claude\
    └── skills\
        └── axa-claim\
            └── SKILL.md                  ← auto-trigger instructions for Claude
```

---

## Rules

- **One claim per person per visit.** Never combine multiple people.
- **Stop at the review page.** Eric clicks final Submit.
- **English receipts only.** If the receipt is not in English, flag it and stop.
- **HKD only** unless the receipt currency is clearly different — then ask.
- **Keep the originals.** Never delete receipts; move them to `submitted_claims/`.

---

## Quick command

Next time, Eric can just drop a receipt and say:

> "Submit this AXA claim."

Or simply drop the receipt with no message — the skill should still trigger.
