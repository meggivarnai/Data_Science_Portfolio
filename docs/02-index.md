# Hayman Fire Recovery

Working on manipulating data and visualizing it in different ways.



## Data read in


```r
#Read in individual data files
ndmi <- read_csv("/Users/meggivarnai/Desktop/csu/Spring2022/DataScience/assignments/Data_Science_Portfolio_test/data/data2/hayman_ndmi.csv") %>% 
  rename(burned=2,unburned=3) %>%
  mutate(data='ndmi')
ndsi <- read_csv("/Users/meggivarnai/Desktop/csu/Spring2022/DataScience/assignments/Data_Science_Portfolio_test/data/data2/hayman_ndsi.csv") %>% 
  rename(burned=2,unburned=3) %>%
  mutate(data='ndsi')
ndvi <- read_csv("/Users/meggivarnai/Desktop/csu/Spring2022/DataScience/assignments/Data_Science_Portfolio_test/data/data2/hayman_ndvi.csv")%>% 
  rename(burned=2,unburned=3) %>%
  mutate(data='ndvi')
# Stack as a tidy dataset
full_long <- rbind(ndvi,ndmi,ndsi) %>%
  gather(key='site',value='value',-DateTime,-data) %>%
  filter(!is.na(value))
```



## Correlations

### Between NDVI and NDMI. 

What is the correlation between NDVI and NDMI? Convert the full_long dataset in to a wide dataset using the function "spread" and then make a plot that shows the correlation as a function of if the site was burned or not (x axis should be ndmi).


```r
full_wide <- spread(full_long,key='data',value='value') %>%
  filter_if(is.numeric,all_vars(!is.na(.))) %>%
  mutate(month=month(DateTime),
         year=year(DateTime))

#create variable for summer months
var.summer_months<- c(5,6,7,8)

#filter data using months we want
summer<-full_wide %>%
  filter(month %in% var.summer_months)

ggplot(summer,aes(x=ndmi,y=ndvi,color=site))+
  geom_point()+
  xlim(-0.5,0.5)+
  theme_few()+
  theme(legend.position=c(0.2,0.8))
```

<img src="02-index_files/figure-html/02 correlation-1.png" width="672" />

```r
#correlation
cor(summer$ndmi,summer$ndvi)
```

```
## [1] 0.6877053
```

A: Based off the plot, there is a positive correlation between ndmi and ndvi. I ran a correlation test and found that the correlation for summer months between NDMI and NDVI is 0.68. This initial analysis is not considering any other variables (ie burned vs unburned), which would be worth looking into to determine if there are stronger relationships present. 

### Between average NDSI and average NDVI.

What is the correlation between average NDSI (normalized snow index) for January - April and average NDVI for June-August? In other words, does the previous year's snow cover influence vegetation growth for the following summer?


```r
#variable ndsi months
var.snow_months<-c(1,2,3,4)

#variable ndvi months
var.growth_months<-c(6,7,8)

#mean NDSI per year
ndsi_avg<-full_wide[c("DateTime","ndsi","month","year","site")] %>%
  filter(month %in% var.snow_months) %>% 
  group_by(site,year) %>%
  summarize(ndsi_avg=mean(ndsi))


#mean NDVI per year
ndvi_avg<-full_wide[c("DateTime","ndvi","month","year","site")] %>%
  filter(month %in% var.growth_months) %>%
  group_by(site,year) %>%
  summarize(ndvi_avg=mean(ndvi))


#combining NDVI and NDSI into one dataset
combined<-inner_join(ndvi_avg,ndsi_avg) 

#correlation
cor(combined$ndvi_avg,combined$ndsi_avg)
```

```
## [1] 0.1803564
```

```r
#plot
ggplot(combined, aes(x=ndvi_avg, y=ndsi_avg,color=site))+
  geom_point()+
  theme_few()+
  theme(legend.position=c(0.2,0.8))
```

