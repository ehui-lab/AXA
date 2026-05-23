---
name: axa-claim
description: Use this skill whenever Eric drops a medical receipt image, PDF, or photo that looks like a Hong Kong outpatient receipt — especially from Union Hospital Polyclinic, Virtus Children at 818, or one that mentions HUI HARVEY / HUI HARRIS, or an "MOSxx-xxxxxxx" invoice number. Also use when Eric says anything like "submit AXA claim", "claim this on AXA", "file this with AXA", or drops a receipt into the AXA project folder. Walks the AXA Global Healthcare portal end-to-end using Playwright MCP, fills the outpatient claim form, and submits completely.
---

# axa-claim — AXA Global Healthcare outpatient claim submission

## When to trigger

Trigger automatically when ANY of the following is true:
- A receipt image / PDF is dropped and the project folder `C:\Users\Eric\Github\AXA` is connected
- The receipt mentions Union Hospital, Virtus Children at 818, MOSxx-xxxxxxx invoice numbers, or one of the
  patient names in `C:\Users\Eric\Github\AXA\reference\family_profiles.json`
- Eric says "submit AXA claim", "claim this", "file with AXA", or similar

## Always read these files first

1. `C:\Users\Eric\Github\AXA\PROJECT.md` — the playbook
2. `C:\Users\Eric\Github\AXA\reference\family_profiles.json` — defaults + matching rules
3. `C:\Users\Eric\Github\AXA\AXA_Claims_Log.json` — to avoid duplicate submissions
4. `C:\Users\Eric\Github\AXA\.env` — credentials (AXA_PASSWORD)

## Step-by-step

### 1. Extract receipt data

Read the dropped receipt image. Pull:
- Patient name → match to portal label via `family_profiles.json`
- Visit date
- Doctor name
- Diagnosis (use verbatim for the "Describe symptoms" field)
- Consultation fee, medication, lab amounts, and **total** (HKD)
- Payment method

If any field is unclear from the image, ask Eric before proceeding.

### 2. Match the patient

Look up the patient in `family_profiles.json` using `match_keywords`.
Process ONE patient at a time — never combine.

### 3. Check for duplicates

Scan `AXA_Claims_Log.json`. If same patient + same visit date already exists, stop and flag.

### 4. Confirm before submitting

Echo a short summary:
- Patient / Visit date / Diagnosis / Amount

Then proceed — Eric trusts the automation to complete the full submission.

### 5. Drive the portal with Playwright MCP

#### Login
```js
// Use browser_run_code_unsafe — fast JS-based login, avoids timeouts
async (page) => {
  await page.goto('https://customer.axaglobalhealthcare.com');
  await page.evaluate(() => {
    document.querySelector('#UserName').value = 'YOUR_PORTAL_EMAIL';
    document.querySelector('#Password').value = 'Care2019!';  // from .env
    ['#UserName','#Password'].forEach(sel => {
      const el = document.querySelector(sel);
      el.dispatchEvent(new Event('input', {bubbles:true}));
      el.dispatchEvent(new Event('change', {bubbles:true}));
    });
    document.querySelector('input[type=submit]').click();
  });
  await page.waitForLoadState('domcontentloaded');
}
```
- If session is still active, navigate directly to `/Partner/Claims/SubmitInvoice`
- If "An active session already exists" appears, scroll down and click Continue

#### Navigate to form
```
https://customer.axaglobalhealthcare.com/Partner/Claims/SubmitInvoice
```

#### Fill the form in one evaluate() call
All real field IDs (confirmed working):

| Field | ID | Value |
|---|---|---|
| Patient | `#PatientName` | option text containing patient name |
| Over 16? | `#IsPatientAbove16` | `No` (for children) |
| Claim number? | `#IsClaimNumberAvailable` | `No` |
| Accident? | `#IsInjuryCausedByAccident` | `No` |
| Symptoms | `#Symptoms` (textarea) | diagnosis verbatim |
| First aware date | `#symptomDate` | `YYYY-MM-DD` (visit date) |
| Treatments | `#TreatmentsRequired` (multi-select) | option containing "consultation" |
| Treatment date | `#treatmentDate` | `YYYY-MM-DD` (visit date) |
| Country | `#CountryOfTreatment` | `Hong Kong` |
| Already paid? | `#IsPaymentCompleted` | `Yes` |
| Amount | `#ClaimAmount` | total HKD as number |
| Currency | `#CurrencyType` | option text "Hong Kong Dollar" |
| Preferred payment? | `#IsPreferredPaymentMethod` | `Yes` |
| English? | `#InvoiceInEnglish` | `Yes` |
| Acknowledge popup | `#HasAcknowledged` (checkbox) | click if unchecked |
| Close popup | `.popup-close` | click |

Dismiss Chrome's "Save password" popup with `page.keyboard.press('Escape')`.

#### Upload the receipt
Use `browser_run_code_unsafe` — NOT `browser_click` on `#browseFiles` (it's off-screen):
```js
async (page) => {
  await page.locator('#browseFiles').setInputFiles('C:\\Users\\Eric\\Github\\AXA\\receipts_inbox\\FILENAME.jpg');
}
```

#### Click Save & review
```js
await page.locator('#submitInvoiceReview').click();
```

#### On the summary page — submit fully
```js
async (page) => {
  const checkboxes = await page.locator('input[type=checkbox]').all();
  for (const cb of checkboxes) {
    if (!await cb.isChecked()) await cb.click();
  }
  await page.locator('input[type=submit], button[type=submit]').first().click();
  await page.waitForLoadState('domcontentloaded');
}
```

Take a screenshot to capture the reference number from the confirmation page.

### 6. Log the submission

After getting the reference number from the confirmation screenshot:

1. Append a new entry to `AXA_Claims_Log.json` matching existing entry shape.
2. Update the `summary` block (`total_claims`, `total_amount_claimed_hkd`, `last_submission_date`).
3. Move receipt: `receipts_inbox\FILENAME.jpg` → `submitted_claims\<ref>_<FirstName>.jpg`
4. Confirm to Eric: "Logged claim <ref> for <patient>, HKD <amount>."

## Hard rules

- Never combine multiple patients into one claim.
- If the receipt is not in English, stop and flag.
- If currency is not HKD, stop and ask.
- If the invoice already exists in the log (same patient + date), stop and flag.
- Never delete receipts. Always move to `submitted_claims\`.
- Receipts must be in `C:\Users\Eric\Github\AXA\receipts_inbox\` for upload to work.

## Credentials

- Portal email: `acc.hkt@gmail.com`
- Password: read from `C:\Users\Eric\Github\AXA\.env` → `AXA_PASSWORD`

## Failure modes / when to ask Eric

- Receipt OCR ambiguous (smudged numbers, covered totals) → show what was read, ask.
- Patient name not in `family_profiles.json` → ask.
- AXA portal layout changed → stop and report field that is missing.
- Login fails → retry once with the JS evaluate approach; if still fails, ask Eric to check password in `.env`.
