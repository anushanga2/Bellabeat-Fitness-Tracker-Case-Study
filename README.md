# Case Study: Bellabeat Fitness Data Analysis
 Bellabeat is a high-tech manufacturer of health-focused products for women. Bellabeat is a successful small company, but they have the potential to become a larger player in the global smart device market. This project analyzes smart device fitness data to unlock new growth opportunities for the company.

## Business Task  
Urška Sršen, who is one of the founders of Bellabeat, would like an analysis of Bellabeat's consumer data to identify opportunities for growth. This data includes data on physical activity, heart rate and sleep monitoring. She would like to gain insights about how people are using smart devices and recommendations for how these trends can be utilized to make decisions on the marketing strategy for Bellabeat. The primary stakeholders in this case study are Urška Sršen and Sando Mur, executive team members. The major business task statement that will be explored in this case study is as follows,  
*  _Development of a comprehensive market analysis and strategy to identify and capitalize on new growth opportunities for Bellabeat._ 

## About the data  
The data for this case study is from [Fitbit Fitness Tracker Data](https://www.kaggle.com/arashnic/fitbit), a public-domain dataset on Kaggle. This dataset has been generated by respondents to a survey via Amazon Mechanical Turk between 2016/12/03 and 2016/12/05. The limitations of this data set are as follows,
* The number of users in the data set is 30. However, the users across sleep, weight, and daily activity vary across the data set. The n_distinct() function provides 33 distinct users for daily activity, 8 users for weight and 24 users for sleep. This amount of data will not follow the central limit theorem.
* The dataset is also limited and lacks information such as demographic data and user information such as gender, age, or location.  
* For the 8 users for weight, 5 users have entered the data manually and 3 via connected wifi device.

## Data Preparation    
The initial data cleaning and preparation was conducted using Bigquery SQL and Google Sheets. These steps involve checking for inconsistencies in data sets with respect to verifying overlappings, blank spaces, duplicates, etc. For the analysis in Bigquerry SQL, the datetime columns were processed to gain time-specified insights. The Bigquerry SQL queries used for the analysis are attached as a separate file in the Bellabeat Case Study folder.  For the statistical and graphical visualization in R, the following methods were followed.   
1. Load Packages  
```
install.packages("tidyverse")  
install.packages("ggplot2")  
install.packages("waffle")
install.packages("plotly")
library(tidyverse)  
library(lubridate)  
library(dplyr)  
library(ggplot2)  
library(tidyr)  
library(RColorBrewer)
library(plotly)
   
```
2. Load the Dataset and assign names
```
activity <- read.csv("Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")  
intensities <- read.csv("Fitabase Data 4.12.16-5.12.16/hourlyIntensities_merged.csv")  
sleep <- read.csv("Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")  
calories <- read.csv("Fitabase Data 4.12.16-5.12.16/hourlyCalories_merged.csv")  
weight <- read.csv("Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")
hourly_steps <- read.csv("Fitabase Data 4.12.16-5.12.16/hourlySteps_merged.csv")
sleep <- unique(sleep)
activity <- rename_with(activity, tolower)  
sleep <- rename_with(sleep, tolower)  
hourly_steps <- rename_with(hourly_steps, tolower)  
intensities <- rename_with(intensities, tolower)  
calories <- rename_with(calories, tolower)
weight <- rename_with(weight, tolower)
```  
3. Fix Formats  
```
intensities$activityhour=as.POSIXct(intensities$activityhour, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone())
intensities$time <- format(intensities$activityhour, format = "%H:%M:%S")
intensities$date <- format(intensities$activityhour, format = "%m/%d/%y")
sleep$sleepday=as.POSIXct(sleep$sleepday, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone())
sleep$date <- format(sleep$sleepday, format = "%m/%d/%y")
calories$activityhour=as.POSIXct(calories$activityhour, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone())
calories$time <- format(calories$activityhour, format = "%H:%M:%S")
calories$date <- format(calories$activityhour, format = "%m/%d/%y")
activity$activitydate=as.POSIXct(activity$activitydate, format="%m/%d/%Y", tz=Sys.timezone())
activity$date <- format(activity$activitydate, format = "%m/%d/%y")
activity <- activity %>% mutate( weekday = weekdays(as.Date(activitydate, "%m/%d/%Y")))
hourly_steps <- hourly_steps %>% 
  rename(date_time = activityhour) %>% 
  mutate(date_time = as.POSIXct(date_time, format ="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone()))
merged_activity_sleep <- merge(activity, sleep, by=c("id","date"), all.x = TRUE)
merged_data_id <- merge(merged_activity_sleep, weight, by = c("id"), all=TRUE)
```
4. Calculating the number of participants for each data set
```
n_distinct(activity$Id)  
n_distinct(calories$Id)
n_distinct(sleep$Id)  
n_distinct(intensities$Id)    
n_distinct(weight$Id)  
```  
## Analyze  
1. Calculating he summary statistics
  ```
activity %>%  
    select(totalsteps,
           totaldistance,
           sedentaryminutes, calories,
           veryactiveminutes, fairlyactiveminutes, lightlyactiveminutes) %>%
    summary()

weight %>%
    select(weightkg, bmi) %>%
    summary()

calories %>%
    select(calories) %>%
    summary()

sleep %>%
  select(totalsleeprecords, totalminutesasleep, totaltimeinbed) %>%
  summary()

hourly_steps <- hourly_steps %>% 
    separate(date_time, into = c("date", "time"), sep = " ") %>% 
    mutate(date = ymd (date))

weekdayofsteps <- (hourly_steps)%>%
    mutate(weekday = weekdays(date))%>%
    group_by (weekday,time) %>% 
    summarize(average_steps = mean(steptotal), .groups = 'drop')

weekdayofsteps$weekday <- ordered(weekdayofsteps$weekday, 
                           levels=c("Monday", "Tuesday", "Wednesday","Thursday","Friday", "Saturday", "Sunday"))
```  
  The summary statistics for each data set are as follows,    
* On average, users take about 7,638 steps per day, and the total average walking distance of users is around 5.49 km. Also, user's average calory burn is around 2304 calories. This amount is in line with the [recommended amount of calorie burn for women](https://www.healthline.com/health/fitness-exercise/how-many-calories-do-i-burn-a-day), which falls between 1,600-2,200 calories per day. However, the recommended daily steps to be walked are below the [recomended levels](https://www.medicalnewstoday.com/articles/325809) of 10,000. One of the possible reasons for this could be most of the users of Bellabeat have healthy BMI levels (Median:24.39), which is between [recommended range](https://www.healthline.com/nutrition/bmi-for-women) of 18.5-24.9. However, the average BMI value of the users is 25.19, which falls in the overweight range. With the assumption that all the data are from adults, the average number of hours slept by users per day is calculated as 6.98 hours, which is close to the [recommended sleep hour range](https://www.mayoclinic.org/healthy-lifestyle/adult-health/expert-answers/how-many-hours-of-sleep-are-enough/faq-20057898) for adults.

2. Relationship between variables
To identify the correlations between variables, following plots were obtained from R.  
* Daily steps vs Calories  
 ```  
ggplot(merged_activity_sleep,aes(totalsteps,calories))+geom_jitter(alpha=.5)+
    geom_rug(position="jitter", size=.08)+geom_smooth(size =.6)+
    labs(title= "Daily steps vs. Calories", x= "daily steps", y="calories")+
    theme_minimal()
```  
![Daily steps Vs Calories](https://github.com/anushanga2/Bellabeat-Fitness-Tracker-Case-Study/blob/main/images/Daily%20steps%20Vs%20Calories.png)

The above graph provides a correlation coefficient 0.6 between Total steps and Calories, representing a moderate positive relationship between the variables. 
* Daily steps vs Sleep
```  
ggplot(data=subset(merged_activity_sleep,!is.na(totalminutesasleep)),aes(totalsteps,totalminutesasleep))+
    geom_rug(position="jitter", size=.08)+geom_jitter(alpha=0.5)+geom_smooth(color= "blue", size =.6)+
    labs(title= "Daily steps vs. Sleep", x= "daily steps", y="minutes asleep")+
    theme_minimal()
```  
![Daily steps Vs Sleep](https://github.com/anushanga2/Bellabeat-Fitness-Tracker-Case-Study/blob/main/images/Daily%20steps%20Vs%20Sleep.png)  
The daily step vs sleep graph provides a correlation coefficient of -0.2 between Total steps and Minutes asleep, representing a low negative relationship between the variables. 
* Sedentary minutues vs Sleep
```  
ggplot(data=subset(merged_activity_sleep,!is.na(sedentaryminutes)),aes(sedentaryminutes,totalminutesasleep))+
    geom_rug(position="jitter", size=.08)+geom_jitter(alpha=0.5)+geom_smooth(color= "blue", size =.6)+
    labs(title= "Sedentary minutes vs. Sleep", x= "Sedentary minutes", y="minutes asleep")+
    theme_minimal()
```   
![Sedentary minutes vs sleep](https://github.com/anushanga2/Bellabeat-Fitness-Tracker-Case-Study/blob/main/images/Sedentary%20minutes%20vs%20sleep.png)  
The sedentary minutes vs sleep graph provides a correlation coefficient of -0.6 between Sedentary minutes and Total minutes asleep, representing a moderate negative relationship between the variables.  
* Percentage of time segments  
```  
perc_veryactive<-sum(activity$veryactiveminutes)/(sum(activity$veryactiveminutes)+sum(activity$fairlyactiveminutes)+sum(activity$lightlyactiveminutes)+sum(activity$sedentaryminutes))
perc_lightlyactive<sum(activity$lightlyactiveminutes)/(sum(activity$veryactiveminutes)+sum(activity$fairlyactiveminutes)+sum(activity$lightlyactiveminutes)+sum(activity$sedentaryminutes))
perc_fairlyactive<-sum(activity$fairlyactiveminutes)/(sum(activity$veryactiveminutes)+sum(activity$fairlyactiveminutes)+sum(activity$lightlyactiveminutes)+sum(activity$sedentaryminutes))
perc_sedentary<-sum(activity$sedentaryminutes)/(sum(activity$veryactiveminutes)+sum(activity$fairlyactiveminutes)+sum(activity$lightlyactiveminutes)+sum(activity$sedentaryminutes))
percentage <- data.frame(
    level=c("Sedentary", "Lightly", "Fairly", "Very Active"),
    minutes=c(perc_sedentary,perc_lightlyactive,perc_fairlyactive,perc_veryactive))
plot_ly(percentage, labels = ~level, values = ~minutes, type = 'pie',textposition = 'outside',textinfo = 'label+percent') %>%
  layout(title = 'Activity Level Minutes',
         xaxis = list(showgrid = FALSE, zeroline = FALSE, showticklabels = FALSE),
         yaxis = list(showgrid = FALSE, zeroline = FALSE, showticklabels = FALSE))

```  
![percentages](https://github.com/anushanga2/Bellabeat-Fitness-Tracker-Case-Study/blob/main/images/Percentage%20Activivty%20Levels.png)   
 
* Active hours of the users based on Intensity  
```  
int_new <- intensities %>%
    group_by(time) %>%
    drop_na() %>%
    summarise(mean_total_int = mean(totalintensity))

ggplot(data=int_new, aes(x=time, y=mean_total_int)) + geom_histogram(stat = "identity", fill='darkgreen') + theme(axis.text.x = element_text(angle = 90)) + labs(title="Average Total Intensity vs. Time")
```  
![average total intensity vs time](https://github.com/anushanga2/Bellabeat-Fitness-Tracker-Case-Study/blob/main/images/average%20total%20intensity%20vs%20time.png)    
Based on the above graph, the most active hours of the users are between 5.00 PM and 8.00 PM. 
* Total steps walked on each day of the week
```  
activity$weekday<-ordered(activity$weekday,levels=c("Monday", "Tuesday", "Wednesday","Thursday","Friday", "Saturday", "Sunday"))
ggplot(data=activity, aes(x=weekday, y=totalsteps))+ geom_bar(stat="identity",fill="darkblue")+ labs(title="Daily Total Steps",x="Week day",y= "Total Steps")
```  
![daily total steps](https://github.com/anushanga2/Bellabeat-Fitness-Tracker-Case-Study/blob/main/images/Daily%20total%20steps.png)  

* Active time variation  
  ```
  ggplot(weekdayofsteps, aes(x= time, y= weekday, 
                           fill= average_steps)) +
    theme(axis.text.x= element_text(angle = 90))+
    labs(title="Active Time Variation", 
         x=" ", y=" ",fill = "Average\nSteps")+
    scale_fill_gradient(low= "white", high="blue4")+
    geom_tile(color= "white",lwd =.5,linetype =1)+
    coord_fixed()+
    theme(plot.title= element_text(hjust= 0.5,vjust= 0.8, size=12),
          panel.background= element_blank())  
  ```  
![Activity time variation](https://github.com/anushanga2/Bellabeat-Fitness-Tracker-Case-Study/blob/main/images/Active%20Time%20Variation.png)  
Activity time variation graph outcomes represent that the most active users are between Wednesday 5.00-8.00 PM and Saturday 12.00-3.00 PM.  
* Grouping the users by type of activity and comparing against sleep time  

 According to the [10000steps article](https://www.10000steps.org.au/articles/healthy-lifestyles/counting-steps/),  the activity levels are detailed with respect to the number of steps. Walking at least 10,000 steps per day is recommended to reach healthy physical activity levels. 
```   
daily_average <- merged_activity_sleep %>% 
    group_by (id) %>% 
    summarise(avg_daily_steps= mean(totalsteps), 
              avg_daily_cal= mean(calories), 
              avg_daily_sleep= mean(totalminutesasleep, 
                                    na.rm = TRUE)) %>% 
    mutate(user_type= case_when(
        avg_daily_steps < 5000 ~ "sedentary",
        avg_daily_steps >= 5000 & avg_daily_steps <7499 ~"lightly active",
        avg_daily_steps >= 7499 & avg_daily_steps <9999 ~"fairly active",
        avg_daily_steps >= 10000 ~"very active"
    ))
sleep_finalavg <- merge(activity_sleep, daily_average[c("id","user_type")], by="id")
sleep_finalavg$user_type <-ordered(sleep_finalavg$user_type, levels=c("sedentary","lightly active","fairly active","very active"))

ggplot(subset(sleep_finalavg,!is.na(totalminutesasleep)),
       aes(user_type,totalminutesasleep, fill=user_type))+
    geom_boxplot()+
    stat_summary(fun="mean", geom="point", 
                 shape=23,size=2, fill="white")+
    labs(title= "Sleep vs. Activity Type", 
         x = "Activity", y =" Minutes asleep")+
    scale_fill_brewer(palette="Spectral")+
    theme(plot.title= element_text(hjust = 0.5,vjust= 0.8, size=12),
          legend.position = "none")
```    
![sleep vs activity](https://github.com/anushanga2/Bellabeat-Fitness-Tracker-Case-Study/blob/main/images/sleep%20vs%20activity.png)   
As per the results detailed above, the sleep time is higher than the recommended levels of sleep for average lightly and fairly active users. In contrast, sleep is below the recommended for typical very active users.  
## Dashboard for weekly results  
[![Dashboard 1](https://github.com/anushanga2/Bellabeat-Fitness-Tracker-Case-Study/blob/main/images/Bellabeat%20Dashboard%201.png)](https://public.tableau.com/views/BellabeatFitnessTrackerCaseStudy_16933303803030/Dashboard1?:language=en-US&:retry=yes&:display_count=n&:origin=viz_share_link)  
Tablaeu dashboard: [Bellabeat Fitness Tracker Case Study](https://public.tableau.com/views/BellabeatFitnessTrackerCaseStudy_16933303803030/Dashboard1?:language=en-US&:display_count=n&:origin=viz_share_link)

## Recommendations  
1. Encourage the users to exercise more on Sundays, Thursdays and Fridays by alerting them through notifications on the Bellabeat app or email.  
2. Educating users about healthy fitness activities or products during less active periods.  
3. Adding new features for the smart devices to motivate users to achieve healthy sleep hours ,mostly during Tuesdays, Fridays and Saturdays. The option can be given to users to notify them about their daily sleep hours to maintain a healthy lifestyle.  
4. Since users have a higher percentage of sedentary time durations, and since users' quality of sleep has reduced with higher sedentary time, the Bellabeat platforms can be used to educate the users about reducing sedentary time and engaging in more fitness activities.  
5. Improving the features of wifi-based weight monitoring to improve the fitness decisions of users. Bellabeat membership can also be used to motivate the users to achieve recommended healthy goals.  
