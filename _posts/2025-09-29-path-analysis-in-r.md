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

As you can see, our dataset has over 2M+ rows:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-1.2.png" alt="linearly separable data">

We are only interested in how users navigate between pages, not the full URLs. So we can remove the main domain (for example, "https://shop.googlemerchandisestore.com") and keep only the path, such as "/home" or "/products".

```R
raw_data %>% 
  
  mutate(page_location = page_location %>% str_remove_all(".*\\.com"))
```

What’s happening in the code:

`mutate()` creates or modifies a column inside the dataset. Here, it updates the existing page_location column.

`str_remove_all(".*\\.com")` uses a regular expression (regex) to remove everything before and including .com.

- `.` matches any character.
- `*` means “zero or more times”, so together `.*` means “any sequence of characters.”
- `\\.` escapes the dot, so it is treated as a literal period rather than the regex wildcard for “any character.”

The result keeps only what comes after .com, which represents the relative page path (e.g., "/home" instead of "https://shop.googlemerchandisestore.com/home"). Here is what the result looks like:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-1.4.png" alt="linearly separable data">

Since we’re working with a large dataset, processing can take some time. To test our code more efficiently, we can apply it to only the first 100,000 rows. Once we’re sure everything works as expected, we can then run it on the full dataset. We can use the `slice()` function to select a specific range of rows:

```R
raw_data %>% 
  
  slice(1:100000) %>% 
  
  mutate(page_location = page_location %>% str_remove_all(".*\\.com"))
```

This returns the first 100,000 rows:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-1.5.png" alt="linearly separable data">

The `%>%` symbol, known as the pipe operator, lets us chain multiple operations together.
It passes the result of one step directly into the next one, so we can apply a new operation to the previous result without creating intermediate variables. In our code, we first select the first 100,000 rows, and then we remove everything before the `.com` in the page_location column.

Next, let's create a `unique_session_id` column, which is the concatenation of the `user_pseudo_id` and `session_id` columns:

```R
raw_data %>% 
  
  slice(1:100000) %>% 
  
  mutate(page_location = page_location %>% str_remove_all(".*\\.com")) %>% 
  
  mutate(unique_session_id = str_c(user_pseudo_id, session_id))
```

Now we have a new column:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-1.6.png" alt="linearly separable data">

With the `select()` function, you can choose which columns to keep or remove from your dataset. In our case, we’ll remove the columns `user_pseudo_id` and `session_id`, since they’re no longer needed for the next steps of our analysis.

```R
raw_data %>% 
  
  slice(1:100000) %>% 
  
  mutate(page_location = page_location %>% str_remove_all(".*\\.com")) %>% 
  
  mutate(unique_session_id = str_c(user_pseudo_id, session_id)) %>% 
  
  select(-user_pseudo_id, -session_id)
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-1.7.png" alt="linearly separable data">

Actually, let's reorder the columns, let's have the `unique_session_id` first:

```R
raw_data %>% 
  
  slice(1:100000) %>% 
  
  mutate(page_location = page_location %>% str_remove_all(".*\\.com")) %>% 
  
  mutate(unique_session_id = str_c(user_pseudo_id, session_id)) %>% 
  
  select(-user_pseudo_id, -session_id) %>% 
  
  select(unique_session_id, event_name, page_location)
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-1.8.png" alt="linearly separable data">

Our goal is to understand which pages users visited and what actions they performed on those pages. To do this, we’ll create a new column that combines both page views and other user events. If the event is `page_view`, we’ll keep the page URL (page_location) as the value. If the event is something else, like `view_promotion` or `add_to_cart`, we’ll use the event name instead.

```R
raw_data %>% 
  
  slice(1:100000) %>% 
  
  mutate(page_location = page_location %>% str_remove_all(".*\\.com")) %>% 
  
  mutate(unique_session_id = str_c(user_pseudo_id, session_id)) %>% 
  
  select(-user_pseudo_id, -session_id) %>% 
  
  select(unique_session_id, event_name, page_location) %>% 
  
  mutate(navigation = case_when(
    event_name == "page_view" ~ page_location,
    TRUE ~ event_name
  ))
```

Here is what's happening in the code:

`mutate(navigation = case_when(...))` creates a new column called navigation.

The `case_when()` function works like an “if–else” statement:

When `event_name` is `page_view`, it sets navigation equal to the `page_location`.

For all other events (TRUE ~ `event_name`), it sets navigation equal to the name of the event (e.g., `view_promotion`, `add_to_cart`, etc.).

The result is a single column that captures both where the user was and what they did there. This navigation column will be key for building the user journey in the next steps, since it merges page views and user actions into a single chronological sequence.

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-1.9.png" alt="linearly separable data">

Now, let's keep only the useful columns, `user_pseudo_id` and `navigation`:

