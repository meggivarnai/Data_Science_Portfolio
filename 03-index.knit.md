---
title: "Snow Data Assignment: Web Scraping, Functions, and Iteration"
author: "Meggi Varnai"
date: "March 22, 2022"
output: html_document
---




# Simple web scraping

R can read html using either rvest, xml, or xml2 packages. Here we are going to navigate to the Center for Snow and Avalance Studies  [Website](https://snowstudies.org/archived-data/) and read a table in. This table contains links to data we want to programatically download for three sites. We don't know much about these sites, but they contain incredibly rich snow, temperature, and precip data. 


## Reading an html 

### Extract CSV links from webpage


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

## Data Download

### Download data in a for loop


```r
#Grab only the name of the file by splitting out on forward slashes
splits <- str_split_fixed(links,'/',8)

#Keep only the 8th column
dataset <- splits[,8] 

#generate a file list for where the data goes
file_names <- paste0('data/',dataset)

for(i in 1:3){
  download.file(links[i],destfile=file_names[i])
}
```

```
## Warning in download.file(links[i], destfile = file_names[i]): URL https://
## snowstudies.org/wp-content/uploads/2020/10/SASP_24hr.csv: cannot open destfile
## 'data/SASP_24hr.csv', reason 'No such file or directory'
```

```
## Warning in download.file(links[i], destfile = file_names[i]): download had
## nonzero exit status
```

```
## Warning in download.file(links[i], destfile = file_names[i]): URL https://
## snowstudies.org/wp-content/uploads/2022/02/SBSP_24hr.csv: cannot open destfile
## 'data/SBSP_24hr.csv', reason 'No such file or directory'
```

```
## Warning in download.file(links[i], destfile = file_names[i]): download had
## nonzero exit status
```

```
## Warning in download.file(links[i], destfile = file_names[i]): URL https://
## snowstudies.org/wp-content/uploads/2022/02/PTSP_24hr.csv: cannot open destfile
## 'data/PTSP_24hr.csv', reason 'No such file or directory'
```

```
## Warning in download.file(links[i], destfile = file_names[i]): download had
## nonzero exit status
```

```r
downloaded <- file.exists(file_names)

evaluate <- !all(downloaded)
```


### Download data in a map


```r
#Map version of the same for loop (downloading 3 files)
if(evaluate == T){
  map2(links[1:3],file_names[1:3],download.file)
}else{print('data already downloaded')}
```

```
## Warning in .f(.x[[i]], .y[[i]], ...): URL https://snowstudies.org/wp-content/
## uploads/2020/10/SASP_24hr.csv: cannot open destfile 'data/SASP_24hr.csv', reason
## 'No such file or directory'
```

```
## Warning in .f(.x[[i]], .y[[i]], ...): download had nonzero exit status
```

```
## Warning in .f(.x[[i]], .y[[i]], ...): URL https://snowstudies.org/wp-content/
## uploads/2022/02/SBSP_24hr.csv: cannot open destfile 'data/SBSP_24hr.csv', reason
## 'No such file or directory'
```

```
## Warning in .f(.x[[i]], .y[[i]], ...): download had nonzero exit status
```

```
## Warning in .f(.x[[i]], .y[[i]], ...): URL https://snowstudies.org/wp-content/
## uploads/2022/02/PTSP_24hr.csv: cannot open destfile 'data/PTSP_24hr.csv', reason
## 'No such file or directory'
```

```
## Warning in .f(.x[[i]], .y[[i]], ...): download had nonzero exit status
```

```
## [[1]]
## [1] 1
## 
## [[2]]
## [1] 1
## 
## [[3]]
## [1] 1
```

## Data read-in 

### Read in just the snow data as a loop


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

















