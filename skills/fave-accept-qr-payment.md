---
name: Accept a FavePay Omni QR payment
description: Create a single-use QR code, let the customer pay, and confirm the result via webhook or lookup.
api: openapi/fave-favepay-omni-openapi.yml
operations: [createQrCode, getTransaction]
---

# Accept a FavePay Omni QR payment

Use this flow to charge a customer who scans a Fave QR / opens a payment URL.

## Prerequisites
- `app_id` and the HMAC secret Fave issued during partner onboarding.
- The `outlet_id` you are charging for and the market `country_code` (`MY`, `SG`, `ID`).
- Base URL: `https://omni.myfave.com` (production) or `https://omni.dino.fave.ninja` (sandbox).

## Steps
1. Build the request for **`createQrCode`** (`POST /api/fpo/v1/{country_code}/qr_codes`) with a unique
   `omni_reference` (format `[prefix]-[id]`, max 28 chars), `total_amount_cents`, `app_id`, `outlet_id`,
   and an optional `format` and `callback_url`.
2. Compute `sign`: HMAC-SHA256 over the URL-encoded fields **in submission order** (exclude `sign` and
   `country_code`), keyed by your secret. Add it as the `sign` body field.
3. Send the request; render the returned `code` as a QR / redirect the customer to the payment URL.
4. Confirm the outcome. Prefer the webhook callback (status `successful`, `status_code` 2) — always verify
   its `sign` before trusting it. If you did not receive one, poll **`getTransaction`**
   (`GET /api/fpo/v1/{country_code}/transactions?omni_reference=...`).

## Rules
- Amounts are integer minor units (`_cents`). Respect market caps (SG `total_amount_cents` max 1999900).
- There is no idempotency-key header; reuse the same `omni_reference` when correlating a payment.
- On errors, read the `error` code from the `{error, message}` envelope (see errors/fave-problem-types.yml).
