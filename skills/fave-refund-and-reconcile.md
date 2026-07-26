---
name: Refund and reconcile transactions
description: Issue a full or partial refund and reconcile an outlet's recent transactions.
api: openapi/fave-favepay-omni-openapi.yml
operations: [refundTransaction, listOutletTransactions, acknowledgeTransaction, getTransaction]
---

# Refund and reconcile transactions

## Refund
1. Call **`refundTransaction`** (`POST /api/fpo/v1/{country_code}/transactions`) with `omni_reference`,
   `app_id`, `status: "refunded"`, and optional `partial_refund_cents`.
2. Sign with HMAC-SHA256 as `sign`.
3. Constraints: refunds are typically allowed **only on the same calendar day** (later → `stale_transaction`,
   HTTP 400). `partial_refund_cents` cannot exceed `charged_amount_cents`. Partial refunds are unavailable for
   DuitNow/PayNow. An already-refunded transaction returns HTTP 200 with `already_refunded`.

## Reconcile
1. Call **`listOutletTransactions`** (`GET /api/fpo/v1/{country_code}/outlets/{outlet_id}/transactions`)
   with `app_id`, `limit` (max 1000), and a `timestamp` lower bound (max ~31 days back). Pass `ack=false`
   to fetch only unacknowledged transactions.
2. For each processed record, call **`acknowledgeTransaction`**
   (`POST /api/fpo/v1/{country_code}/transactions/{id}`) so it drops from the unacknowledged feed.
   Note: `id` (path) is excluded from the signature.
3. Use **`getTransaction`** to fetch full detail (fees, cashback, promo breakdown) for any single record.
