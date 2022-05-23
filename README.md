### Enriuqe Lara's Porfolio

# Google Data Analytics Capstone Project: Cyclistic Bike-share Analysis
Cyclistic is a Chicago-based bike sharing company. The firm operates a bike-sharing program with about 5,800 bikes and 600 docking stations located across the city. Cyclistic makes bike-sharing more accessible to those with disabilities by providing reclining bikes, hand tricycles, and cargo bikes. The comapny started a successful bike-sahring service in 2016 and since then, the firm has grown to over 5200 bicycles that are geotracked and linked to a network of over 620 stations across Chicago. So far the company has relied on building general awareness and appealing to customer segments. With options for single-day passes, full-day passes, and yearly memberships, pricing flexibility has been a cornerstone of the company's success. Annual memberships, according to our Director of Marketing, will be the key to future growth.

### Scenario
You are a junior data analyst working for the marketing analyst team for Cyclistic. The Director of Marketing Lily Moreno believes that annual memeberships are the key to future success for the company. Your team wants the understand the differences between how casual and annual memebers use the cyclistic bike-sharing program. Using these insights your team will make a recommendation to convert casual riders to annual members. The information provided needs to be compelling filled with strong visuals in order for the executives at cyclistic to approve it. 

Moreno has asked you to ansers the first question: How do annual members and casual riders use Cyclistic bikes diﬀerently?


### Business Task
Identify the difference between our casual and annual riders. We'll be on the lookout for patterns and clues as to what distinguishes the two groups. We will give recommendations based on our findings on how to raise the number of annual members. Our marketing staff will then be informed of our results.

### Stakeholders
- Director of Marketing: Lily Moreno
- Cyclistic Marketing Analytics Team (my team)
- Cyclistic Executive Team 

