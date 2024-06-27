# BellaBeat-Casetudy
This repository contains the R markdown for the BellaBeat case study that I did as a business analysis project as part of the google data analytics course
---
title: "Capstone markdown (1)"
author: "Owuor Odhiambo"
date: "2024-06-24"
output:
  pdf_document: default
  html_document: default
---


## R Markdown introduction & loading packages

This markdown is for the purpose of providig context for my data analysis for the BellaBeat Case study.

Context: The main objective of cleaning the datasets fot this project was to get to organize all data points into seeperate tables that had to do with hourly clories and hourly intensities, Daily activities and daily sleep. This tables would then be visualized in tablue. 

```{r load packages}
install.packages("tidyverse")
install.packages("dplyr")
install.packages("tibble")

library(tidyverse)
library(dplyr)
library("tibble")

```


## Cleaning hourly calories and hourly intensities

Step 1 (importing datasets): importing the datasets hourltIntensities_merged.csv and hourlyCalories_merged.csv from the files pannel.

Step 2 (getting rid of duplicates): This was done by using the n_distinct function. 

```{r getting rid of duplicates}
data_1 <- tibble(hourlyCalories_merged)

dlist <- lapply(X = data_1, FUN = n_distinct)
print(dlist)

data_2 <- tibble(hourlyIntensities_merged)

dlist2 <- lapply(X = data_2, FUN = n_distinct)
print(dlist2)
```

Step 3 (Merging tables and formatting time): The intention was to understand the corelation between intensities and calories burnt as well as map this relationship out in a line graph to see typical audience activity in a day. To do this I had to change the date format from MM/DD/YY - HH:MM:SS to just HH:MM:SS for the first 24 hours. To do this I used pipes as well as th Cbind and format functions.

```{r MERGING TABLES AND CHANGING DATE FORMAT}
new_activty_hour <- format(mdy_hms(hourly_intensities_and_calories$ActivityHour[1:24]),
                           format = "%H:%M:%S") %>%  cbind(data_1$Calories[1:24],data_2$TotalIntensity[1:24])
```

Step 4 (creating a data frame and exporting the table): The next step was create a dataframe and export the table as a CSV file. To do this I used the data.frame function and the write.csv function.

```{r Create dataframe and export table as CSV}
DF1 <- data.frame(new_activty_hour)

#rename columns in table
colnames(DF1) <- c("Time","Calories Burnt","Intensities")

write.csv(DF1,file = "Hourlyactivity.csv")
```

## understanding the average daily intensities of our audience

Step 1 (getting the average of each activity level): The intention of understaning the average daily intensities of our audience was to be able to gain insights on their fitness level to inform our marketing strategy. To this I used the aggregate mean function. 

```{r understanding daily intensities}
SedentartyMinutes <- data.frame(aggregate(SedentaryMinutes ~ ActivityDay,dailyIntensities_merged,mean))
view(SedentartyMinutes)

LightyActiveMinutes <- data.frame(aggregate(LightlyActiveMinutes ~ ActivityDay,dailyIntensities_merged,mean))
view(LightyActiveMinutes)

FairlyActiveMinutes <- data.frame(aggregate(FairlyActiveMinutes ~ ActivityDay,dailyIntensities_merged,mean))
view(FairlyActiveMinutes)

VeryActiveMinutes <- data.frame(aggregate(VeryActiveMinutes ~ ActivityDay,dailyIntensities_merged,mean))
view(VeryActiveMinutes)
```

Step 2 (merge tables): The next step was to merge the tables using the cbind function. 

```{r Merging tables}
ActivityTable <- cbind(SedentartyMinutes, LightyActiveMinutes$LightlyActiveMinutes,FairlyActiveMinutes$FairlyActiveMinutes,VeryActiveMinutes$VeryActiveMinutes)
view(ActivityTable)
```

Step 3 (Rename columns and export table): Next I renamed the columns and exported the file as a CSV. To do this i used the colnames and write.csv functions respectively.

```{r}
colnames(ActivityTable) <- c("Sedentary.Minutes","Lightly.Active.Minutes","Fairly.Active.Minutes","Very.Active.Minutes")

write.csv(ActivityTable,file = "ActivyTable.csv")
```


## understanding sleep pattern of our audience

Step 1 (getting the average of each minutes spent in bed vs minutes spent asleep): The intention of understaning the sleep habits of our audience to gain insights on their sleep to inform our marketing strategy. To this I used the aggregate mean function.

```{r understanding sleep habits}
MinutesAsleep <- data.frame(aggregate(TotalMinutesAsleep ~ SleepDay,sleepDay_merged,mean))
view(MinutesAsleep)

MinutesInBed <- data.frame(aggregate(TotalTimeInBed ~ SleepDay,sleepDay_merged,mean))
view(MinutesInBed)
```

Step 2 (merge tables): The next step was to merge the tables using the cbind function. 

```{r Merge tables}
Sleeptable <- cbind(MinutesAsleep, MinutesInBed$TotalTimeInBed)
```

Step 3 (Rename columns and export table): Next I renamed the columns and exported the file as a CSV. To do this i used the colnames and write.csv functions respectively.

```{r Rename and export}
colnames(Sleeptable) <- c("Sleep.Day","Average.Time.Asleep","Average.Time.In.Bed")

write.csv(Sleeptable,file = "Sleeptable.csv")
```

##Cleaning and visualizing weight data 

Step 1 (Establish the weight of audiences): Use the sqldf function to find the distinct amount of audiences above the body mass index (BMI) is over and under 24.9.

```{r sqldf}
count_overweight <- sqldf("SELECT COUNT(DISTINCT(Id))
                          FROM weightLogInfo_merged
                          WHERE BMI > 24.9")

count_healthyweight <- sqldf("SELECT COUNT(DISTINCT(Id))
                          FROM weightLogInfo_merged
                          WHERE BMI < 24.9")
```

Step 2(visualize data): Use the piecent function to create a pie chart to visualize the audience weight.

```{r piecent}
count_weight <- c(5,3)

piepercent <- round(100*count_weight / sum(count_weight),1)
colors = c("red","green")
pie(count_weight,labels=paste(piepercent,"%"),col=colors,radius=1
    ,main="Percentage of Over vs Healthy Weight Users")
legend("bottomright",c("OverWeight","HealthyWeight"),cex=0.8,fill=colors)
```


##Visualize and visualize usage data
Step 1: Summarize total steps,Total distance and sedentary minutes using pipes and the select function.

```{rSummary}
dailyactivity <- dailyActivity_merged

dailyactivity %>%  
  select(TotalSteps,
         TotalDistance,
         SedentaryMinutes) %>%
  summary()
```

Step 2: Reformat dates 

```{r}
dailyactivity %>% 
  group_by(date = ActivityDate) %>% 
  arrange(date) %>% 
  ggplot() + 
  geom_bar(mapping = aes(x=date),fill="blue") +
  labs(x="Date",y="Count",
       title="No. of times users used tracker across April-May 2016")+
  theme(axis.text.x = element_text(angle = 90)) 
```
