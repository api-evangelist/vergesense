---
title: "New filters added to the Detections API"
url: "https://headwayapp.co/vergesense-changelog/new-filters-added-to-the-detections-api-287620"
date: "2024-03-07"
author: "Alina"
feed_url: "https://headwayapp.co/vergesense-changelog/rss"
---
Improvement VergeSense has recently introduced two new filters to the /spaces/detections endpoint: filter[count][gt] - allows filtering records by values exceeding the detected person count. filter[signs_of_life][eq] - filters records based on the presence or absence of signs of life. Furthermore, the signs_of_life field has been updated to return a boolean value (or null), replacing the previous output which could be one of 0|1|null .