### Data
Special thank you to Motivate International Inc. for providing the following data:
[Cyclistic Trip Data](https://divvy-tripdata.s3.amazonaws.com/index.html)
- The data seems trustworthy, and the time period appears to correspond to the proper dates and times.
- Data is understandable and reasonable

### Data Limitation
Upon further review of the data some of the data some of the start_station_name, start_station_id, end_station_name, end_station_id variables had missing values. These missing values are important to the overall anal 



## Analysis
```{r}
## Required Libraries
library(tidyverse)
library(lubridate)
library(janitor)
library(readxl)
library(dplyr)
library(scales) 
library(ggplot2)
library(Rserve)
library(RColorBrewer)
library(chron)
```

```{r}
##Importing the csv files into new data frames
df1 <- read_csv("202101-divvy-tripdata.csv")
df2 <- read_csv("202102-divvy-tripdata.csv")
df3 <- read_csv("202103-divvy-tripdata.csv")
df4 <- read_csv("202104-divvy-tripdata.csv")
df5 <- read_csv("202105-divvy-tripdata.csv")
df6 <- read_csv("202106-divvy-tripdata.csv")
df7 <- read_csv("202107-divvy-tripdata.csv")
df8 <- read_csv("202108-divvy-tripdata.csv")
df9 <- read_csv("202109-divvy-tripdata.csv")
df10 <- read_csv("202110-divvy-tripdata.csv")
df11 <- read_csv("202111-divvy-tripdata.csv")
df12 <- read_csv("202112-divvy-tripdata.csv")
```

## Data Cleaning
```{r}
#Data Cleaning 
### Combining the different data files
bike_rides <- rbind(df1, df2, df3, df4, df5, df6, df7, df8, df9, df10, df11, df12)

### Using the lubridate function to convert start date and end date to TIMESTAMPS
bike_rides$started_at <- lubridate::mdy_hm(bike_rides$started_at)
bike_rides$ended_at <- lubridate::mdy_hm(bike_rides$ended_at)

### also using lubridate to convert ride_length to time stamps as well
bike_rides$ride_length <- lubridate::hms(bike_rides$ride_length)

### Removing any empty rows
bike_rides %>%
  remove_empty()

```

## Descriptive Analysis
```{r}
# Descriptive Analysis
###Number of rides in our dataset
bike_rides %>% 
  group_by(member_casual) %>% 
  count()

###Members vs Non Members Count/Percentages
bike_rides %>% 
  group_by(member_casual) %>% 
  drop_na() %>% 
  summarise(n = n()) %>% 
  mutate(percent = round(n / sum(n), 2))

###Difference between the days bikers members and casuals like to ride?
bike_rides %>% 
  group_by(day_week_label, member_casual) %>% 
  drop_na() %>% 
  summarise(n = n()) %>% 
  mutate(percent = round(n / sum(n), 3)) %>% 
  arrange(desc(percent))

### Different ride lenghts spent by member and casuals
bike_rides %>%  
  group_by(member_casual) %>% 
  drop_na() %>%  
  summarise(avg_ride_length = mean(times(ride_length)))

###What time of years do most people ride bikes
####Creating a month_of_use lable to identify the month of the ride
bike_rides$month_of_use <- lubridate::month(bike_rides$started_at)
bike_rides %>% 
  group_by(month_of_use) %>% 
  summarise(n = n()) %>% 
  drop_na() %>% 
  mutate(percentage = round(n/sum(n), 3)) %>% 
  arrange(desc(percentage))

### most popular ride type?
rideable_type_percent <- bike_rides %>% 
                          group_by(rideable_type) %>% 
                          summarise(n = n()) %>% 
                          mutate(percentage = round(n/sum(n), 3)) %>% 
                          arrange(desc(percentage))

```

## Visualization
```{r}
### visualizing the number rides per ride type by casual or member
ride_type_label <- c("Classic bike", "Electric bike", "Docked bike")
bike_rides %>% 
  group_by(rideable_type, member_casual) %>% 
  summarize(count = n()) %>% 
  mutate(percentage = round(count/sum(count), 3)) %>% 
  ggplot(aes(x=reorder(rideable_type, -count), y=count, fill=member_casual)) + 
  geom_bar(stat="identity", position="dodge", colour="black") + theme_classic() +
  labs(title = "Number of Rides for Each Ride Type", 
       x = "Different Type of Ride", y = "Number of Bike Rides") + 
  scale_x_discrete(labels = ride_type_label) + scale_y_continuous(labels = comma) + 
  theme(plot.title = element_text(face='bold')) + 
  scale_fill_manual("legend", values=c("member" = "mistyrose", "casual" = "lightblue"))

```
```{r}
### visualizing our riders through the day
bike_rides %>%  
  group_by(member_casual, day_week_label) %>% 
  summarize(Count = n()) %>% 
  ggplot(aes(x=day_week_label, y=Count, fill=member_casual)) +
  geom_bar(colour="black", stat='identity', position = 'dodge') + theme_classic() +
  labs(title = "Number of Bike Rides for Different days of the week", 
  x = "Day of the Week", y = "Number of Bike Rides") + 
  theme(plot.title = element_text(face='bold'), legend.title= element_blank()) +
  scale_y_continuous(labels = comma) + 
  scale_fill_manual("legend", values = c("member" = "mistyrose", "casual" = "lightblue")) 


```

```{r}
## finding the hr which most people start their ride
bike_rides$start_hour <- lubridate::hour(bike_rides$started_at)
#### only show DF with Hrs that are between 5 and 21 = 6m and 8pm
bike_rides_5am_8pm <- subset(bike_rides, start_hour > 5 & start_hour <21) 

### Visual for the hrs between 5am and 8pm
number_hrs_label <- c("6am", "7am", "8am", "9am", "10am", "11am", "12pm", 
                      "1pm", "2pm", "3pm", "4pm", "5pm", "6pm", "7pm", "8pm")

bike_rides_5am_8pm %>% 
  group_by(member_casual, start_hour) %>% 
  summarize(count = n()) %>% 
  ggplot(aes(x = start_hour, y = count, fill = factor(member_casual, levels = c("member", "casual")))) +
  geom_area(stat = "identity", position = "dodge", colour = "black", alpha = 0.6) + 
  scale_y_continuous(labels = comma) + 
  scale_x_continuous(labels = number_hrs_label, breaks = 6:20) +
  labs(x = "Time of Day", y = "Number of Rides", title = "Number of rides taken by different type of members during the day") +
  theme_classic() +theme(plot.title = element_text(face="bold"),
                                                   legend.title= element_blank()) +
  scale_fill_manual("legend", values=c("member" = "mistyrose", "casual" = "lightblue"))

```

### Conclusion
From our anylsis we have found that annual and casual riders differe in many aspect, but we believe these are some of the of the most important differences.
1. Casual riders 




