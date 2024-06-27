## Preview
Welcome to the **Cyclistic bike-share analysis case study**! A bike-share program that features more than 5,800 bicycles and 600 docking stations. 

Until now, Cyclistic’s marketing strategy relied on building general awareness and appealing to broad consumer segments. One approach that helped make these possible was the flexibility of its pricing plans: single-ride passes, full-day passes, and annual memberships. Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who purchase annual memberships are Cyclistic members.

Moreno, the director of marketing and manager has set a clear goal: **Design marketing strategies aimed at converting casual riders into annual members**. In order to do that, however, the team needs to better understand how annual members and casual riders differ. This will be done by analyzing the Cyclistic historical bike trip data to identify trends.

## Load packages
* Start by installing `tidyverse` package:
```{r install package}
install.packages("tidyverse")
```
![tidyverse package](https://github.com/Blessingdominic/Rproject/blob/main/tidyverse%20package.png)

* Load packages by using the `library()` function. Use the conflicted package to manage conflicts.
```{r load packages}
library(tidyverse)
library(conflicted)
```
![load data](https://github.com/Blessingdominic/Rproject/blob/main/load%20data.png)
## Import data  
* Use `read_csv()` function to upload *[Divvy datasets](https://www.kaggle.com/datasets/blessingukpong1/cyclistic-data/data)* here
```{r load dataset}
Q1_2019 <- read_csv("Divvy_Trips_2019_Q1.csv") 
Q1_2020 <- read_csv("Divvy_Trips_2020_Q1.csv")
```
![import data](https://github.com/Blessingdominic/Rproject/blob/main/import%20data.png)
## Wrangle data and combine into a single file  
* Compare the column names of each of the files as they need to match perfectly before using a command to join them into one file
```{r}
colnames(Q1_2019)
colnames(Q1_2020)
```
![column names](https://github.com/Blessingdominic/Rproject/blob/main/column%20names.png)
* Rename Q1_2019 columns to make them consistent with Q1_2020
```{r}
Q1_2019 <- rename(Q1_2019
                   ,ride_id = trip_id
                   ,rideable_type = bikeid
                   ,started_at = start_time
                   ,ended_at = end_time
                   ,start_station_name = from_station_name
                   ,start_station_id = from_station_id
                   ,end_station_name = to_station_name
                   ,end_station_id = to_station_id
                   ,member_casual = usertype
)

```

* Inspect the data frames and look for incongruencies
```{r}
str(Q1_2019)
str(Q1_2020) 
```
![data structure 1](https://github.com/Blessingdominic/Rproject/blob/main/data%20structure%201.png)
![data structure 2](https://github.com/Blessingdominic/Rproject/blob/main/data%20structure%202.png)n b
* Convert ride_id and rideable_type to character so that they can stack correctly. These columns are in character data types in the Q1_2020 data
```{r}
Q1_2019 <-  mutate(Q1_2019, ride_id = as.character(ride_id)
                   ,rideable_type = as.character(rideable_type))
```

* Stack individual quarter's data frames into one big data frame
```{r}
all_trips <- bind_rows(Q1_2019, Q1_2020)
```

* Remove lat, long, birthyear, and gender fields as this data was dropped in the beginning in 2020
```{r}
all_trips <- all_trips %>%  
  select(-c("start_lat", "start_lng", "end_lat", "end_lng", "birthyear", "gender",  "tripduration"))
```

## Clean up and add data to prepare for analysis

**Inspect the new table that has been created**
* List of column names
```{r}
colnames(all_trips)
```

* How many rows are in data frame?
```{r}
nrow(all_trips)
```

* Dimensions of the data frame?
```{r}
dim(all_trips)
```

* See the first 6 rows of data frame
```{r}
head(all_trips)
```

* See the last 6 rows of data frame
```{r}
tail(all_trips)
```

* See list of columns and data types (numeric, character, etc)
```{r}
str(all_trips)
```

* Statistical summary of data. Mainly for numerics
```{r}
summary(all_trips)
```

**There are a few problems we will need to fix:**

(1) In the __member_casual__ column, there are two names for members ( __Member__ and __Subscriber__) and two names for casual riders ( __Customer__ and __Casual__). We will need to consolidate that from four to two labels.

(2) The data can only be aggregated at the ride-level, which is too granular. We will want to add some additional columns of data such as day, month, year, that provide additional opportunities to aggregate the data.

(3) We will want to add a calculated field for length of ride since the __Q1_2020__ data did not have the __trip_duration__ column. We will add __ride_length__ to the entire data frame for consistency.

There are some rides where __trip_duration__ shows up as negative, including several hundred rides where Divvy took bikes out of circulation for Quality Control reasons. We will want to delete these rides.

**Solution to problem:**

(1) In the __member_casual__ column, replace __Subscriber__ with __Member__ and __Customer__ with __Casual__.

(2) Before 2020, Divvy used different labels for these two types of riders ... we will want to make our data frame consistent with their current nomenclature

* Begin by seeing how many observations fall under each usertype
```{r}
table(all_trips$member_casual)
```

* Reassign 2019 values to 2020 values
```{r}
all_trips <- all_trips %>% 
  mutate(member_casual = recode(member_casual
                                ,"Subscriber" = "member"
                                ,"Customer" = "casual"))
```

* Check to make sure the proper number of observations were reassigned
```{r}
table(all_trips$member_casual)
```

* Add columns that list the date, month, day, and year of each ride.
This will allow us to aggregate ride data for each month, day, or year ... before completing these operations we could only aggregate at the ride level
```{r}
all_trips$date <- as.Date(all_trips$started_at)
```
The default format is yyyy-mm-dd

```{r}
all_trips$month <- format(as.Date(all_trips$date), "%b")
```

```{r}
all_trips$day <- format(as.Date(all_trips$date), "%d")
```

```{r}
all_trips$year <- format(as.Date(all_trips$date), "%Y")
```

```{r}
all_trips$day_of_week <- format(as.Date(all_trips$date), "%A")
```

* Add a __ride_length__ calculation to all_trips(in seconds)
```{r}
all_trips$ride_length <- difftime(all_trips$ended_at, all_trips$started_at)
```

* Inspect the structure of the columns
```{r}
str(all_trips)
```

* Convert __ride_length__ from difftime to numeric so we can run calculations on the data
```{r}
all_trips$ride_length <- as.numeric(all_trips$ride_length)
```

* Remove "bad" data
The data frame includes a few hundred entries when bikes were taken out of docks and checked for quality by Divvy or ride_length was negative
```{r}
all_trips %>% 
  dplyr::filter(ride_length < 1)
```

* We will create a new version of the dataframe (v2) since data is being removed
```{r}
all_trips_v2 <- all_trips[!(all_trips$start_station_name == "HQ QR" | all_trips$ride_length < 0),]
```
The "|" stands for "OR"

## Conduct Descriptive Analysis
* Descriptive analysis on ride_length (all figures in seconds)
```{r}
mean(all_trips_v2$ride_length)
```
straight average (total ride length / rides)

```{r}
median(all_trips_v2$ride_length)
```
midpoint number in the ascending array of ride lengths

```{r}
max(all_trips_v2$ride_length)
```
longest ride

```{r}
min(all_trips_v2$ride_length)
```
shortest ride

* Condense the four lines above to one line using summary() on the specific attribute
```{r}
summary(all_trips_v2$ride_length)
```

* Compare members and casual users
```{r}
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = mean)
```

```{r}
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = median)
```

```{r}
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = max)
```

```{r}
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = min)
```

* See the average ride time by each day for members vs casual users
```{r}
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)
```
Notice that the days of the week are out of order. Let's fix that.

```{r}
all_trips_v2$day_of_week <- ordered(all_trips_v2$day_of_week, levels=c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))
```

* Run the average ride time by each day for members vs casual users
```{r}
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)
```

**Analyze ridership data by type and weekday**
```{r}
all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n(), average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)
```

* Let's visualize the number of rides by rider type
```{r}
all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n(), average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  %>% 
  ggplot(aes(x = weekday, y = number_of_rides, fill = member_casual)) +
  geom_col(position = "dodge")
```

* Let's create a visualization for average duration
```{r}
all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  %>% 
  ggplot(aes(x = weekday, y = average_duration, fill = member_casual)) +
  geom_col(position = "dodge")
```


## Key Findings
Based on the given data, here are the insights and observations about the usage patterns of Cyclistic bike rides:

### 1. Usage Frequency
- **Annual Members**: Annual members make up approximately 91% of the total rides.
- **Casual Riders**: Casual riders account for around 9% of the total rides.
- **Interpretation**: The majority of Cyclistic users are annual members, indicating a preference for membership among frequent riders.

### 2. Ride Duration
- **Casual Riders**: Casual riders account for approximately 87% of the total ride duration.
- **Annual Members**: Annual members make up about 13% of the total ride duration.
- **Interpretation**: Despite being fewer in number, casual riders tend to take longer rides compared to annual members. This could imply that casual riders use Cyclistic bikes for more leisurely or exploratory purposes, while annual members might use them for shorter, possibly more routine commutes.

### 3. Day-Specific Patterns
- **Highest Number of Rides**: The highest number of rides occurs on **Tuesdays**, with a total of 127,974 rides.
- **Highest Ride Duration**: The highest ride duration occurs on **Thursdays**, with an average duration of 8,451.67 seconds (approximately 2 hours and 21 minutes).
- **Interpretation**: 
  - **Tuesdays**: The high number of rides on Tuesdays could suggest that people are more active during the early part of the week, possibly due to work-related commutes or early-week errands.
  - **Thursdays**: The longer ride durations on Thursdays might indicate that users are taking longer trips as the week progresses, perhaps combining commuting with social activities or leisure before the weekend.

### Summary
- **Annual Members** dominate in terms of ride frequency, suggesting regular and consistent use, likely for commuting or routine activities.
- **Casual Riders** take significantly longer rides, indicating their rides are more likely for leisure or exploration.
- **Tuesdays** and **Thursdays** show distinct patterns in ride numbers and durations, respectively, which could help Cyclistic in planning promotions or maintenance schedules.

These insights can help Cyclistic tailor its services and marketing strategies to better cater to the needs of both annual members and casual riders, as well as optimize bike availability and maintenance schedules based on day-specific usage patterns.


## Recommendation
Based on the observations, here are three recommendations for Cyclistic:

### 1. **Enhance Membership Benefits and Marketing**
Given that annual members constitute 91% of the rides, Cyclistic should focus on:
- **Loyalty Programs**: Introduce or enhance loyalty programs that reward frequent riders. Benefits could include discounts, exclusive access to new bike models, or priority booking.
- **Referral Incentives**: Encourage existing members to refer new members by offering incentives such as free ride time or discounts on annual renewals.
- **Targeted Marketing**: Run marketing campaigns targeting potential annual members, highlighting the cost-effectiveness and convenience of an annual membership for regular commuters.

### 2. **Promote Casual Riding for Leisure and Exploration**
Since casual riders tend to take longer rides, Cyclistic can:
- **Leisure Ride Packages**: Offer packages tailored for casual riders, such as day passes or weekend specials, that cater to tourists and occasional users.
- **Tourism Partnerships**: Collaborate with local tourism boards and attractions to create guided bike tours or special offers that encourage tourists to explore the city using Cyclistic bikes.
- **Enhance Ride Experience**: Improve bike comfort and introduce features appealing to leisure riders, such as better seats, bike baskets, or GPS-enabled bikes for easier navigation.

### 3. **Optimize Bike Availability and Maintenance Schedules**
Given the distinct usage patterns on different days:
- **Tuesdays**: Increase bike availability and ensure sufficient maintenance staff on Mondays to prepare for the high demand on Tuesdays. This could include redistributing bikes to high-traffic areas.
- **Thursdays**: Since Thursdays see the longest ride durations, ensure bikes are well-maintained and in top condition mid-week to handle these longer rides. Additionally, consider providing special offers or discounts for long-duration rentals on Thursdays.
- **Data-Driven Planning**: Continuously analyze ride data to identify other trends and adjust bike distribution and maintenance schedules accordingly. Implement predictive maintenance to reduce downtime and improve the overall user experience.

### Summary
By focusing on these strategies, Cyclistic can enhance member satisfaction, attract more casual riders, and ensure optimal bike availability and maintenance, ultimately driving higher usage and better service quality.
