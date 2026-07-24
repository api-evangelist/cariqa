---
name: Start and stop a charging session
description: Find an available connector, start a charging session for an onboarded user, monitor it, and stop it — handling the payment and provider error cases.
api: openapi/cariqa-openapi-original.yml
operations:
  - stations_around_list
  - stations_details_retrieve
  - users_charging_start_create
  - users_charging_sessions_retrieve
  - users_charging_stop_create
---

# Start and stop a charging session

Prerequisite: the user is onboarded with billing details and a default payment
method (see `cariqa-onboard-user-and-payment.md`). Base URL
`https://connect.cariqa.com/api/v1`, header `Authorization: Bearer <token>`.

## Steps

1. **Pick a station/connector** — `GET /api/v1/stations/around/`
   (`stations_around_list`) with `latitude`, `longitude`, `distance`, then
   `GET /api/v1/stations/details/` (`stations_details_retrieve`) to read live EVSE
   status. Capture the target `evse_id`.
2. **Start charging** — `POST /api/v1/users/{user_id}/charging/start/`
   (`users_charging_start_create`) with the `evse_id`. On success you receive a
   session; capture its `session_id`.
3. **Monitor** — poll `GET /api/v1/users/{user_id}/charging-sessions/{session_id}/`
   (`users_charging_sessions_retrieve`). Consumed kWh updates only if the connector
   sends progress messages; many operators only send a final CDR.
4. **Stop charging** — `POST /api/v1/users/{user_id}/charging/stop/{session_id}/`
   (`users_charging_stop_create`). The final CDR confirms the session completed.

## Error handling (see errors/cariqa-problem-types.yml + errors/cariqa-decline-codes.yml)

- `409 already_charging` — user already has/starting a session; refresh state, do not loop.
- `409 billing_data_required` — fix billing details first.
- `402 payment_error` — outstanding debt; send user to the outstanding-payments flow.
- `402 pre_authorization_failed` — if `error.payment_intent_client_secret` is present,
  complete SCA with Stripe then retry; else ask for another payment method.
- `424 station_availability_issue` / `charging_provider_error` — refresh station data
  and retry later; do not assume the session changed state.
- `409 already_stopped` on stop — treat as stopped; refresh session details.
- `500 unexpected_error` — capture any support code in `error.details` for support.

Do not aggressively auto-retry; there is no idempotency key.