<img src="02-index_files/figure-html/02 avg correlation-1.png" width="672" />
A: When we are considering all sites and time frames, the data shows that there is low correlation between snow cover and vegetation growth, 0.18. This low correlation between the previous year's snow cover on vegetation growth for the following summer excludes several variables like burned and unburned, which on our plot we can see how that there is a difference between these variables. Our analysis also doesn't account for differences in runoff patterns due to elevation and slope that would impact the correlation between snow cover and vegetation growth. 

### Pre- and post-burn and burned and unburned.

How is the snow effect from question 2 different between pre- and post-burn and burned and unburned? 


```r
#create dataframe on pre-and post 2002- hayman fire.
preyears<-c(1984:2002)
postyears<-c(2003:2019)

#Snow
presnow<-ndsi_avg %>%
  filter(year %in% preyears) %>%
  group_by(year) %>%
  summarize(ndsi_avg)
  
postsnow<-ndsi_avg %>%
  filter(year %in% postyears)%>%
  group_by(year) %>%
  summarize(ndsi_avg)

unburnedsnow<-ndsi_avg %>%
  filter(site %in% 'unburned')%>%
  group_by(site)%>%
  summarize(ndsi_avg)

burnedsnow<-ndsi_avg %>%
  filter(site %in% 'burned')%>%
  group_by(site)%>%
  summarize(ndsi_avg)

#Vegetation
preveg<-ndvi_avg %>%
  filter(year %in% preyears)%>%
  group_by(year) %>%
  summarize(ndvi_avg)

postveg<-ndvi_avg %>%
  filter(year %in% postyears)%>%
  group_by(year) %>%
  summarize(ndvi_avg)

unburnedveg<-ndvi_avg %>%
  filter(site %in% 'unburned')%>%
  group_by(site)%>%
  summarize(ndvi_avg)

burnedveg<-ndvi_avg %>%
  filter(site %in% 'burned')%>%
  group_by(site)%>%
  summarize(ndvi_avg)

#correlations
Pre<-cor(presnow$ndsi_avg,preveg$ndvi_avg)
Post<-cor(postsnow$ndsi_avg,postveg$ndvi_avg)
Unburned<-cor(unburnedsnow$ndsi_avg,unburnedveg$ndvi_avg)
Burned<-cor(burnedsnow$ndsi_avg,burnedveg$ndvi_avg)

#Answers in a table
answers<-data.frame(Pre,Post,Unburned,Burned)
answers
```

```
##          Pre    Post    Unburned     Burned
## 1 0.07340662 0.24394 -0.03100231 0.08700527
```
Based off our correlations, we see no relationship between ndsi vs ndvi when we consider pre and post fire, as well as unburned and burned. Other then Post, all other correlations are less than question 2. This alludes to other underlying variables impacting our values such as shifts in climate year to year, soil type, and runoff rates that could be influencing these relationships. 

## Maximum NDVI and NDSI values

August is the greenest month on average.


```r
#filtering vegetation index and creating new column with averages. 
green_avgmonth<-ndvi%>% 
  pivot_longer(
    cols= c("burned","unburned"),
    names_to = ("site"),
    values_to = ("value"),
    values_drop_na=TRUE) %>%
  mutate(month=month(DateTime))%>%
  group_by(month) %>%
  summarize(ndvi_avgmonth=mean(value)) %>%
  filter(ndvi_avgmonth == max(ndvi_avgmonth))

green_avgmonth
```

```
## # A tibble: 1 × 2
##   month ndvi_avgmonth
##   <dbl>         <dbl>
## 1     8         0.387
```


January is the snowiest month on average. 

```r
#filtering snow index and creating new column with averages. 
snow_avgmonth<-ndsi%>%
  pivot_longer(
    cols= c("burned","unburned"),
    names_to = ("site"),
    values_to = ("value"),
    values_drop_na=TRUE) %>%
  mutate(month=month(DateTime))%>%
  group_by(month) %>%
  summarize(ndsi_avgmonth=mean(value)) %>%
  filter(ndsi_avgmonth == max(ndsi_avgmonth))
```



