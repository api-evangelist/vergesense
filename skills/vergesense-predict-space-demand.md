---
name: vergesense-predict-space-demand
description: >-
  Use the VergeSense Predict API (Large Spatial Model) to forecast how a floor plan will actually be
  used at a given attendance level, and at what attendance a meaningful number of people stop finding
  a space.
api: VergeSense API
base_url: https://api.vergesense.com
operations:
- post_predict-space-usage
- post_predict-space-shortage
- post_predict-normalized-space-type-distribution
- get_metricsaggregatefloors
- space_types
generated: '2026-09-02'
method: generated
source: openapi/vergesense-api-openapi.json + https://vergesense.readme.io/reference/predict-api
---

# Forecast space demand for a floor

The Predict API is the newest surface on this platform, shipped 2026-03-13 and powered by the
VergeSense Large Spatial Model — a model the provider says is trained on 5,000+ floorplans and 200M+
square feet of observed people-space interaction.

These are **POSTs that compute and persist nothing**. They are side-effect-free, so they are safely
retryable and there is nothing to reverse — unlike the webhook surface, which is the only real write
on this API.

## Input

All three operations take a **GeoJSON of the spaces on a floor** as the request body. See
`https://vergesense.readme.io/reference/predict-api-geojson` for the expected geometry. This is the
only place in the VergeSense contract where a floor plan is passed in rather than read out, which
means you can model a **proposed** layout, not only a measured one.

## Steps

1. **Ground the model in what the floor really is.**
   `GET /spaces/types` (`space_types`) to learn the configured space types, and
   `GET /metrics/aggregate/floors/{floor_id}` (`get_metricsaggregatefloors`) for the floor's observed
   baseline. A forecast is only interesting next to the measured number.

2. **Predict usage at an attendance level.**
   `POST /predict/space_usage` (`post_predict-space-usage`) with the floor GeoJSON and the number of
   people attempting to use those spaces. Returns the predicted number of people who will actually
   use each space.

3. **Find the breaking point.**
   `POST /predict/space_shortage` (`post_predict-space-shortage`) with the same GeoJSON. Returns the
   attendance level at which a meaningful number of people would be unable to find a space. This is
   the number a return-to-office mandate or a portfolio right-sizing decision turns on.

4. **Understand the shape of demand.**
   `POST /predict/normalized_space_type_distribution`
   (`post_predict-normalized-space-type-distribution`) returns occupancy distribution histograms
   grouped by **normalized space type and capacity tier** — the evidence for "you need fewer eight-
   person rooms and more two-person rooms", which is the recommendation these forecasts usually
   produce.

5. **Compare scenarios.** Re-run steps 2–4 against a modified GeoJSON to test a redesign. Because
   nothing is persisted, you can iterate freely.

## Watch out for

- **These three operations declare only a 200 in the contract** — no 400 and no 404, unlike the rest
  of the API. Do not assume a malformed GeoJSON produces a structured error; validate your geometry
  before sending.
- **30-second request timeout** applies here too. A very large floor plan may need to be split.
- **120 requests/minute per source IP.** Scenario sweeps hit this quickly and there is no
  `Retry-After` header to pace against.
- **Normalized space types come from the same vocabulary whose attributes are being deprecated**
  effective 2026-04-01. Check the changelog before hard-coding type names.
