---
name: axa-claim
description: Use this when Eric drops a medical receipt image or says "submit AXA claim", "claim this", or "file with AXA". Submits an outpatient claim on the AXA Global Healthcare portal via Playwright.
---

# axa-claim — Submit an outpatient claim

## Before starting

Read:
1. `C:\Users\Eric\Github\AXA\reference\secrets.json` — credentials and patient profiles
2. `C:\Users\Eric\Github\AXA\claims_data\AXA_Claims_Log.json` — check for duplicates

## Steps

### 1. Extract from receipt

Read the receipt image. Pull:
- Patient name, visit date, doctor, diagnosis
- Consultation fee, medication, lab amounts, **total** (HKD)
- Payment method, invoice number

Ask Eric if anything is unclear before proceeding.

### 2. Match patient

Look up in `secrets.json` → `dependants` via `match_keywords`.
Use `portal_label` for the portal dropdown, `first_name` for file naming.
One patient at a time — never combine.

### 3. Check for duplicates

Scan `claims_data\AXA_Claims_Log.json`. If same patient + visit date exists, stop and flag.

### 4. Confirm

Echo: **Patient / Visit date / Diagnosis / Total HKD** — then proceed.

### 5. Submit via Playwright

#### Login
```js
async (page) => {
  await page.goto('https://customer.axaglobalhealthcare.com');
  await page.evaluate(() => {
    document.querySelector('#UserName').value = '<portal_email from secrets.json>';
    document.querySelector('#Password').value = '<portal_password from secrets.json>';
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
- "An active session already exists" → click Continue, then re-login
- Session still active → go directly to `/Partner/Claims/SubmitInvoice`

#### Fill the form
Navigate to `https://customer.axaglobalhealthcare.com/Partner/Claims/SubmitInvoice`

| Field | ID | Value |
|---|---|---|
| Patient | `#PatientName` | `portal_label` from secrets.json |
| Over 16? | `#IsPatientAbove16` | `No` |
| Claim number? | `#IsClaimNumberAvailable` | `No` |
| Accident? | `#IsInjuryCausedByAccident` | `No` |
| Symptoms | `#Symptoms` | diagnosis verbatim |
| First aware date | `#symptomDate` | `YYYY-MM-DD` |
| Treatments | `#TreatmentsRequired` | option containing "consultation" |
| Treatment date | `#treatmentDate` | `YYYY-MM-DD` |
| Country | `#CountryOfTreatment` | `Hong Kong` |
| Already paid? | `#IsPaymentCompleted` | `Yes` |
| Amount | `#ClaimAmount` | total HKD as number |
| Currency | `#CurrencyType` | `Hong Kong Dollar` |
| Preferred payment? | `#IsPreferredPaymentMethod` | `Yes` |
| English? | `#InvoiceInEnglish` | `Yes` |
| Acknowledge popup | `#HasAcknowledged` | click if unchecked |
| Close popup | `.popup-close` | click |

Dismiss "Save password" popup: `page.keyboard.press('Escape')`

#### Upload receipt
```js
async (page) => {
  await page.locator('#browseFiles').setInputFiles('C:\\Users\\Eric\\Github\\AXA\\invoices\\receipts_inbox\\FILENAME.jpg');
}
```

#### Save & review
```js
await page.locator('#submitInvoiceReview').click();
```

#### Submit
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

Take a screenshot to capture the reference number.

### 6. Log the result

1. Append entry to `claims_data\AXA_Claims_Log.json` matching existing shape.
2. Update `summary` block (`total_claims`, `total_amount_claimed_hkd`, `last_submission_date`).
3. Move: `invoices\receipts_inbox\FILENAME.jpg` → `invoices\submitted_claims\<ref>_<FirstName>.jpg`
4. Confirm: "Logged claim `<ref>` for `<patient>`, HKD `<amount>`."

## Failure modes

- OCR ambiguous → show what was read, ask Eric
- Patient not in `secrets.json` → ask Eric
- Portal layout changed → stop, report missing field
- Login fails → retry once; if still fails, ask Eric to check `secrets.json`
