---
title: ""
date: 2024-03-29
tags: [GA4, R, path-analysis, BigQuery]
permalink: "ga4-path-analysis-in-r"
header:
  image: "images/article-4-kafka-streaming/21-schedule/project-diagram.png"
excerpt: "How to Perform Path Analysis in R with GA4 Data"
mathjax: "true"
---

# What is path analysis?

Path analysis is a technique web and product analysts use to understand how users move through a website or app. It helps uncover friction points and dark patterns in the user journey.

For example: if you notice many users returning to a previous page, that’s a red flag. Something on the current page isn’t working such as broken UI, confusing copy, or a missing step.

# Why run path analysis in R and not simply through the GA4 interface?

You can run path analysis directly in [GA4’s Explore section](https://support.google.com/analytics/answer/9317498?hl=en). It’s visually appealing, but the view quickly becomes crowded — and it only shows the most frequent paths.

With R, you’re not limited to the top results. You can count every possible path, no matter how rare. You can also mix dimensions, such as combining page location with events, to build a richer picture of user behavior.

For this analysis, we're going to use [Google's Merchandise Store BigQuery Export](https://developers.google.com/analytics/bigquery/web-ecommerce-demo-dataset?sjid=6808846338302705229-EU).

We want to understand how users move through the website and what they actually do once they are there. For this, we are interested in page views and ecommerce events.

Events like "session_start" or "first_visit" don’t tell us much about navigation, so we will exclude them. We also want the results in chronological order, so we can later reconstruct user paths.

Here is the query with comments:

```sql
select 
  -- Unique user identifier (anonymous in GA4)
  user_pseudo_id, 

  -- Extract the session_id from event_params
  (select value.int_value from unnest(event_params) where key = "ga_session_id") as session_id, 

  -- The event name (e.g. page_view, purchase)
  event_name, 

  -- Extract the page URL from event_params
  (select value.string_value from unnest(event_params) where key = "page_location") as page_location 

from 
  -- Public GA4 sample ecommerce dataset
  bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_* 

where 
  -- Exclude events that don't describe navigation or user actions
  event_name not in ("session_start", "first_visit", "scroll", "user_engagement") 

order by 
  -- Order events by user, session, and timestamp to reconstruct paths
  user_pseudo_id, session_id, event_timestamp
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-1.1.png" alt="linearly separable data">

Let's save the result as a BigQuery table.

Now, let's download the entire table in R:

```R
# Load libraries
library(tidyverse)
library(bigrquery)

# 1 Auth ----
bq_auth(path = "service-account.json")

# 2 Retrieve data ----
project_id <- "de-simmer-course-2"
dataset_id <- "articles"
table_id <- "2025_09_29_article_path_analysis_query"

  # download table
bq_table_info <- bq_table(project_id, dataset_id, table_id)
raw_data <- bq_table_download(bq_table_info, bigint = "integer64")
```

Here is what is happening:

```R
# Load libraries
library(tidyverse)
library(bigrquery)
```

tidyverse provides tools for data wrangling and visualization.
bigrquery lets you connect to and query Google BigQuery directly from R.

```R
# Authenticate to BigQuery
bq_auth(path = "service-account.json")
```
This line authenticates your R session with Google Cloud using a service account (service-account.json). It’s what allows R to access your BigQuery project and datasets. In this [guest post](https://www.simoahava.com/analytics/join-ga4-google-ads-data-in-google-bigquery/#load-data-from-the-ga4-api) that I had written for [Simo Ahava's blog](https://www.simoahava.com/), I explain, among other things, how to create a service account.

```R
# Define table identifiers
project_id <- "de-simmer-course-2"
dataset_id <- "articles"
table_id <- "2025_09_29_article_path_analysis_query"
```

These variables store the location of your BigQuery table:

- project_id = your Google Cloud project name
- dataset_id = the dataset inside that project
- table_id = the specific table you want to download

```R
# Download the table
bq_table_info <- bq_table(project_id, dataset_id, table_id)
raw_data <- bq_table_download(bq_table_info, bigint = "integer64")
```

bq_table() builds a reference to the table using the three IDs.
bq_table_download() actually retrieves the table’s data from BigQuery into R as a dataframe (tibble).

The bigint = "integer64" argument ensures large integers (like timestamps or user IDs) are preserved correctly instead of being truncated.