```R
raw_data %>% 

  # Intermediary code
  
  select(unique_session_id, navigation)
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-2.1.png" alt="linearly separable data">

Our next goal is to remove duplicate consecutive events. For example, if a user generates two consecutive `page_view` events on the same page, we want to keep only one of them.

To do this, we first need to compare each event with the one that came before it. For every `unique_session_id`, we’ll shift the `navigation` column one step forward to create a new column, `navigation_lag`.

```R
raw_data %>% 

  # Intermediary code
  
  group_by(unique_session_id) %>% 
  mutate(navigation_lag = navigation %>% lag()) %>% 
  ungroup()
```

Now, within each `unique_session_id`, the values of `navigation` have been moved down one row in the `navigation_lag` column:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-2.2.png" alt="linearly separable data">

Next, whenever the value in `navigation_lag` is `NA`, it means that this was the first page of the session. We’ll label those cases as "entrance" and keep the rest unchanged:

```R
raw_data %>% 

  # Intermediary code
  
  mutate(navigation_lag = case_when(
    is.na(navigation_lag) ~ "entrance",
    TRUE ~ navigation_lag
  ))
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-2.3.png" alt="linearly separable data">

Finally, whenever `navigation` equals `navigation_lag`, as displayed in the image below:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-2.4.png" alt="linearly separable data">

That means the user hasn't changed `page_location`, therefore these rows are redundant and we can remove them:

```R
raw_data %>% 

  # Intermediary code
  
  filter(navigation != navigation_lag)
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-2.5.png" alt="linearly separable data">

To make the paths more readable, let's rename "/" or "/store.html" with "home":

```R
raw_data %>% 

  # Intermediary code
  
  mutate(navigation = case_when(
    navigation %in% c("/", "/store.html") ~ "home",
    TRUE ~ navigation
  ))
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-2.7.png" alt="linearly separable data">

Now, for each `unique_session_id`, we can combine all navigation steps into a single sequence to represent the user’s full path. Each step will be separated by " >>> " for readability:

```R
raw_data %>% 

  # Intermediary code
  
  group_by(unique_session_id) %>% 
  reframe(path = paste(navigation, collapse = " >>> ")) %>% 
  ungroup()
```

What this does:

- `group_by(unique_session_id)` groups all events belonging to the same session.
- `paste(navigation, collapse = " >>> ")` concatenates all navigation values in order, separating them with " >>> ".
- `reframe()` returns one row per session, with a single path column that stores the full journey.

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-2.6.png" alt="linearly separable data">

Now, let's count how many times all the different user paths occur and order them in descending order:

```R
raw_data %>% 

  # Intermediary code
  
  group_by(path) %>% 
  reframe(count = n()) %>% 
  ungroup() %>% 
  
  arrange(desc(count))
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-2.6.png" alt="linearly separable data">

Now that we’ve confirmed the code works as expected, we can remove the `slice(1:100000)` line so that it runs on the entire dataset and we can save the result in a variable named `paths`:

```R
paths <- 

raw_data %>% 

  # Intermediary code
  
  group_by(path) %>% 
  reframe(count = n()) %>% 
  ungroup() %>% 
  
  arrange(desc(count))
```

Now, let's enrich out user paths. Let's count how many nodes each user path contains:

```R
paths_enriched <- paths %>% 
  
  mutate(number_of_nodes = str_count(path, " >>> ") + 1)
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-3.1.png" alt="linearly separable data">

# Data analysis

Now that we have all user paths, let’s check how many of them include the event `view_promotion`. To do that, we will label each path depending on whether this event appears in it, and then calculate how frequent it is:

```R
paths_enriched %>% 
  
  mutate(paths_with_view_promotion = case_when(
    path %>% str_detect("view_promotion") ~ "view_promotion",
    TRUE ~ "no view_promotion"
  )) %>% 
  
  group_by(paths_with_view_promotion) %>% 
  reframe(count = sum(count)) %>% 
  ungroup() %>% 
  
  mutate(pct_count = count / sum(count))
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-3.2.png" alt="linearly separable data">

It would seem that approximately 38% of user paths contain a `view_promotion` event, which suggests that promotions are clearly visible and actively engaged with on the website.

Now, let's count the occurence of the number of nodes.

```R
paths_enriched %>% 
  
  group_by(number_of_nodes) %>% 
  reframe(count = sum(count)) %>% 
  ungroup() %>% 

  mutate(pct_count = count / sum(count))
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-3.3.png" alt="linearly separable data">

We can see that almost half of all paths end after just one node. This suggests that users either land on a page that doesn’t meet their needs or are unable to find what they’re looking for by navigating further through the site.

Now, let's count the number of paths with a `purchase` event, which is obviously the most important signal for an e-commerce website:

