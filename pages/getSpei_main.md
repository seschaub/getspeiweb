---
title: "getSpei"
author: "Sergei Schaub"
date: "4 April 2019"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

# getSpei

getSpei is a package that allows you to convinently convert SPEI netCDF files into objects (data frames or lists) that are easy to use. 
Here I explain how getSpei works and how to visualize SPEI data. 

### 0. Pre-R preperation 

Before starting in R you need to download the netCDF file from [Global SPEI database](http://spei.csic.es/database.html) (here I used [SPEIbase v2.5](http://digital.csic.es/handle/10261/153475)) and save it. 


### 1. R preperation 
First, we need to install and load two packages: devtools and getSpei (note, other, required, packages are automatically installed when you run functions of the getSpei package). Second, we need to set the working directory to the folder where the SPEI netCDF files are stored. 

```{r}
# Installing and loading required packages: 
install.package("devtools")
require(devtools)
devtools::install_github('seschaub/getSpei')
require(getSpei)

# Set working directory (note that directory needs to be specified):
workdir <- "H:/Project_Forelle/New26032019/Data"
setwd(workdir)

```

### 2. spec_spei

Lets start with the spec_spei function. It allows us to convert the SPEI netCDF file into a data frame. In detail, it allows us to set a time range (years) and specify specific (multiple locations). The function also returns the exact coordinates of the center of the grid the SPEI value is taken from. 


### 2.1 Data processing
First, we need to create a data frame that contains location and its coordinates. I choose here only three locations, however, these can be expended. Second, we need to specify the SPEI (SPEI netCDF file), start year, end year and locations and run the function (This might take few minutes). 

```{r}

# create dataframe:
location_id  <- c("Vienna", "Zurich", "New York City")
longitude    <- c(16.37,8.54,-74.25)
latitude     <- c(48.20,47.37,40.71)
locations_df <- data.frame(location_id, longitude, latitude)


# specify arguments and run function:
d1 <- spec_spei(spei_files = c("spei01","spei06"), start_y = 2000, end_y = 2010, locations = locations_df)


```

# How here head() maybe simly a screenshot. 


### 2.2 Data visualization 
Lets say we want to plot the SPEI01 and SPEI06 (i.e. the one month and six month SPEI) for all three locations in the month August (month == 8).
Note that in case of many locations (>30) I would recommend split the data by locations.
I also will add in the plot a threshold-line at -1.5 that indicates the threshold for severe droughts (Yu et al. 2014).
```{r}
# load ggplot2:
install.package("ggplot2")
require(ggplot2)

# define threshold:
threshold <- -1.5

# create figure and show it:
plot1 <- ggplot()+
            geom_line(data = (d1 %>% filter(month==8)),aes(as.numeric(year),spei01,colour="SPEI01 August"))+
            geom_line(data = (d1 %>% filter(month==8)),aes(as.numeric(year),spei06,colour="SPEI06 August"))+
            geom_point(data = (d1 %>% filter(month==8)),aes(year,spei01,colour="SPEI01 August")) +
            geom_point(data = (d1 %>% filter(month==8)),aes(year,spei06,colour="SPEI06 August"))+
            geom_hline(yintercept = threshold)+ 
            facet_wrap( ~ location_id, nrow=1)+
            xlab("Year") + ylab("SPEI")+
            theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(),
                  panel.background = element_blank(),
                  panel.border = element_rect(colour = "black", fill=NA, size=1),
                  strip.background = element_blank(),
                  strip.text = element_text(size = 14),legend.position="bottom",legend.title=element_blank()) +
           scale_x_continuous(limits = c(2000, 2010), breaks = c(seq(2000, 2010, by=2)))
show(plot1)
```
# HERE WE NEED A FIGURE




### 3. all_spei

Lets turn to the all_spei function. It allows us to convert the SPEI netCDF file into a data frame. In detail, it allows us to set a time range (years) and returns us a global dataset. 


### 3.1 Data processing
The steps for the function all_spei is even simpler, we only have to specify the SPEI (SPEI netCDF file), start year and end year and run the function (This might take again few minutes).

```{r}
# specify arguments and run function:
d2 <- all_spei(spei_files = c("spei01","spei06"), start_y = 2003, end_y = 2004)

```

### 3.2 Data visualization 
To visulaize the data we need to run an extra function, i.e. all_spei_viz. This function returns a list containing of a grid and SPEI data. 
Here, we will use this function to vizulaize the six month SPEI in the month August in year 2003. Note that, differntly to the other function, we need to decide here on one SPEI and a particular month of a year. 
After we run the function we have to save the different elements of the list seperatly, decide on cutoffs and colours and visulaize it. 

```{r}
# run function
list1 <- all_spei_viz(c("spei01"), year = 2003, month = 8) 

# visualizing
grid1 <- list1$grid  # extract grid from list generated by all_spei_viz 
d1    <- list1$data  # extract SPEI data from list generated by all_spei_viz 
cutoffs <- c(-2, -1.5, -1, 1, 1.5, 2) # decide of cutoffs, we use cutoffs based on Yu et al. (2014)
color <- c("#ca0020","#f4a582","#a1d99b","#92c5de","#0571b0") # define colors
map1 <- levelplot(d1 ~ lon * lat, data = grid1, at = cutoffs, cuts = 6, pretty = T, col.regions = color) # create map
show(map1)

```
# HERE WE NEED A FIGURE





####  References: 
Yu, M., Li, Q., Hayes, M. J., Svoboda, M. D., & Heim, R. R. (2014). Are droughts becoming more frequent or severe in China based on the standardized precipitation evapotranspiration index: 1951–2010?. International Journal of Climatology, 34(3), 545-558.


Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.