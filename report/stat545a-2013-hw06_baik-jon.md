Avoiding Parking Tickets in Vancouver, B.C.
========================================================
### Jonathan Baik
### STAT 545A Homework 6
### Oct 21 2013
<hr>

## Contents
* The Data
* The Different Types of Parking Offences
* Types of Cars that Were Ticketed
* When to Avoid Parking in Vancouver
* Repeat Offenders
* Most Ticketed Locations
* Conclusions




## The Data

We investigate a data set containing information on all the parking tickets issued in the city of Vancouver between January 1, 2004 and September 25, 2008. The data set originates from [The Vancouver Sun's website](http://www.vancouversun.com/parking/basic-search.html) where the an interface is provided for the user to query the parking ticket database. 

We obtained a dump of the entire data set from a local Vancouverite programmer, David Grant,[David Grant's website]

We load the data and perform a `subset` operation to remove the `Oceania` continent, as recommended by Jenny. A quick sanity check is done by checking the structure of the data set with `str` function after importing and subsetting the data.


```
## Warning: cannot open file 'gapminderDataFiveYear.txt': No such file or
## directory
```

```
## Error: cannot open the connection
```

```
## Error: object 'gDat' not found
```

```
## Error: object 'gDat' not found
```

```
## Error: object 'gDat' not found
```

```
## Error: object 'gDat' not found
```

```
## Error: object 'gDat' not found
```



```r
str(gDat)
```

```
## Error: object 'gDat' not found
```


## `ggplot2` Figures from the Gapminder Data Set

### Depict the maximum and minimum of GDP per capita for all continents

We re-vist one visualization task from [the previous homework](http://rpubs.com/jonnybaik/stat545a-2013-hw04_baik-jon) to compare the `lattice` plot with the `ggplot2` plot. This task is from Jenny's menu of data aggregation tasks [(link to task)](http://www.stat.ubc.ca/~jenny/STAT545A/hw04_univariateLattice.html#depict-the-maximum-and-minimum-of-gdp-per-capita-for-all-continents.). We alter the task very slightly by including the 10% trimmed mean in addition to the minimum and maximum GDP per capita.

We begin by using `ddply` to aggregate the data. We use a custom function in `ddply` to retrieve the countries with the minimum and maximum GDP per capita for each year.


```r
# Min/Max GDP by year and Country
minMaxGDP <- ddply(gDat, ~year + continent, .drop = FALSE, function(x) {
    minCountry = as.character(x$country[which.min(x$gdpPercap)])
    minGdp = min(x$gdpPercap)
    maxCountry = as.character(x$country[which.max(x$gdpPercap)])
    maxGdp = max(x$gdpPercap)
    trimMean = mean(x$gdpPercap, trim = 0.1)
    return(data.frame(gdpPercap = c(minGdp, maxGdp, trimMean), country = c(minCountry, 
        maxCountry, ""), stat = c("minGdpPercap", "maxGdpPercap", "trimMean")))
})
```

```
## Error: object 'gDat' not found
```


We display the aggregated data in a "long" table below. To save some space, *we display only the first 20 rows* of the 144 rows in the table.


```
## Error: object 'minMaxGDP' not found
```

```
## Error: object 'minMaxGDPx' not found
```


Now, we create the accompanying figure for the above table in `lattice` and `ggplot2`. For each continent, we plot the progression of the minimum, the maximum, and the 10% trimmed mean of the GDP per capita over the years in the data set. A locally fitted (LOESS) regression line is plotted over the points in the data. We make one change from our original plot in the previous homework - the `gdpPercap` variable on the y-axis is plotted in the `log` scale to allow us better compare the lines.


```r
# Plot points and a loess smoothed fit in lattice
xyplot(gdpPercap ~ year | continent, 
       group=stat, 
       data=minMaxGDP, 
       grid='h',
       auto.key = TRUE, 
       type=c("p", "smooth"), 
       scales=list(y = list(log = 10)))
```

```
## Error: object 'minMaxGDP' not found
```

```r

# Do the same with ggplot2
ggplot(data=minMaxGDP,
       aes(x=year, y=gdpPercap, group=stat, colour=stat)) +
  geom_point() +
  geom_smooth(fill=NA, method="loess") +
  facet_wrap(~continent) +
  scale_y_log10(breaks = trans_breaks("log10", function(x) 10^x),
                labels = trans_format("log10", math_format(10^.x)))
```

```
## Error: object 'minMaxGDP' not found
```


We focus on comparing the `lattice` plot with the `ggplot2` plot. For a narrative of the plots, please refer to the previous homework.

For the `lattice` plot, this plot was built without delving into making our own `panel` functions, which is where the true customization power lies in `lattice`. That being said, we were able to produce a nice figure quite easily using just the `xyplot` function. Faceting is achieved by simply providing a formula, grouping is expressed explicitely, and `lattice` takes care of the colours. Grid lines are added by simply calling `grid='h'`, and the type of plot is provided in a character vector (`type=c("p", "smooth")` provides points and a smoothed LOESS line). I found that changing the axis to the log scale was unintuitive, requiring us to provide a `list` to the `scales` argument. On the plus side, the scales are rather well displayed by default in `lattice`.

In the `ggplot2` plot, we use the grammar of graphics to build our plot. The data set and the main aesthetics are provided in the `ggplot` call, where we specify the variable for the x-axis, the variable for the y-axis, the grouping variable, and the colour. We then tell `ggplot` what to draw on the plot via calls to geom layers. We use `geom_point` to add points to the plot, and `geom_smooth` to add a smooth line fit to the points. Faceting is explicitely called using `facet_wrap`, and the faceting variable can be called using a formula. The y-axis can be transformed to the log scale by adding the `scale_y_log10` call, but to make the scale look nice like the `lattice` plot, we used the package `scales` by Hadley Wickham.

The default order of the facets are different in the `lattice` plots when compared to plots made with `ggplot2`. The plots are added from the bottom left to to the top right, following the factor ordering in the data, when using `lattice`. On the other hand, `ggplot2` adds the plots from the top left to the bottom right, which I found to be more intuitive.

### The Bubble Plot

We now focus our attention to creating Hans Rosling's infamous bubble plot. We will have the (log) GDP per capita on the x-axis, and Life Expectancy on the y-axis. The size of the points on the plot will represent the population size, and the colour of the points will represent the continent. We will focus on the data at years 1952 and 2007, the earliest and latest data points in the `gapminder` data set.

To help us with creating the bubble plot in `lattice`, Jenny provides a supplementary data set that contains nice colours and sorts the countries by size so that smaller countries are not hidden behind larger countries. We keep Oceania in the following plots.


```r
# Get the copy of the data set that is ordered and given with colours
gdURL <- "http://www.stat.ubc.ca/~jenny/notOcto/STAT545A/examples/gapminder/data/gapminderWithColorsAndSorted.txt"
gDatSorted <- read.delim(file = gdURL, as.is = 7)

# Get the data for 1952 and 2007
gDatSub <- subset(gDatSorted, year == 1952 | year == 2007)

# Continent Colours
gdURL <- "http://www.stat.ubc.ca/~jenny/notOcto/STAT545A/examples/gapminder/data/gapminderContinentColors.txt"
continentColors <- read.delim(file = gdURL, as.is = 3)  # protect color
```


The `lattice` plot code is adapted from Jenny's code from the [course materials](http://www.stat.ubc.ca/~jenny/STAT545A/block10_latticeNittyGritty.html) with some very minor modifications.

We begin by constructing a list to define the legend for the `lattice` plot. In particular, the location of the plot is given, as well as parameters for the labels, such as the text and the colour.

Then the bubble plot is made using the `xyplot` scatterplot function. We specify the size of the points by defining the `cex` parameter. The area of the points are related to the population size, and a custom panel function is used to fill in the country colours using Jenny's pre-specified colours in the data set.


```r
# Define continent key
continentKey <-
  with(continentColors,
       list(x=0.95, y=0.05, corner=c(1, 0),
            text=list(as.character(continent)),
            points=list(pch=21, col='grey20', fill=color)))

# The bubble plot for the 2007 data, taken from Jenny's code
xyplot(lifeExp ~ gdpPercap | factor(year), data=gDatSub, aspect=2/3,
       grid=TRUE, scales=list(x=list(log=10, equispaced.log=FALSE)),
       cex=sqrt(gDatSub$pop/pi)/1500, fill.color=gDatSub$color,
       col='grey20', key=continentKey,
       panel=function(x, y, ..., cex, fill.color, subscripts) {
         panel.xyplot(x, y, cex=cex[subscripts],
                      pch=21, fill=fill.color[subscripts], ...)
         })
```

![plot of chunk bubblePlotLattice](figure/bubblePlotLattice.png) 


The `ggplot2` figure is, in my opinion, more straightforward. The data is specified, and then the variable's x and y mapping are specified in the aesthetics of the `ggplot` function call. The more specific aesthetics for the points are given inside an `aes` call from within the `geom_point` layer, such as the size and the fill colour. We specify the range of sizes of the points using the `scale_size_continuous` call, and set it to not show the legend for the point size. To make the aspect ratio similar to the `lattice` plot, we add a call to the `coord_fixed` function, and set the ratio appropriately. We again use the scales package to make our x-axis labels look nicer in the log scale.


```r
# Now in ggplot
ggplot(gDatSub, aes(x=gdpPercap, y=lifeExp)) + 
  geom_point(aes(size=sqrt(pop/pi), fill=continent), 
             colour="black", pch=21) +
  scale_x_log10("log(gdpPercap)",
                breaks = trans_breaks("log10", function(x) 10^x),
                labels = trans_format("log10", math_format(10^.x))) + 
  scale_size_continuous(range=c(1,40), guide="none") +
  facet_wrap(~year, nrow=2) +
  ylim(28,85) + coord_fixed(ratio=0.03)
```

![plot of chunk bubblePlotGG](figure/bubblePlotGG.png) 


## Conclusions

I found that making plots using the `ggplot2` package was more straightforward than using `lattice` to produce similar plots. For both packages, I used google to search how to make my plots look a certain way, but I found the instructions for `ggplot2` to be more understandable and intuitive. While I do not claim that `ggplot2` is superior to `lattice`, I find that the coding structure of the `ggplot2` functions was easier to understand.

Looking at the code for `lattice`, sometimes it is very easy to make specialized plots (for example, adding a regression line is as easy as adding a letter to the `type` argument in `xyplot`), but at other times, it is very hard to achieve some effect (such as adding colours for each bubble in the bubble plot - this required creating a custom panel function to add the colours!).

I will continue to learn both packages, as I am still in the learning phases. I did notice some performance advantages when making plots with `lattice` (in some other plots not in this report), and I have been told that the `ggplot2` functions can be some level of magnitude slower than `lattice`. This may prove to be useful when plotting figures for very large data sets

> For the code used to generate this report, [click here](https://gist.github.com/jonnybaik/6863975#file-stat545a-2013-hw05_baik-jon-rmd)