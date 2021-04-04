---
layout: post
title: "Data Exploration of COVID-19 in Toronto"
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

## Data visualization

```
reported %>%
  ggplot(aes(x = reported_date, y = Case_count, fill = Status))+
  geom_bar(stat = 'identity')+
  theme_minimal()+
  labs(title = "Cases reported by day in Toronto, Canada",
       subtitle = "Confirmed and probable cases",
       x = "Date", y = "Case count",
       caption = str_c("Created by: Yichen Xu for STA303/1002,U of T\n",
                       "Source: Ontario Ministry of Health, Integrated Public Health Information System and CORES\n", 
                       date_daily[1,1]))+
  theme(legend.title = element_blank(), legend.position = c(.15,.8))+
  scale_x_date(labels = scales::date_format("%d %b %y"), 
               limits= c(date("2020-01-01"),Sys.Date()))+
  scale_y_continuous(limits =c(0, 2000))+
  scale_fill_manual(values= c("#003F5C","#86BCB6","#B9CA5D"))
  ```

# Task 2: Outbreak type
## Data wrangling

```
outbreak <- outbreak_raw %>%
  mutate(episode_week = date(episode_week)) %>%
  mutate(outbreak_or_sporadic = str_replace_all(outbreak_or_sporadic,"OB Associated", "Outbreak associated")) %>%
  mutate(outbreak_or_sporadic = str_to_sentence(outbreak_or_sporadic)) %>%
  mutate(outbreak_or_sporadic = fct_relevel(outbreak_or_sporadic, "Sporadic", "Outbreak associated")) #%>%
outbreak1 <- outbreak %>%
  group_by(episode_week) %>%
  summarise(total_cases=sum(cases),.groups = "drop")#%>%
```

## Data visualization

```
outbreak %>%
  ggplot(aes(x=episode_week, y=cases,fill=outbreak_or_sporadic))+
  geom_bar(stat = "identity")+
  theme_minimal()+
  labs(title = "Cases by outbreak type and week in Toronto, Canada",
       subtitle = "Confirmed and probable cases",
       x = "Date",
       y = "Case count",
       caption = str_c("Created by: Yichen Xu for STA303/1002,U of T\n",
                       "Source: Ontario Ministry of Health, Integrated Public Health Information System and CORES\n", 
                       date_daily[1,1]))+
  theme(legend.title = element_blank(), legend.position = c(.15,.8))+
  scale_x_date(labels = scales::date_format("%d %b %y"), 
               limits= c(date("2020-01-01"),Sys.Date()+7))+
  scale_y_continuous(limits =c(0, max(outbreak1$total_cases)))+
  scale_fill_manual(values= c("#86BCB6","#B9CA5D"))
```

# Task 3: Neighbourhoods
## Data wrangling: part 1

```
income <- nbhood_profile %>%
  filter(`_id` == 1143) %>%
  pivot_longer(-c("_id","Category","Topic","Data Source","Characteristic"), 
               names_to = "neighbourhood_name", values_to = "Percentage") %>%
  mutate(Percentage = parse_number(Percentage))
```

## Data wrangling: part 2

```
nbhoods_all <- nbhoods_shape_raw %>%
  mutate(neighbourhood_name = str_remove(AREA_NAME,"\\s\\(\\d+\\)$")) %>%
  mutate(neighbourhood_name = str_replace(neighbourhood_name,"North St.James Town","North St. James Town")) %>%
  mutate(neighbourhood_name = str_replace(neighbourhood_name,"Weston-Pellam Park","Weston-Pelham Park")) %>%
  mutate(neighbourhood_name = str_replace(neighbourhood_name,"Cabbagetown-South St.James Town","Cabbagetown-South St. James Town")) %>%
  left_join(income,by="neighbourhood_name",sort = TRUE)%>%
  left_join(nbhood_raw,by="neighbourhood_name",sort = TRUE) %>%
  rename(rate_per_100000 = rate_per_100_000_people)
```

## Data wrangling: part 3

```
nbhoods_final<-nbhoods_all %>%
  mutate_if(is.numeric, replace_na, replace = 0) %>%
  mutate(med_inc = median(Percentage)) %>%
  mutate(med_rate = median(rate_per_100000)) %>%
  mutate(nbhood_type = case_when(
    Percentage>=med_inc & rate_per_100000>=med_rate ~ "Higher low income rate, higher case rate",
    Percentage>=med_inc & rate_per_100000<=med_rate ~ "Higher low income rate, lower case rate",
    Percentage<=med_inc & rate_per_100000>=med_rate ~ "Lower low income rate, higher case rate",
    Percentage<=med_inc & rate_per_100000<=med_rate ~ "Lower low income rate, lower case rate"
  ))
```

## Data visualization

```
ggplot(data = nbhoods_final)+
  geom_sf(aes(fill = Percentage))+
  theme_map()+
  theme(legend.position = "right")+
  labs(title = "Percentage of 18 to 64 year olds living in a low income family (2015)",
       subtitle = "Neighbourhoods of Toronto, Canada",
       caption = str_c("Created by: Yichen Xu for STA303/1002,U of T\n",
                       "Source: Census Profile 98-316-X2016001 via OpenData Toronto\n", 
                       date_daily[1,1]))+
  scale_fill_gradient(name="% low income", low = "darkgreen", 
                      high = "lightgrey")
```
