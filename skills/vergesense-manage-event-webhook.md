---
name: vergesense-manage-event-webhook
description: >-
  Create, inspect, modify, pause and resume a VergeSense webhook subscription so a system receives
  space_report, space_availability or motion_detected events — safely, given that this API has no
  idempotency key.
api: VergeSense API
base_url: https://api.vergesense.com
operations:
- webhookssetup
- webhooks
- webhooks-1
- webhooksid
- webhooksidenable
- webhooksiddisable
- webhooksidlogs
- webhooksid-1
generated: '2026-09-02'
method: generated
source: openapi/vergesense-api-openapi.json + https://vergesense.readme.io/reference/events
---

# Manage a VergeSense event webhook

This is the **only write surface on the VergeSense API**, and it has **no idempotency key**. Read the
safety rules before you POST anything.

## Safety rules

- **Never retry `POST /webhooks` blind.** There is no `Idempotency-Key` header and no request
  fingerprinting. A retry after an ambiguous timeout creates a *second* subscription, and both will
  deliver, so the receiving system sees every space event twice. If a create times out, call
  `GET /webhooks` (`webhooks`) first and only create again if it genuinely is not there.
- **Prefer disable over delete.** `POST /webhooks/{id}/disable` (`webhooksiddisable`) and
  `POST /webhooks/{id}/enable` (`webhooksidenable`) are a perfectly symmetric, unbounded pair.
  `DELETE /webhooks/{id}` (`webhooksid-1`) is the one **irreversible** operation on this API: there is
  no restore, and a recreated webhook gets a **new id**, breaking every downstream reference.
- **Read before you patch.** `PATCH /webhooks/{id}` (`webhooksid`) has no revision history and no
  rollback. Capture the current configuration from `GET /webhooks` first, or you cannot put it back.

## Steps

1. **Discover what is configurable.**
   `GET /webhooks/setup` (`webhookssetup`). This returns the configuration options available before
   anything is created — the closest thing this API has to a dry run. Use it to validate your
   intended shape rather than creating and patching.

2. **List what already exists.**
   `GET /webhooks` (`webhooks`). Check for a duplicate subscription pointing at the same endpoint
   before creating a new one.

3. **Create the subscription.**
   `POST /webhooks` (`webhooks-1`). Choose the event type deliberately:
   - `space_report` — fires on every updated sensor report for a space. High volume. Set the **Send
     Frequency** option to on-change if you do not need every report.
   - `space_availability` — fires only when a space flips between `available` and `occupied`. Payload
     is an **array**, one record per space the reporting sensor covers.
   - `motion_detected` — fires when motion is seen in a previously empty space. **L208 and L410
     sensors only**; L302 sensors will never emit it, so a silent stream may be a hardware fact
     rather than a bug.

4. **Meet the receiver contract.** VergeSense will only deliver to an endpoint that
   - is **HTTPS** (plain HTTP is never used),
   - answers with a **2xx**, and
   - answers **within 5 seconds**.
   Acknowledge fast and process asynchronously. If **more than 99% of deliveries fail within any
   24-hour window**, VergeSense disables the webhook automatically and emails you the top errors.

5. **Authenticate the callback.** Authentication is optional but recommended, and there is **no
   payload signature** — no HMAC, no signing secret. Your options are Basic, a bearer token, an
   arbitrary custom header, or full OAuth 2.0 (OAuth is API-only, not configurable in the UI). You
   can also ask VergeSense for its sending IP ranges and allow-list them; the list is not published.

6. **Verify delivery.**
   `GET /webhooks/{id}/logs` (`webhooksidlogs`). This is the only observability surface on delivery.

7. **Pause, resume, or take it down.**
   Disable with `webhooksiddisable`, resume with `webhooksidenable`. Only delete
   (`webhooksid-1`) when the subscription is genuinely finished with — and remember it cannot be
   undone.

## Error handling

- **403** — missing or invalid `vs-api-key`, or a key not scoped to the building. Do not retry.
- **400** — malformed request; details are in the body, but there is no error-code vocabulary.
- **409** — conflict with another request. With no idempotency key you cannot disambiguate a
  conflicting retry, so treat it as terminal, re-read state with `GET /webhooks`, and decide again.
- **429** — you exceeded 120 requests/minute from this source IP. There is no `Retry-After`; back off
  exponentially.
