# Rproject
## Preview
Welcome to the **Cyclistic bike-share analysis case study**! A bike-share program that features more than 5,800 bicycles and 600 docking stations. 

Until now, Cyclistic’s marketing strategy relied on building general awareness and appealing to broad consumer segments. One approach that helped make these possible was the flexibility of its pricing plans: single-ride passes, full-day passes, and annual memberships. Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who purchase annual memberships are Cyclistic members.

Moreno, the director of marketing and manager has set a clear goal: **Design marketing strategies aimed at converting casual riders into annual members**. In order to do that, however, the team needs to better understand how annual members and casual riders differ. This will be done by analyzing the Cyclistic historical bike trip data to identify trends.

## Load packages
* Start by installing `tidyverse` package:
```{r install package}
install.packages("tidyverse")
```
![Screenshot 2024-06-27 184723](https://1drv.ms/i/c/2cc54bb9dec5ce43/EVZnGmF7k6xBgGjfvosTXGgBB4robpnAsusTDOZWv075fw?e=codJkL)

* Load packages by using the `library()` function. Use the conflicted package to manage conflicts.
```{r load packages}
library(tidyverse)
library(conflicted)
```

## Import data  
* Use `read_csv()` function to upload *[Divvy datasets](https://www.kaggle.com/datasets/blessingukpong1/cyclistic-data/data)* here
```{r load dataset}
Q1_2019 <- read_csv("Divvy_Trips_2019_Q1.csv") 
Q1_2020 <- read_csv("Divvy_Trips_2020_Q1.csv")
```
