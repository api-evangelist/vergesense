---
title: "Changing ID format for Hourly Building and Hourly Floor Metrics"
url: "https://headwayapp.co/vergesense-changelog/changing-id-format-for-hourly-building-and-hourly-floor-metrics-307956"
date: "2025-01-17"
author: "Carter, Director of Software Development"
feed_url: "https://headwayapp.co/vergesense-changelog/rss"
---
New VergeSense is improving the performance of our hourly building metrics and hourly floor metrics endpoints. As part of this work however, we are needing to change the id attribute that is returned from these endpoints. Change: id will now have the format "[building_id]|[timestamp]" Example: "334452|2024-12-20T02:00:00Z" The previous format just a numeric string ID such as "123456789" .
