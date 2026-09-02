---
name: vergesense-portfolio-utilization-report
description: >-
  Produce a portfolio-down occupancy utilization report from the VergeSense API — portfolio totals,
  then per-building, then the floors and space types driving the number — over a chosen time period.
api: VergeSense API
base_url: https://api.vergesense.com
operations:
- get_metricsaggregateportfolio
- buildings-1
- get_metricsaggregatebuildings{building_id}
- get_metricsaggregatefloors
- get_metricsaggregatespacetypes
- get_metricsaggregatespacegroups
generated: '2026-09-02'
method: generated
source: openapi/vergesense-api-openapi.json + https://vergesense.readme.io/reference/reference-getting-started
---

# Portfolio utilization report

Read-only. Nothing in this skill writes, so there is nothing to reverse.

## Before you start

- Send `vs-api-key: <key>` on **every** request. A missing or invalid key returns **403**, not 401,
  and there is no `WWW-Authenticate` challenge to tell you which.
- All requests must be `https://`. Plain `http://` is rejected outright.
- Your key is scoped to a **set of buildings** chosen when it was created. A building you cannot see
  will simply not appear — an empty result is not necessarily an error.
- Timestamps are ISO 8601. Both `2019-01-15T23:30:28-08:00` and `2019-01-15T23:30:28Z` are accepted;
  responses always come back as UTC `Z`.

## Steps

1. **Establish the portfolio baseline.**
   `GET /metrics/aggregate/portfolio` (`get_metricsaggregateportfolio`) with your period bounds.
   This aggregates across every building the key can access and is the number the rest of the report
   has to reconcile to.

2. **List the buildings in scope.**
   `GET /buildings` (`buildings-1`). Keep both `building_id` and `building_ref_id` for every row —
   the reference id is the customer-defined one, and it is what will match the facilities team's own
   records. Also keep the `timezone` field: aggregate windows are meaningless across a portfolio
   without it.

3. **Pull each building's aggregate.**
   `GET /metrics/aggregate/buildings/{building_id}`
   (`get_metricsaggregatebuildings{building_id}`) for the same period. Do this **serially or in small
   batches** — the limit is 120 requests/minute per source IP and there is no `RateLimit-*` header to
   tell you how much budget is left. A 429 means you have to back off blind, so pace yourself rather
   than discover the ceiling.

4. **Descend into the buildings that look wrong.**
   For each outlier, `GET /metrics/aggregate/floors/{floor_id}` (`get_metricsaggregatefloors`). This
   operation declares a **404** as well as a 400 — a 404 means the floor id does not exist, not that
   it had no traffic.

5. **Explain the number by space type and space group.**
   `GET /metrics/aggregate/space_types` (`get_metricsaggregatespacetypes`) and
   `GET /metrics/aggregate/space_groups` (`get_metricsaggregatespacegroups`), both scoped to one
   floor. This is where "the building is at 42%" turns into "the focus rooms are saturated and the
   large collaboration areas are empty".

## Watch out for

- **Wide date ranges time out.** The server cuts requests off at **30 seconds** and the provider's
  own published remedy is to narrow the query with filters, not to retry. Split a year into months
  before you split it into retries.
- **`average_person_count` and `average_person_count_when_used` are rounded to 2 decimal places**
  (since 2024-12-04). Do not re-derive percentages to more precision than that.
- **Space-type attributes are being deprecated** effective 2026-04-01. If a field you depend on
  vanishes from `/spaces/types`, check the changelog before filing a bug —
  `https://headwayapp.co/vergesense-changelog/`.
- **Version pinning is implicit.** Your key was pinned to the newest API version at the moment of its
  first ever request. Send `vs-version: YYYY-MM-DD` explicitly if the report must be reproducible.
