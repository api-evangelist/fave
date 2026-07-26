---
name: Charge via merchant scan
description: Charge a customer by processing the QR/barcode payload the merchant scanned from the Fave app.
api: openapi/fave-favepay-omni-openapi.yml
operations: [merchantScan, getTransaction]
---

# Charge via merchant scan (customer-presented QR)

Use this when the merchant scans the code the customer shows in their Fave app.

## Steps
1. Capture the customer's QR/barcode payload as `code`.
2. Call **`merchantScan`** (`POST /api/fpo/v1/{country_code}/merchant_scan`) with `omni_reference`,
   `total_amount_cents`, `app_id`, `outlet_id`, and `code`. Attach optional `details` (arbitrary JSON) —
   note it is **excluded from the signature**.
3. Compute `sign` (HMAC-SHA256 over ordered fields, excluding `sign`, `country_code`, and `details`).
4. On success you receive the transaction object with `status` and `status_code`. Reconcile later with
   **`getTransaction`** if needed.

## Rules
- `details` is not signed — never rely on it for authentication.
- Amounts are integer `_cents`; `omni_reference` max 28 chars.
- Handle `401 Unauthorized` by recomputing the signature in submission order.
