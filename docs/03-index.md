# Snow Data Assignment

This assignment focuses on web scraping, functions, and iteration using snow data from the Center for Snow and Avalance Studies  [Website](https://snowstudies.org/archived-data/). For simple web scraping, we want to programatically download for three sites. We don't know much about these sites, but they contain incredibly rich snow, temperature, and precipitation data. 




## Webscraping

Reading an html, extract CSV links from webpage


```r
site_url <- 'https://snowstudies.org/archived-data/'

#Read the web url
webpage <- read_html(site_url)

#See if we can extract tables and get the data that way
tables <- webpage %>%
  html_nodes('table') %>%
  magrittr::extract2(3) %>%
  html_table(fill = TRUE)
#That didn't work, so let's try a different approach

#Extract only weblinks and then the URLs!
links <- webpage %>%
  html_nodes('a') %>%
  .[grepl('24hr',.)] %>%
  html_attr('href')
```

### Data Download

Download data in a for loop


```r
#Grab only the name of the file by splitting out on forward slashes
splits <- str_split_fixed(links,'/',8)

#Keep only the 8th column
dataset <- splits[,8] 

#generate a file list for where the data goes
file_names <- paste0('data/data3/',dataset)

for(i in 1:3){
  download.file(links[i],destfile=file_names[i])
}

downloaded <- file.exists(file_names)

evaluate <- !all(downloaded)
```


Download data in a map


```r
#Map version of the same for loop (downloading 3 files)
if(evaluate == T){
  map2(links[1:3],file_names[1:3],download.file)
}else{print('data already downloaded')}
```

```
## [[1]]
## [1] 0
## 
## [[2]]
## [1] 0
## 
## [[3]]
## [1] 0
```

### Data read-in 

Read in just the snow data as a loop


```r
#Pattern matching to only keep certain files
snow_files <- file_names %>%
  .[!grepl('SG_24',.)] %>%
  .[!grepl('PTSP',.)]

#empty_data <- list()

# snow_data <- for(i in 1:length(snow_files)){
#   empty_data[[i]] <- read_csv(snow_files[i]) %>%
#     select(Year,DOY,Sno_Height_M)
# }

#snow_data_full <- do.call('rbind',empty_data)

#summary(snow_data_full)
```


### Read in the data as a map function


```r
our_snow_reader <- function(file){
  name = str_split_fixed(file,'/',2)[,2] %>%
    gsub('_24hr.csv','',.)
  df <- read_csv(file) %>%
    select(Year,DOY,Sno_Height_M) %>%
    mutate(site = name)
}

snow_data_full <- map_dfr(snow_files,our_snow_reader)
```

```
## Rows: 6211 Columns: 52
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## dbl (52): ArrayID, Year, DOY, Hour, LoAir_Min_C, LoAir_Min_Time, LoAir_Max_C...
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
## Rows: 6575 Columns: 48
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## dbl (48): ArrayID, Year, DOY, Hour, LoAir_Min_C, LoAir_Min_Time, LoAir_Max_C...
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```r
summary(snow_data_full)
```

```
##       Year           DOY         Sno_Height_M        site          
##  Min.   :2003   Min.   :  1.0   Min.   :-3.523   Length:12786      
##  1st Qu.:2008   1st Qu.: 92.0   1st Qu.: 0.350   Class :character  
##  Median :2012   Median :183.0   Median : 0.978   Mode  :character  
##  Mean   :2012   Mean   :183.1   Mean   : 0.981                     
##  3rd Qu.:2016   3rd Qu.:274.0   3rd Qu.: 1.520                     
##  Max.   :2021   Max.   :366.0   Max.   : 2.905                     
##                                 NA's   :4554
```


### Plot snow data


```r
snow_yearly <- snow_data_full %>%
  group_by(Year,site) %>%
  summarize(mean_height = mean(Sno_Height_M,na.rm=T))
```

```
## `summarise()` has grouped output by 'Year'. You can override using the `.groups`
## argument.
```

```r
ggplot(snow_yearly,aes(x=Year,y=mean_height,color=site)) + 
  geom_point() +
  ggthemes::theme_few() + 
  ggthemes::scale_color_few()
```

```
## Warning: Removed 2 rows containing missing values (geom_point).
```

<img src="03-index_files/figure-html/03 plotting snow-1.png" width="672" />


##  Extract the meteorological data URLs. Here we want you to use the `rvest` package to get the URLs for the `SASP forcing` and `SBSP_forcing` meteorological datasets.


```r
site_url <- 'https://snowstudies.org/archived-data/'

#Read the web url
webpage <- read_html(site_url)
weblinks<-webpage %>% #reuse web url and webpage from sample, using same name
  html_nodes('a') %>% # a indicates to take a node as a reference (using 'a') on the website
  .[grepl('forcing',.)] %>% #pattern matching '.' references the nodes found in 'a'.
  html_attr('href') #remove reference line for html
```

## Webscraping 2
Download the meteorological data. Use the `download_file` and `str_split_fixed` commands to download the data and save it in your data folder. You can use a for loop or a map function. 


```r
?download.file #need url and destfile (where you want the file to go and how to name it)

split<- str_split_fixed(weblinks,'/',8) #finding our destfile names
  
splitdata<-split[,8] %>% #column 8 holds the names we want
  gsub('.txt','',.)  # helps us keep track of site names?


filenames<-paste0('data/',splitdata) #creating 

for (i in 1:2){
  download.file(weblinks[i],destfile = filenames[i])
}
```

### Data Download 

Custom function to read in the data and append a site column to the data. 


```r
# this code grabs the variable names from the metadata pdf file
library(pdftools)
headers <- pdf_text('https://snowstudies.org/wp-content/uploads/2022/02/Serially-Complete-Metadata-text08.pdf') %>%
  readr::read_lines(.) %>%
  trimws(.) %>%
  str_split_fixed(.,'\\.',2) %>%
  .[,2] %>%
  .[1:26] %>%
  str_trim(side = "left")

weather_reader <- function(filenames){
  name=str_split_fixed(filenames,'/',2)[,2] #finding files
  name2=str_split_fixed(filenames,'/',4)[,2] #finding site name
  test=read.delim(filenames,header=FALSE,sep = "",col.names = headers,skip=4) %>%
    select(1:14) %>%
    mutate(site=name2) #adding column for site name
}
```

## Extract meterorological data 

Use the `map` function to read in both meteorological files. Display a summary of your tibble.


```r
weather_data <-map_dfr(filenames,weather_reader) #runs function and saves it 

summary(weather_data)
```

```
##       year          month             day             hour           minute 
##  Min.   :2003   Min.   : 1.000   Min.   : 1.00   Min.   : 0.00   Min.   :0  
##  1st Qu.:2005   1st Qu.: 3.000   1st Qu.: 8.00   1st Qu.: 5.75   1st Qu.:0  
##  Median :2007   Median : 6.000   Median :16.00   Median :11.50   Median :0  
##  Mean   :2007   Mean   : 6.472   Mean   :15.76   Mean   :11.50   Mean   :0  
##  3rd Qu.:2009   3rd Qu.: 9.000   3rd Qu.:23.00   3rd Qu.:17.25   3rd Qu.:0  
##  Max.   :2011   Max.   :12.000   Max.   :31.00   Max.   :23.00   Max.   :0  
##      second  precip..kg.m.2.s.1. sw.down..W.m.2.     lw.down..W.m.2.  
##  Min.   :0   Min.   :0.000e+00   Min.   :-9999.000   Min.   :-9999.0  
##  1st Qu.:0   1st Qu.:0.000e+00   1st Qu.:   -3.510   1st Qu.:  173.4  
##  Median :0   Median :0.000e+00   Median :   -0.344   Median :  231.4  
##  Mean   :0   Mean   :3.838e-05   Mean   :-1351.008   Mean   :-1325.7  
##  3rd Qu.:0   3rd Qu.:0.000e+00   3rd Qu.:  294.900   3rd Qu.:  272.2  
##  Max.   :0   Max.   :6.111e-03   Max.   : 1341.000   Max.   :  365.8  
##   air.temp..K.   windspeed..m.s.1.   relative.humidity.... pressure..Pa.  
##  Min.   :242.1   Min.   :-9999.000   Min.   :  0.011       Min.   :63931  
##  1st Qu.:265.8   1st Qu.:    0.852   1st Qu.: 37.580       1st Qu.:63931  
##  Median :272.6   Median :    1.548   Median : 59.910       Median :65397  
##  Mean   :272.6   Mean   : -790.054   Mean   : 58.891       Mean   :65397  
##  3rd Qu.:279.7   3rd Qu.:    3.087   3rd Qu.: 81.600       3rd Qu.:66863  
##  Max.   :295.8   Max.   :  317.300   Max.   :324.800       Max.   :66863  
##  specific.humidity..g.g.1.     site          
##  Min.   :0.000000          Length:138336     
##  1st Qu.:0.001744          Class :character  
##  Median :0.002838          Mode  :character  
##  Mean   :0.003372                            
##  3rd Qu.:0.004508                            
##  Max.   :0.014780
```

## Line plot of mean temperature

Make a line plot of mean temp by year by site (using the `air temp [K]` variable). Is there anything suspicious in the plot? Adjust your filtering if needed.


```r
# checking that we have data for both sites
unique(weather_data$site)
```

```
## [1] "SBB_SASP_Forcing_Data" "SBB_SBSP_Forcing_Data"
```

```r
#filtering to take average temperature
mean_temp_data<-weather_data %>%
  filter(year>2003)%>%
  group_by(site,year) %>%
  summarise(mean_temp=mean(air.temp..K.))
#plot
ggplot(mean_temp_data, aes(x=year,y=mean_temp,color=site))+
  geom_line()+
  theme_few()+
  labs(x='Year', y= 'Mean Temp (K)',
       title= 'Mean Temperature by Year')+
  theme(legend.position = "bottom",
        legend.box = "horizontal")+
  scale_colour_manual(labels= c("Swamp Angel Study Plot","Senator Beck Study Plot"),
                      values = c("lavender","skyblue2"))
```

<img src="03-index_files/figure-html/03.1 temp graph-1.png" width="672" />
A: It is suspicious that the mean temp is so much lower in 2003. When looking at the 2003 data, we see we only have two months of temperature data, so i filtered 2003 out. 

## Monthly average temperatures

Write a function that makes line plots of monthly average temperature at each site for a given year. Use a for loop to make these plots for 2005 to 2010. Are monthly average temperatures at the Senator Beck Study Plot ever warmer than the Swamp Angel Study Plot?
Hint: https://ggplot2.tidyverse.org/reference/print.ggplot.html


```r
#new data frame with monthly average temperature
years <- c(2005:2010)

#this is what we want the function to do
by_year<- weather_data%>%
  group_by(month,year,site) %>%
  summarise(monthly_temp=mean(air.temp..K.)) %>%
  ggplot( aes(x=month, y=monthly_temp, color=site))+
  geom_line()+
  labs(x= 'Month', y= 'Average Air Temperature (K)')

#we want functions to pull from raw datasets!
monthly_plots <- function(weather_data,years){
  by_year<- weather_data%>%
  filter(yr==year) %>%
  group_by(month,year,site) %>%
  summarise(monthly_temp=mean(air.temp..K.)) 
  
#plot
  plots<-(ggplot(by_year, aes(x=month, y=monthly_temp, color=site))+
  geom_line()+
  theme_few()+
  scale_color_few()+
  labs(x= 'Month',
       y= 'Average Air Temperature (K)',
       title=by_year$year))+
  theme(legend.position = "bottom",
        legend.box = "horizontal")
  
  print(plots)
}

#looping to plot                         
for (yr in years){
  monthly_plots(weather_data,years)
}
```

<img src="03-index_files/figure-html/03.1 monthly avg function-1.png" width="672" /><img src="03-index_files/figure-html/03.1 monthly avg function-2.png" width="672" /><img src="03-index_files/figure-html/03.1 monthly avg function-3.png" width="672" /><img src="03-index_files/figure-html/03.1 monthly avg function-4.png" width="672" /><img src="03-index_files/figure-html/03.1 monthly avg function-5.png" width="672" /><img src="03-index_files/figure-html/03.1 monthly avg function-6.png" width="672" />
A: No, Senator Beck Study Plot is never warmer than the Swamp Angel Study Plot during this time period. 
