---
layout: post
title: "Data exploration"
date: 2021-02-17
---

# Task 1: Daily cases
## Data wrangling

```{r cases_dw}
reported <- reported_raw %>%
  mutate_if(is.numeric, replace_na, replace = 0) %>%
  mutate(reported_date = date(reported_date)) %>%
  pivot_longer(cols = -c(1), names_to = "Status", values_to ="Case_count") %>%
  mutate(Status = str_to_sentence(Status)) %>%
  mutate(Status = fct_relevel(Status, "Active", "Recovered","Deceased"))
```
