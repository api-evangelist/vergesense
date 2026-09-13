---
title: "Thresholded building and floor data added to Metrics Spaces API endpoint"
url: "https://headwayapp.co/vergesense-changelog/thresholded-building-and-floor-data-added-to-metrics-spaces-api-endpoint-304227"
date: "2024-11-08"
author: "Carter, Director of Software Development"
feed_url: "https://headwayapp.co/vergesense-changelog/rss"
---
Improvement Thresholded buildings and floors (those powered by VergeSense EN1 devices) will now be returned if they have data available for them from our /metrics/spaces API endpoint. Previously, this information was not available from this endpoint but was available from our /metrics/hourly/spaces API endpoint. This update brings these metrics endpoints more consistently in line with each other.