```R
paths_enriched %>% 
  
  mutate(paths_with_purchase = case_when(
    path %>% str_detect("purchase") ~ "purchase",
    TRUE ~ "no purchase"
  )) %>% 
  
  group_by(paths_with_purchase) %>% 
  reframe(count = sum(count)) %>% 
  ungroup() %>% 
  
  mutate(pct_count = count / sum(count))
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-3.4.png" alt="linearly separable data">

We can see that only 1.45% of user paths end in a purchase.

Now we want to understand how far users progress along the purchase funnel. To do that, we’ll look at each path and identify the latest funnel event it contains, such as `view_item`, `add_to_cart`, `begin_checkout`, or `purchase`.

```R
paths_enriched %>% 
    
  filter(path %>% str_detect("purchase|add_payment_info|begin_checkout|add_shipping_info|add_to_cart|view_item")) %>% 

  mutate(latest_funnel_step = case_when(
    path %>% str_detect("purchase") ~ "purchase",
    path %>% str_detect("add_payment_info") ~ "add_payment_info",
    path %>% str_detect("begin_checkout") ~ "begin_checkout",
    path %>% str_detect("add_shipping_info") ~ "add_shipping_info",
    path %>% str_detect("add_to_cart") ~ "add_to_cart",
    path %>% str_detect("view_item") ~ "view_item",
    TRUE ~ "Other"
  )) %>% 
  
  group_by(latest_funnel_step) %>% 
  reframe(count = sum(count)) %>% 
  ungroup() %>% 
  
  mutate(pct_count = count / sum(count)) %>% 
    
  arrange(desc(count))
```

The first line keeps only the paths that contain at least one of the funnel-related events: `purchase`, `add_payment_info`, `begin_checkout`, `add_shipping_info`, `add_to_cart`, or `view_item`.

Afterwards, we label each path by its latest funnel step using `case_when()`, we create a new column called `latest_funnel_step`. The code checks for each event in descending order of funnel depth. For example, if a path contains "purchase", it’s labeled "purchase". If not, but it contains "add_payment_info", it’s labeled "add_payment_info", and so on. This ensures each path is assigned the furthest step it reached in the funnel.

Later, the data is grouped by `latest_funnel_step`, and the total number of paths for each step is calculated using `sum(count)`.

Then, We then compute the share of each funnel step relative to the total, creating a `pct_count` column. Finally, `arrange(desc(count))` sorts the funnel steps from the most common to the least common.

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-3.5.png" alt="linearly separable data">

Right away, we can see something unusual: the `begin_checkout` event appears less frequently than the `purchase` event. This likely points to a tracking issue somewhere in the funnel. In fact, analyzing data like this is one of the most effective ways to spot inconsistencies in tracking.

While reviewing the data, I noticed that many user paths start with "/signin". That’s unlikely, since a session beginning with a sign-in page doesn’t make much sense. My guess is that users are signing in, but tracking is being lost right after authentication Let’s check how many paths actually start with "/signin".

```R
paths_enriched %>% 
  
  filter(path %>% str_detect("\\/signin")) %>%

  mutate(sign_in_or_no = case_when(
    path %>% str_detect("^\\/signin") ~ "Starts with sign in",
    TRUE ~ "No start with sign in"
  )) %>% 
  
  group_by(sign_in_or_no) %>% 
  reframe(count = sum(count)) %>% 
  ungroup() %>% 
  
  mutate(pct_count = count / sum(count))
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-3.6.png" alt="linearly separable data">

It appears that authentication is the first recorded step in about 15% of user paths that include a sign-in event, this definitely points out to a tracking issue.

Now, let's see what is the average user path length based on the landing page. Maybe there are certain landing pages that lead to longer navigation. First of all, let's extract the landing page from each path:

```R
paths_enriched %>% 
  
  mutate(landing_page = path %>% str_extract(".* >>>") %>% str_replace_all(" >>>.*", "")) %>%
    
  mutate(landing_page = ifelse(is.na(landing_page), path, landing_page))
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-3.7.png" alt="linearly separable data">

Then, let's calculate a weighted average of `number_of_nodes` based on `count`:

```R
paths_enriched %>% 
  
  mutate(landing_page = path %>% str_extract(".* >>>") %>% str_replace_all(" >>>.*", "")) %>%
    
  mutate(landing_page = ifelse(is.na(landing_page), path, landing_page)) %>% 
    
  group_by(landing_page) %>% 
  reframe(weighted_avg_nodes = weighted.mean(number_of_nodes, count),
          count = sum(count)) %>% 
  ungroup() %>% 
    
  arrange(desc(count))
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-3.8.png" alt="linearly separable data">

It would seem that people that land on the homepage do explore the website quite a lot, whereas people who land on a product or category page do not stick for long. Maybe those pages deserve more attention.

Finally, let's see what are the most common exit pages:

```R
paths_enriched %>% 
  
  mutate(last_step = str_extract(path, "[^>]+$") %>% str_trim()) %>%
    
  group_by(last_step) %>% 
  reframe(count = sum(count)) %>% 
  ungroup() %>% 
    
  arrange(desc(count))
```

Here is the result:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-5-path-analysis/image-4.1.png" alt="linearly separable data">

The fact that many sessions end on category or product pages (like /Apparel) suggests users reach a point of evaluation but not conversion. That can mean:
- Product appeal issue — users aren’t convinced by what they see (price, description, imagery)
- UX issue — product discovery or comparison is frustrating (e.g., filters, sort order, load speed)