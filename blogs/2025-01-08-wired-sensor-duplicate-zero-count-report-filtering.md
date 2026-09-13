---
title: "Wired Sensor Duplicate Zero-Count Report Filtering"
url: "https://headwayapp.co/vergesense-changelog/wired-sensor-duplicate-zero-count-report-filtering-307449"
date: "2025-01-08"
author: "Tom"
feed_url: "https://headwayapp.co/vergesense-changelog/rss"
---
Improvement VergeSense wired sensor models (E104, E105, E106) are now enabled with a duplicate zero-count report filtering feature. With this feature enabled a report will be marked as a duplicate and not processed if the following conditions are met: The current report's person count is zero and the previous report's person count is zero The previous report's timestamp is less than an hour old These duplicate zero-count reports are redundant and provide no new information about the space. The one hour limit guarantees that we will process at least one report each hour.
