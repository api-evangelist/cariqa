---
name: Onboard a user and attach a payment method
description: Create a Cariqa Connect user, complete their billing details, and attach a default Stripe payment method so they are ready to charge.
api: openapi/cariqa-openapi-original.yml
operations:
  - users_create
  - users_billing_details_update
  - users_setup_intents_retrieve
  - users_payment_methods_list
  - users_payment_methods_default_create
---

# Onboard a user and attach a payment method

Backend-to-backend flow to make a driver chargeable. All requests go to
`https://connect.cariqa.com/api/v1` (use `https://dev.connect.cariqa.com` for
development) with header `Authorization: Bearer <token>`.

## Steps

1. **Create the user** — `POST /api/v1/users/` (`users_create`). Send the user's
   profile. Capture the returned `user_id`; every subsequent call is nested under it.
2. **Complete billing details** — `PUT /api/v1/users/{user_id}/billing-details/`
   (`users_billing_details_update`). Required before charging, or start-charging
   fails with `409 billing_data_required`. Honor country-specific billing fields
   (e.g. Italy) where applicable.
3. **Get a setup intent** — `GET /api/v1/users/{user_id}/setup-intents/`
   (`users_setup_intents_retrieve`). Returns a Stripe client secret. Use it with
   the Stripe client-side SDK / payment sheet to collect and confirm the card.
   Cariqa never receives raw card data (Stripe handles PCI + PSD2/SCA).
4. **Confirm attachment** — `GET /api/v1/users/{user_id}/payment-methods/`
   (`users_payment_methods_list`) to verify the method was saved.
5. **Set the default** — `POST /api/v1/users/{user_id}/payment-methods/{pm}/default/`
   (`users_payment_methods_default_create`).

## Rules

- In development, real cards are auto-replaced with a Stripe test card ending `4242`;
  no real charge occurs (see `sandbox/cariqa-sandbox.yml`).
- Errors use `{ "detail": ..., "error": { "type": ..., "details": ... } }`
  (see `errors/cariqa-problem-types.yml`). `payment_method_error` (400) means the
  method is invalid — ask the user to re-enter.
- No idempotency-key header is supported; do not blind-retry writes.
- `403` means an invalid/expired token; `401` means the Authorization header is missing.
