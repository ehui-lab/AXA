---
name: axa-track
description: Use this when Eric asks for claims status, invoice history, or "show me my claims". Downloads fresh CSVs from the AXA portal for all family members, then compares against the local claims log and displays both tables.
---

# axa-track — Claims tracking

Always download fresh CSVs — never use cached files.

## Steps

### 1. Read secrets

Read `C:\Users\Eric\Github\AXA\reference\secrets.json` to get member IDs for all family members.

### 2. Download fresh CSVs from portal

For each member (policy holder + all dependants), navigate to:
```
https://customer.axaglobalhealthcare.com/Partner/Claims/DownloadClaimsData/<member_id>
```

Log in first if needed (same login flow as axa-claim).

Save each CSV to `claims_data\` with filename pattern:
```
AXA-All-Claims-<member_id>-<DD-MM-YYYY>.csv
```

### 3. Compare log vs portal

Read `C:\Users\Eric\Github\AXA\claims_data\AXA_Claims_Log.json`.

For each entry in the log, match against the downloaded CSVs by invoice number or (patient + visit date).

Show the comparison table:

| Patient | Our Ref | Invoice | Visit Date | Amount | Portal Claim | Portal Status | Paid Date |
|---|---|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... | ... | ... |

- Match found → show portal status and paid date
- No match → "Not yet in portal"

### 4. Show 2026 claims history

After the comparison, show all 2026 claims from the CSVs sorted by patient name then ascending date:

| Patient | Date | Claim | Treatment | Provider | Amount | Paid | Status | Paid Date |
|---|---|---|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... | ... | ... | ... |

Include all family members. Abbreviate provider names (drop "HONG KONG -" prefix).
