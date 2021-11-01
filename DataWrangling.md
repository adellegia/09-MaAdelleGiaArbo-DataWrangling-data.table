---
title: "Data Wrangling at Scale with data.table"
output:
  html_document:
    toc: TRUE
    keep_md: TRUE
    df_print: paged
    number_sections: FALSE
    highlight: tango
    theme: lumen
    toc_depth: 3
    toc_float: true
    css: custom.css 
    self_contained: false
    includes:
      after_body: footer.html
---


This presentation will detail data.tables data wrangling functionality, including:

- Manipulating data
- Modifying variables
- Summarizing data
- Chaining
- Joining
- Plotting data

We will end the presentation with an exercise. 


# What's data wrangling? 🤔
Data wrangling is the process of converting raw data to another format which can be readily analyzable. 

<img src="pics/data_wrangling.png" width="40%" style="display: block; margin: auto;" />

# data.table 📹
`data.table` provides an efficient and high performance alternative of base R's data.frame when conducting data wrangling. 

`data.table` enables this efficiency by providing: 

- concise syntax: fast to type, fast to read
- fast speed
- memory efficiency
- a large community
- rich features

To install the package, we use `install.packages('data.table')`.

## Importing with `fread()`

`data.table`'s efficiency begins from the outset with `fread()`, which is short for fast read is data.table's version of `read_csv()`.

Let's import and read the `mtcars` dataset and call it `mt` using `fread()`.

```r
library(data.table)
mt <- fread("mtcars.csv")
```

Let's check how fast `fread()` actually is it compared to `read.csv`. You can also write a file using `fwrite()` in `data.table` like `write.csv`. 

```r
# Create a large .csv file
set.seed(28)
trial <- data.frame(matrix(runif(10000000), nrow=1000000))
#write.csv(trial, 'trial.csv', row.names = F)
```

We can then see that `fread()` is at least 20 times faster! Let's check it out!

```r
# Time taken by read.csv to import
system.time({trial_df <- read.csv('trial.csv')})
```

```
##    user  system elapsed 
##   38.57    0.84   39.61
```

```r
# Time taken by fread to import
system.time({trial_df <- fread('trial.csv')})
```

```
##    user  system elapsed 
##    0.47    0.10    0.09
```

## Creating data tables
To highlight what a data table is, we will create data tables using different functions and compare the results with a data frame. Let's use a built-in R data called `airquality`.

```r
head(airquality)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["Ozone"],"name":[1],"type":["int"],"align":["right"]},{"label":["Solar.R"],"name":[2],"type":["int"],"align":["right"]},{"label":["Wind"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["Temp"],"name":[4],"type":["int"],"align":["right"]},{"label":["Month"],"name":[5],"type":["int"],"align":["right"]},{"label":["Day"],"name":[6],"type":["int"],"align":["right"]}],"data":[{"1":"41","2":"190","3":"7.4","4":"67","5":"5","6":"1","_rn_":"1"},{"1":"36","2":"118","3":"8.0","4":"72","5":"5","6":"2","_rn_":"2"},{"1":"12","2":"149","3":"12.6","4":"74","5":"5","6":"3","_rn_":"3"},{"1":"18","2":"313","3":"11.5","4":"62","5":"5","6":"4","_rn_":"4"},{"1":"NA","2":"NA","3":"14.3","4":"56","5":"5","6":"5","_rn_":"5"},{"1":"28","2":"NA","3":"14.9","4":"66","5":"5","6":"6","_rn_":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
class(airquality)
```

```
## [1] "data.frame"
```
To convert this data frame to a data table, we can either use:

* `data.table()` and `as.data.table()` - This function creates a copy of the data and converts it to a data.table

* `setDT()` - This function converts the data to a data.table, there is then no need to assign to a new object

We convert `airquality` data frame to a data table using `as.data.table`. It then becomes both a data table and a data frame.

```r
class(airquality)
```

```
## [1] "data.frame"
```

```r
airqualityDT <- as.data.table(airquality)
class(airqualityDT)
```

```
## [1] "data.table" "data.frame"
```

Now, we do the same for the `mt` data frame.

```r
mt <- as.data.frame(fread("mtcars.csv"))

class(mt)
```

```
## [1] "data.frame"
```

```r
setDT(mt) # we do not need to assign mt to a new object
class(mt)
```

```
## [1] "data.table" "data.frame"
```

```r
# for illustration purposes, let's use `as.data.table` and assign it to mtDT
mtDT <- as.data.table(mt)
class(mtDT)
```

```
## [1] "data.table" "data.frame"
```

We can "manually" create a data table using `data.table()`.

```r
DT <- data.table(x = 1:8,
                 y = round(pi*1:8,2),
                 z = letters[1:8])
knitr::kable(DT)
```



|  x|     y|z  |
|--:|-----:|:--|
|  1|  3.14|a  |
|  2|  6.28|b  |
|  3|  9.42|c  |
|  4| 12.57|d  |
|  5| 15.71|e  |
|  6| 18.85|f  |
|  7| 21.99|g  |
|  8| 25.13|h  |

Packages and functions that work with data frames also work for data tables. Since a data.table is a data.frame, it is compatible with R functions and packages that accept only data.frames.

```r
names(mtDT)
```

```
##  [1] "carname" "mpg"     "cyl"     "disp"    "hp"      "drat"    "wt"     
##  [8] "qsec"    "vs"      "am"      "gear"    "carb"
```

```r
dim(mtDT)
```

```
## [1] 32 12
```

```r
str(mtDT)
```

```
## Classes 'data.table' and 'data.frame':	32 obs. of  12 variables:
##  $ carname: chr  "Mazda RX4" "Mazda RX4 Wag" "Datsun 710" "Hornet 4 Drive" ...
##  $ mpg    : num  4.58 4.58 4.77 4.63 4.32 ...
##  $ cyl    : int  6 6 4 6 8 6 8 4 4 6 ...
##  $ disp   : num  160 160 108 258 360 ...
##  $ hp     : int  110 110 93 110 175 105 245 62 95 123 ...
##  $ drat   : num  3.9 3.9 3.85 3.08 3.15 2.76 3.21 3.69 3.92 3.92 ...
##  $ wt     : num  2.62 2.88 2.32 3.21 3.44 ...
##  $ qsec   : num  16.5 17 18.6 19.4 17 ...
##  $ vs     : int  0 0 1 1 0 1 0 1 1 1 ...
##  $ am     : int  1 1 1 0 0 0 0 0 0 0 ...
##  $ gear   : int  4 4 4 3 3 3 3 4 4 4 ...
##  $ carb   : int  4 4 1 1 2 1 4 2 2 4 ...
##  - attr(*, ".internal.selfref")=<externalptr>
```

For reference, the `mt` data has the following variables:

- [, 1]	**carname**   - Car name
- [, 2]	**mpg**   - Miles/(US) gallon
- [, 3]	**cyl**   - Number of cylinders
- [, 4]	**disp**  - Displacement (cu.in.)
- [, 5]	**hp**    - Gross horsepower
- [, 6]	**drat**  - Rear axle ratio
- [, 7]	**wt**    - Weight (1000 lbs)
- [, 8]	**qsec**  - 1/4 mile time
- [, 9]	**vs**    - Engine (0 = V-shaped, 1 = straight)
- [,10]	**am**    - Transmission (0 = automatic, 1 = manual)
- [,11]	**gear**  - Number of forward gears
- [,12]	**carb**  - Number of carburetors


# Data manipulation 🚗
When compared to a data frame, the basic arguments within brackets are **NOT** row and column numbers but rather "i", "j" and "by". 

For example, a data table named DT, DT[i, j, by] translates to "Take DT, subset rows using **i**, then calculate **j** grouped by **by**".

Use data.table subset [ operator the same way you would use data.frame one, but...

* no need to prefix each column with DT$ (like subset() and with() but built-in)
* any R expression using any package is allowed in j argument, not just list of columns
* extra argument by to compute j expression by group

<img src="pics/data_table_syntax.png" width="50%" style="display: block; margin: auto;" />

Let's compare filtering using conditional statements in a data frame vs. in a data table. 
You will notice one of the primary benefits of data table, you only need to pass the column names!

```r
mt[mt$cyl == 6 & mt$gear == 4, ]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[3],"type":["int"],"align":["right"]},{"label":["disp"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["hp"],"name":[5],"type":["int"],"align":["right"]},{"label":["drat"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["wt"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["qsec"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["vs"],"name":[9],"type":["int"],"align":["right"]},{"label":["am"],"name":[10],"type":["int"],"align":["right"]},{"label":["gear"],"name":[11],"type":["int"],"align":["right"]},{"label":["carb"],"name":[12],"type":["int"],"align":["right"]}],"data":[{"1":"Mazda RX4","2":"4.582576","3":"6","4":"160.0","5":"110","6":"3.90","7":"2.620","8":"16.46","9":"0","10":"1","11":"4","12":"4"},{"1":"Mazda RX4 Wag","2":"4.582576","3":"6","4":"160.0","5":"110","6":"3.90","7":"2.875","8":"17.02","9":"0","10":"1","11":"4","12":"4"},{"1":"Merc 280","2":"4.381780","3":"6","4":"167.6","5":"123","6":"3.92","7":"3.440","8":"18.30","9":"1","10":"0","11":"4","12":"4"},{"1":"Merc 280C","2":"4.219005","3":"6","4":"167.6","5":"123","6":"3.92","7":"3.440","8":"18.90","9":"1","10":"0","11":"4","12":"4"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# datatable syntax
mtDT[cyl==6 & gear==4, ]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[3],"type":["int"],"align":["right"]},{"label":["disp"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["hp"],"name":[5],"type":["int"],"align":["right"]},{"label":["drat"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["wt"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["qsec"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["vs"],"name":[9],"type":["int"],"align":["right"]},{"label":["am"],"name":[10],"type":["int"],"align":["right"]},{"label":["gear"],"name":[11],"type":["int"],"align":["right"]},{"label":["carb"],"name":[12],"type":["int"],"align":["right"]}],"data":[{"1":"Mazda RX4","2":"4.582576","3":"6","4":"160.0","5":"110","6":"3.90","7":"2.620","8":"16.46","9":"0","10":"1","11":"4","12":"4"},{"1":"Mazda RX4 Wag","2":"4.582576","3":"6","4":"160.0","5":"110","6":"3.90","7":"2.875","8":"17.02","9":"0","10":"1","11":"4","12":"4"},{"1":"Merc 280","2":"4.381780","3":"6","4":"167.6","5":"123","6":"3.92","7":"3.440","8":"18.30","9":"1","10":"0","11":"4","12":"4"},{"1":"Merc 280C","2":"4.219005","3":"6","4":"167.6","5":"123","6":"3.92","7":"3.440","8":"18.90","9":"1","10":"0","11":"4","12":"4"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

## Filtering rows
Selecting rows is largely similar to data frame.


```r
# select a row
mtDT[1,]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[3],"type":["int"],"align":["right"]},{"label":["disp"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["hp"],"name":[5],"type":["int"],"align":["right"]},{"label":["drat"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["wt"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["qsec"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["vs"],"name":[9],"type":["int"],"align":["right"]},{"label":["am"],"name":[10],"type":["int"],"align":["right"]},{"label":["gear"],"name":[11],"type":["int"],"align":["right"]},{"label":["carb"],"name":[12],"type":["int"],"align":["right"]}],"data":[{"1":"Mazda RX4","2":"4.582576","3":"6","4":"160","5":"110","6":"3.9","7":"2.62","8":"16.46","9":"0","10":"1","11":"4","12":"4"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# selecting first five rows
mtDT[1:5,]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[3],"type":["int"],"align":["right"]},{"label":["disp"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["hp"],"name":[5],"type":["int"],"align":["right"]},{"label":["drat"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["wt"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["qsec"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["vs"],"name":[9],"type":["int"],"align":["right"]},{"label":["am"],"name":[10],"type":["int"],"align":["right"]},{"label":["gear"],"name":[11],"type":["int"],"align":["right"]},{"label":["carb"],"name":[12],"type":["int"],"align":["right"]}],"data":[{"1":"Mazda RX4","2":"4.582576","3":"6","4":"160","5":"110","6":"3.90","7":"2.620","8":"16.46","9":"0","10":"1","11":"4","12":"4"},{"1":"Mazda RX4 Wag","2":"4.582576","3":"6","4":"160","5":"110","6":"3.90","7":"2.875","8":"17.02","9":"0","10":"1","11":"4","12":"4"},{"1":"Datsun 710","2":"4.774935","3":"4","4":"108","5":"93","6":"3.85","7":"2.320","8":"18.61","9":"1","10":"1","11":"4","12":"1"},{"1":"Hornet 4 Drive","2":"4.626013","3":"6","4":"258","5":"110","6":"3.08","7":"3.215","8":"19.44","9":"1","10":"0","11":"3","12":"1"},{"1":"Hornet Sportabout","2":"4.324350","3":"8","4":"360","5":"175","6":"3.15","7":"3.440","8":"17.02","9":"0","10":"0","11":"3","12":"2"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# no need to put comma in selecting rows
mtDT[1:2]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[3],"type":["int"],"align":["right"]},{"label":["disp"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["hp"],"name":[5],"type":["int"],"align":["right"]},{"label":["drat"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["wt"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["qsec"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["vs"],"name":[9],"type":["int"],"align":["right"]},{"label":["am"],"name":[10],"type":["int"],"align":["right"]},{"label":["gear"],"name":[11],"type":["int"],"align":["right"]},{"label":["carb"],"name":[12],"type":["int"],"align":["right"]}],"data":[{"1":"Mazda RX4","2":"4.582576","3":"6","4":"160","5":"110","6":"3.9","7":"2.620","8":"16.46","9":"0","10":"1","11":"4","12":"4"},{"1":"Mazda RX4 Wag","2":"4.582576","3":"6","4":"160","5":"110","6":"3.9","7":"2.875","8":"17.02","9":"0","10":"1","11":"4","12":"4"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# subsetting using conditional statements
mtDT[cyl < 5 & am == 0]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[3],"type":["int"],"align":["right"]},{"label":["disp"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["hp"],"name":[5],"type":["int"],"align":["right"]},{"label":["drat"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["wt"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["qsec"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["vs"],"name":[9],"type":["int"],"align":["right"]},{"label":["am"],"name":[10],"type":["int"],"align":["right"]},{"label":["gear"],"name":[11],"type":["int"],"align":["right"]},{"label":["carb"],"name":[12],"type":["int"],"align":["right"]}],"data":[{"1":"Merc 240D","2":"4.939636","3":"4","4":"146.7","5":"62","6":"3.69","7":"3.190","8":"20.00","9":"1","10":"0","11":"4","12":"2"},{"1":"Merc 230","2":"4.774935","3":"4","4":"140.8","5":"95","6":"3.92","7":"3.150","8":"22.90","9":"1","10":"0","11":"4","12":"2"},{"1":"Toyota Corona","2":"4.636809","3":"4","4":"120.1","5":"97","6":"3.70","7":"2.465","8":"20.01","9":"1","10":"0","11":"3","12":"1"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
mtDT[cyl %between% c(5,8)] # conditions a range of values
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[3],"type":["int"],"align":["right"]},{"label":["disp"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["hp"],"name":[5],"type":["int"],"align":["right"]},{"label":["drat"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["wt"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["qsec"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["vs"],"name":[9],"type":["int"],"align":["right"]},{"label":["am"],"name":[10],"type":["int"],"align":["right"]},{"label":["gear"],"name":[11],"type":["int"],"align":["right"]},{"label":["carb"],"name":[12],"type":["int"],"align":["right"]}],"data":[{"1":"Mazda RX4","2":"4.582576","3":"6","4":"160.0","5":"110","6":"3.90","7":"2.620","8":"16.46","9":"0","10":"1","11":"4","12":"4"},{"1":"Mazda RX4 Wag","2":"4.582576","3":"6","4":"160.0","5":"110","6":"3.90","7":"2.875","8":"17.02","9":"0","10":"1","11":"4","12":"4"},{"1":"Hornet 4 Drive","2":"4.626013","3":"6","4":"258.0","5":"110","6":"3.08","7":"3.215","8":"19.44","9":"1","10":"0","11":"3","12":"1"},{"1":"Hornet Sportabout","2":"4.324350","3":"8","4":"360.0","5":"175","6":"3.15","7":"3.440","8":"17.02","9":"0","10":"0","11":"3","12":"2"},{"1":"Valiant","2":"4.254409","3":"6","4":"225.0","5":"105","6":"2.76","7":"3.460","8":"20.22","9":"1","10":"0","11":"3","12":"1"},{"1":"Duster 360","2":"3.781534","3":"8","4":"360.0","5":"245","6":"3.21","7":"3.570","8":"15.84","9":"0","10":"0","11":"3","12":"4"},{"1":"Merc 280","2":"4.381780","3":"6","4":"167.6","5":"123","6":"3.92","7":"3.440","8":"18.30","9":"1","10":"0","11":"4","12":"4"},{"1":"Merc 280C","2":"4.219005","3":"6","4":"167.6","5":"123","6":"3.92","7":"3.440","8":"18.90","9":"1","10":"0","11":"4","12":"4"},{"1":"Merc 450SE","2":"4.049691","3":"8","4":"275.8","5":"180","6":"3.07","7":"4.070","8":"17.40","9":"0","10":"0","11":"3","12":"3"},{"1":"Merc 450SL","2":"4.159327","3":"8","4":"275.8","5":"180","6":"3.07","7":"3.730","8":"17.60","9":"0","10":"0","11":"3","12":"3"},{"1":"Merc 450SLC","2":"3.898718","3":"8","4":"275.8","5":"180","6":"3.07","7":"3.780","8":"18.00","9":"0","10":"0","11":"3","12":"3"},{"1":"Cadillac Fleetwood","2":"3.224903","3":"8","4":"472.0","5":"205","6":"2.93","7":"5.250","8":"17.98","9":"0","10":"0","11":"3","12":"4"},{"1":"Lincoln Continental","2":"3.224903","3":"8","4":"460.0","5":"215","6":"3.00","7":"5.424","8":"17.82","9":"0","10":"0","11":"3","12":"4"},{"1":"Chrysler Imperial","2":"3.834058","3":"8","4":"440.0","5":"230","6":"3.23","7":"5.345","8":"17.42","9":"0","10":"0","11":"3","12":"4"},{"1":"Dodge Challenger","2":"3.937004","3":"8","4":"318.0","5":"150","6":"2.76","7":"3.520","8":"16.87","9":"0","10":"0","11":"3","12":"2"},{"1":"AMC Javelin","2":"3.898718","3":"8","4":"304.0","5":"150","6":"3.15","7":"3.435","8":"17.30","9":"0","10":"0","11":"3","12":"2"},{"1":"Camaro Z28","2":"3.646917","3":"8","4":"350.0","5":"245","6":"3.73","7":"3.840","8":"15.41","9":"0","10":"0","11":"3","12":"4"},{"1":"Pontiac Firebird","2":"4.381780","3":"8","4":"400.0","5":"175","6":"3.08","7":"3.845","8":"17.05","9":"0","10":"0","11":"3","12":"2"},{"1":"Ford Pantera L","2":"3.974921","3":"8","4":"351.0","5":"264","6":"4.22","7":"3.170","8":"14.50","9":"0","10":"1","11":"5","12":"4"},{"1":"Ferrari Dino","2":"4.438468","3":"6","4":"145.0","5":"175","6":"3.62","7":"2.770","8":"15.50","9":"0","10":"1","11":"5","12":"6"},{"1":"Maserati Bora","2":"3.872983","3":"8","4":"301.0","5":"335","6":"3.54","7":"3.570","8":"14.60","9":"0","10":"1","11":"5","12":"8"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
mtDT[carname %like% "Mazda"] # finds a pattern
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[3],"type":["int"],"align":["right"]},{"label":["disp"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["hp"],"name":[5],"type":["int"],"align":["right"]},{"label":["drat"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["wt"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["qsec"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["vs"],"name":[9],"type":["int"],"align":["right"]},{"label":["am"],"name":[10],"type":["int"],"align":["right"]},{"label":["gear"],"name":[11],"type":["int"],"align":["right"]},{"label":["carb"],"name":[12],"type":["int"],"align":["right"]}],"data":[{"1":"Mazda RX4","2":"4.582576","3":"6","4":"160","5":"110","6":"3.9","7":"2.620","8":"16.46","9":"0","10":"1","11":"4","12":"4"},{"1":"Mazda RX4 Wag","2":"4.582576","3":"6","4":"160","5":"110","6":"3.9","7":"2.875","8":"17.02","9":"0","10":"1","11":"4","12":"4"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

## Selecting columns
There are some pointers you have to remember to select  a column.

```r
# using index
mtDT[,1:2]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["mpg"],"name":[2],"type":["dbl"],"align":["right"]}],"data":[{"1":"Mazda RX4","2":"4.582576"},{"1":"Mazda RX4 Wag","2":"4.582576"},{"1":"Datsun 710","2":"4.774935"},{"1":"Hornet 4 Drive","2":"4.626013"},{"1":"Hornet Sportabout","2":"4.324350"},{"1":"Valiant","2":"4.254409"},{"1":"Duster 360","2":"3.781534"},{"1":"Merc 240D","2":"4.939636"},{"1":"Merc 230","2":"4.774935"},{"1":"Merc 280","2":"4.381780"},{"1":"Merc 280C","2":"4.219005"},{"1":"Merc 450SE","2":"4.049691"},{"1":"Merc 450SL","2":"4.159327"},{"1":"Merc 450SLC","2":"3.898718"},{"1":"Cadillac Fleetwood","2":"3.224903"},{"1":"Lincoln Continental","2":"3.224903"},{"1":"Chrysler Imperial","2":"3.834058"},{"1":"Fiat 128","2":"5.692100"},{"1":"Honda Civic","2":"5.513620"},{"1":"Toyota Corolla","2":"5.822371"},{"1":"Toyota Corona","2":"4.636809"},{"1":"Dodge Challenger","2":"3.937004"},{"1":"AMC Javelin","2":"3.898718"},{"1":"Camaro Z28","2":"3.646917"},{"1":"Pontiac Firebird","2":"4.381780"},{"1":"Fiat X1-9","2":"5.224940"},{"1":"Porsche 914-2","2":"5.099020"},{"1":"Lotus Europa","2":"5.513620"},{"1":"Ford Pantera L","2":"3.974921"},{"1":"Ferrari Dino","2":"4.438468"},{"1":"Maserati Bora","2":"3.872983"},{"1":"Volvo 142E","2":"4.626013"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# using column name
mtDT[ , mpg] # returns a vector
```

```
##  [1] 4.582576 4.582576 4.774935 4.626013 4.324350 4.254409 3.781534 4.939636
##  [9] 4.774935 4.381780 4.219005 4.049691 4.159327 3.898718 3.224903 3.224903
## [17] 3.834058 5.692100 5.513620 5.822371 4.636809 3.937004 3.898718 3.646917
## [25] 4.381780 5.224940 5.099020 5.513620 3.974921 4.438468 3.872983 4.626013
```

```r
mtDT[ , "mpg"] # returns the column
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["mpg"],"name":[1],"type":["dbl"],"align":["right"]}],"data":[{"1":"4.582576"},{"1":"4.582576"},{"1":"4.774935"},{"1":"4.626013"},{"1":"4.324350"},{"1":"4.254409"},{"1":"3.781534"},{"1":"4.939636"},{"1":"4.774935"},{"1":"4.381780"},{"1":"4.219005"},{"1":"4.049691"},{"1":"4.159327"},{"1":"3.898718"},{"1":"3.224903"},{"1":"3.224903"},{"1":"3.834058"},{"1":"5.692100"},{"1":"5.513620"},{"1":"5.822371"},{"1":"4.636809"},{"1":"3.937004"},{"1":"3.898718"},{"1":"3.646917"},{"1":"4.381780"},{"1":"5.224940"},{"1":"5.099020"},{"1":"5.513620"},{"1":"3.974921"},{"1":"4.438468"},{"1":"3.872983"},{"1":"4.626013"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# selecting multiple columns using "list" or puting inside ".()"
mtDT[, list(mpg, cyl)]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["mpg"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[2],"type":["int"],"align":["right"]}],"data":[{"1":"4.582576","2":"6"},{"1":"4.582576","2":"6"},{"1":"4.774935","2":"4"},{"1":"4.626013","2":"6"},{"1":"4.324350","2":"8"},{"1":"4.254409","2":"6"},{"1":"3.781534","2":"8"},{"1":"4.939636","2":"4"},{"1":"4.774935","2":"4"},{"1":"4.381780","2":"6"},{"1":"4.219005","2":"6"},{"1":"4.049691","2":"8"},{"1":"4.159327","2":"8"},{"1":"3.898718","2":"8"},{"1":"3.224903","2":"8"},{"1":"3.224903","2":"8"},{"1":"3.834058","2":"8"},{"1":"5.692100","2":"4"},{"1":"5.513620","2":"4"},{"1":"5.822371","2":"4"},{"1":"4.636809","2":"4"},{"1":"3.937004","2":"8"},{"1":"3.898718","2":"8"},{"1":"3.646917","2":"8"},{"1":"4.381780","2":"8"},{"1":"5.224940","2":"4"},{"1":"5.099020","2":"4"},{"1":"5.513620","2":"4"},{"1":"3.974921","2":"8"},{"1":"4.438468","2":"6"},{"1":"3.872983","2":"8"},{"1":"4.626013","2":"4"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
mtDT[, .(mpg, cyl)]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["mpg"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[2],"type":["int"],"align":["right"]}],"data":[{"1":"4.582576","2":"6"},{"1":"4.582576","2":"6"},{"1":"4.774935","2":"4"},{"1":"4.626013","2":"6"},{"1":"4.324350","2":"8"},{"1":"4.254409","2":"6"},{"1":"3.781534","2":"8"},{"1":"4.939636","2":"4"},{"1":"4.774935","2":"4"},{"1":"4.381780","2":"6"},{"1":"4.219005","2":"6"},{"1":"4.049691","2":"8"},{"1":"4.159327","2":"8"},{"1":"3.898718","2":"8"},{"1":"3.224903","2":"8"},{"1":"3.224903","2":"8"},{"1":"3.834058","2":"8"},{"1":"5.692100","2":"4"},{"1":"5.513620","2":"4"},{"1":"5.822371","2":"4"},{"1":"4.636809","2":"4"},{"1":"3.937004","2":"8"},{"1":"3.898718","2":"8"},{"1":"3.646917","2":"8"},{"1":"4.381780","2":"8"},{"1":"5.224940","2":"4"},{"1":"5.099020","2":"4"},{"1":"5.513620","2":"4"},{"1":"3.974921","2":"8"},{"1":"4.438468","2":"6"},{"1":"3.872983","2":"8"},{"1":"4.626013","2":"4"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# selecting multiple columns using a character vector
col <- c('mpg', 'cyl', 'disp')

#mtDT[, col] # returns an error

# need to put, with = FALSE, or add ".." before the character vector
mtDT[, col, with = FALSE]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["mpg"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[2],"type":["int"],"align":["right"]},{"label":["disp"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"4.582576","2":"6","3":"160.0"},{"1":"4.582576","2":"6","3":"160.0"},{"1":"4.774935","2":"4","3":"108.0"},{"1":"4.626013","2":"6","3":"258.0"},{"1":"4.324350","2":"8","3":"360.0"},{"1":"4.254409","2":"6","3":"225.0"},{"1":"3.781534","2":"8","3":"360.0"},{"1":"4.939636","2":"4","3":"146.7"},{"1":"4.774935","2":"4","3":"140.8"},{"1":"4.381780","2":"6","3":"167.6"},{"1":"4.219005","2":"6","3":"167.6"},{"1":"4.049691","2":"8","3":"275.8"},{"1":"4.159327","2":"8","3":"275.8"},{"1":"3.898718","2":"8","3":"275.8"},{"1":"3.224903","2":"8","3":"472.0"},{"1":"3.224903","2":"8","3":"460.0"},{"1":"3.834058","2":"8","3":"440.0"},{"1":"5.692100","2":"4","3":"78.7"},{"1":"5.513620","2":"4","3":"75.7"},{"1":"5.822371","2":"4","3":"71.1"},{"1":"4.636809","2":"4","3":"120.1"},{"1":"3.937004","2":"8","3":"318.0"},{"1":"3.898718","2":"8","3":"304.0"},{"1":"3.646917","2":"8","3":"350.0"},{"1":"4.381780","2":"8","3":"400.0"},{"1":"5.224940","2":"4","3":"79.0"},{"1":"5.099020","2":"4","3":"120.3"},{"1":"5.513620","2":"4","3":"95.1"},{"1":"3.974921","2":"8","3":"351.0"},{"1":"4.438468","2":"6","3":"145.0"},{"1":"3.872983","2":"8","3":"301.0"},{"1":"4.626013","2":"4","3":"121.0"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
mtDT[, ..col]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["mpg"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[2],"type":["int"],"align":["right"]},{"label":["disp"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"4.582576","2":"6","3":"160.0"},{"1":"4.582576","2":"6","3":"160.0"},{"1":"4.774935","2":"4","3":"108.0"},{"1":"4.626013","2":"6","3":"258.0"},{"1":"4.324350","2":"8","3":"360.0"},{"1":"4.254409","2":"6","3":"225.0"},{"1":"3.781534","2":"8","3":"360.0"},{"1":"4.939636","2":"4","3":"146.7"},{"1":"4.774935","2":"4","3":"140.8"},{"1":"4.381780","2":"6","3":"167.6"},{"1":"4.219005","2":"6","3":"167.6"},{"1":"4.049691","2":"8","3":"275.8"},{"1":"4.159327","2":"8","3":"275.8"},{"1":"3.898718","2":"8","3":"275.8"},{"1":"3.224903","2":"8","3":"472.0"},{"1":"3.224903","2":"8","3":"460.0"},{"1":"3.834058","2":"8","3":"440.0"},{"1":"5.692100","2":"4","3":"78.7"},{"1":"5.513620","2":"4","3":"75.7"},{"1":"5.822371","2":"4","3":"71.1"},{"1":"4.636809","2":"4","3":"120.1"},{"1":"3.937004","2":"8","3":"318.0"},{"1":"3.898718","2":"8","3":"304.0"},{"1":"3.646917","2":"8","3":"350.0"},{"1":"4.381780","2":"8","3":"400.0"},{"1":"5.224940","2":"4","3":"79.0"},{"1":"5.099020","2":"4","3":"120.3"},{"1":"5.513620","2":"4","3":"95.1"},{"1":"3.974921","2":"8","3":"351.0"},{"1":"4.438468","2":"6","3":"145.0"},{"1":"3.872983","2":"8","3":"301.0"},{"1":"4.626013","2":"4","3":"121.0"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

## Subsetting rows and columns
Combing what we learned above, we can filter rows and select columns together from a data table.

```r
# selecting first row, second column
mtDT[1,2] # returns the column
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["mpg"],"name":[1],"type":["dbl"],"align":["right"]}],"data":[{"1":"4.582576"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
mtDT[1,cyl] # returns a vector
```

```
## [1] 6
```

```r
mtDT[1,"cyl"] # returns the column
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["cyl"],"name":[1],"type":["int"],"align":["right"]}],"data":[{"1":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
mtDT[1,list(mpg, cyl)] # returns the column
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["mpg"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[2],"type":["int"],"align":["right"]}],"data":[{"1":"4.582576","2":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
mtDT[1, .(mpg, cyl)] # returns the column
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["mpg"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[2],"type":["int"],"align":["right"]}],"data":[{"1":"4.582576","2":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
mtDT[1, c(mpg, cyl)] # returns a vector
```

```
## [1] 4.582576 6.000000
```

# Modifying variables 🔑

Data.table also makes it easy to:

- Drop columns; 
- Rename columns; and 
- Assign and save new variables. 

## Dropping columns

```r
col <- c('mpg', 'cyl', 'disp')

mtDT[, !col, with = FALSE]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["hp"],"name":[2],"type":["int"],"align":["right"]},{"label":["drat"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["wt"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["qsec"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["vs"],"name":[6],"type":["int"],"align":["right"]},{"label":["am"],"name":[7],"type":["int"],"align":["right"]},{"label":["gear"],"name":[8],"type":["int"],"align":["right"]},{"label":["carb"],"name":[9],"type":["int"],"align":["right"]}],"data":[{"1":"Mazda RX4","2":"110","3":"3.90","4":"2.620","5":"16.46","6":"0","7":"1","8":"4","9":"4"},{"1":"Mazda RX4 Wag","2":"110","3":"3.90","4":"2.875","5":"17.02","6":"0","7":"1","8":"4","9":"4"},{"1":"Datsun 710","2":"93","3":"3.85","4":"2.320","5":"18.61","6":"1","7":"1","8":"4","9":"1"},{"1":"Hornet 4 Drive","2":"110","3":"3.08","4":"3.215","5":"19.44","6":"1","7":"0","8":"3","9":"1"},{"1":"Hornet Sportabout","2":"175","3":"3.15","4":"3.440","5":"17.02","6":"0","7":"0","8":"3","9":"2"},{"1":"Valiant","2":"105","3":"2.76","4":"3.460","5":"20.22","6":"1","7":"0","8":"3","9":"1"},{"1":"Duster 360","2":"245","3":"3.21","4":"3.570","5":"15.84","6":"0","7":"0","8":"3","9":"4"},{"1":"Merc 240D","2":"62","3":"3.69","4":"3.190","5":"20.00","6":"1","7":"0","8":"4","9":"2"},{"1":"Merc 230","2":"95","3":"3.92","4":"3.150","5":"22.90","6":"1","7":"0","8":"4","9":"2"},{"1":"Merc 280","2":"123","3":"3.92","4":"3.440","5":"18.30","6":"1","7":"0","8":"4","9":"4"},{"1":"Merc 280C","2":"123","3":"3.92","4":"3.440","5":"18.90","6":"1","7":"0","8":"4","9":"4"},{"1":"Merc 450SE","2":"180","3":"3.07","4":"4.070","5":"17.40","6":"0","7":"0","8":"3","9":"3"},{"1":"Merc 450SL","2":"180","3":"3.07","4":"3.730","5":"17.60","6":"0","7":"0","8":"3","9":"3"},{"1":"Merc 450SLC","2":"180","3":"3.07","4":"3.780","5":"18.00","6":"0","7":"0","8":"3","9":"3"},{"1":"Cadillac Fleetwood","2":"205","3":"2.93","4":"5.250","5":"17.98","6":"0","7":"0","8":"3","9":"4"},{"1":"Lincoln Continental","2":"215","3":"3.00","4":"5.424","5":"17.82","6":"0","7":"0","8":"3","9":"4"},{"1":"Chrysler Imperial","2":"230","3":"3.23","4":"5.345","5":"17.42","6":"0","7":"0","8":"3","9":"4"},{"1":"Fiat 128","2":"66","3":"4.08","4":"2.200","5":"19.47","6":"1","7":"1","8":"4","9":"1"},{"1":"Honda Civic","2":"52","3":"4.93","4":"1.615","5":"18.52","6":"1","7":"1","8":"4","9":"2"},{"1":"Toyota Corolla","2":"65","3":"4.22","4":"1.835","5":"19.90","6":"1","7":"1","8":"4","9":"1"},{"1":"Toyota Corona","2":"97","3":"3.70","4":"2.465","5":"20.01","6":"1","7":"0","8":"3","9":"1"},{"1":"Dodge Challenger","2":"150","3":"2.76","4":"3.520","5":"16.87","6":"0","7":"0","8":"3","9":"2"},{"1":"AMC Javelin","2":"150","3":"3.15","4":"3.435","5":"17.30","6":"0","7":"0","8":"3","9":"2"},{"1":"Camaro Z28","2":"245","3":"3.73","4":"3.840","5":"15.41","6":"0","7":"0","8":"3","9":"4"},{"1":"Pontiac Firebird","2":"175","3":"3.08","4":"3.845","5":"17.05","6":"0","7":"0","8":"3","9":"2"},{"1":"Fiat X1-9","2":"66","3":"4.08","4":"1.935","5":"18.90","6":"1","7":"1","8":"4","9":"1"},{"1":"Porsche 914-2","2":"91","3":"4.43","4":"2.140","5":"16.70","6":"0","7":"1","8":"5","9":"2"},{"1":"Lotus Europa","2":"113","3":"3.77","4":"1.513","5":"16.90","6":"1","7":"1","8":"5","9":"2"},{"1":"Ford Pantera L","2":"264","3":"4.22","4":"3.170","5":"14.50","6":"0","7":"1","8":"5","9":"4"},{"1":"Ferrari Dino","2":"175","3":"3.62","4":"2.770","5":"15.50","6":"0","7":"1","8":"5","9":"6"},{"1":"Maserati Bora","2":"335","3":"3.54","4":"3.570","5":"14.60","6":"0","7":"1","8":"5","9":"8"},{"1":"Volvo 142E","2":"109","3":"4.11","4":"2.780","5":"18.60","6":"1","7":"1","8":"4","9":"2"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
mtDT[, !..col] 
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["hp"],"name":[2],"type":["int"],"align":["right"]},{"label":["drat"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["wt"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["qsec"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["vs"],"name":[6],"type":["int"],"align":["right"]},{"label":["am"],"name":[7],"type":["int"],"align":["right"]},{"label":["gear"],"name":[8],"type":["int"],"align":["right"]},{"label":["carb"],"name":[9],"type":["int"],"align":["right"]}],"data":[{"1":"Mazda RX4","2":"110","3":"3.90","4":"2.620","5":"16.46","6":"0","7":"1","8":"4","9":"4"},{"1":"Mazda RX4 Wag","2":"110","3":"3.90","4":"2.875","5":"17.02","6":"0","7":"1","8":"4","9":"4"},{"1":"Datsun 710","2":"93","3":"3.85","4":"2.320","5":"18.61","6":"1","7":"1","8":"4","9":"1"},{"1":"Hornet 4 Drive","2":"110","3":"3.08","4":"3.215","5":"19.44","6":"1","7":"0","8":"3","9":"1"},{"1":"Hornet Sportabout","2":"175","3":"3.15","4":"3.440","5":"17.02","6":"0","7":"0","8":"3","9":"2"},{"1":"Valiant","2":"105","3":"2.76","4":"3.460","5":"20.22","6":"1","7":"0","8":"3","9":"1"},{"1":"Duster 360","2":"245","3":"3.21","4":"3.570","5":"15.84","6":"0","7":"0","8":"3","9":"4"},{"1":"Merc 240D","2":"62","3":"3.69","4":"3.190","5":"20.00","6":"1","7":"0","8":"4","9":"2"},{"1":"Merc 230","2":"95","3":"3.92","4":"3.150","5":"22.90","6":"1","7":"0","8":"4","9":"2"},{"1":"Merc 280","2":"123","3":"3.92","4":"3.440","5":"18.30","6":"1","7":"0","8":"4","9":"4"},{"1":"Merc 280C","2":"123","3":"3.92","4":"3.440","5":"18.90","6":"1","7":"0","8":"4","9":"4"},{"1":"Merc 450SE","2":"180","3":"3.07","4":"4.070","5":"17.40","6":"0","7":"0","8":"3","9":"3"},{"1":"Merc 450SL","2":"180","3":"3.07","4":"3.730","5":"17.60","6":"0","7":"0","8":"3","9":"3"},{"1":"Merc 450SLC","2":"180","3":"3.07","4":"3.780","5":"18.00","6":"0","7":"0","8":"3","9":"3"},{"1":"Cadillac Fleetwood","2":"205","3":"2.93","4":"5.250","5":"17.98","6":"0","7":"0","8":"3","9":"4"},{"1":"Lincoln Continental","2":"215","3":"3.00","4":"5.424","5":"17.82","6":"0","7":"0","8":"3","9":"4"},{"1":"Chrysler Imperial","2":"230","3":"3.23","4":"5.345","5":"17.42","6":"0","7":"0","8":"3","9":"4"},{"1":"Fiat 128","2":"66","3":"4.08","4":"2.200","5":"19.47","6":"1","7":"1","8":"4","9":"1"},{"1":"Honda Civic","2":"52","3":"4.93","4":"1.615","5":"18.52","6":"1","7":"1","8":"4","9":"2"},{"1":"Toyota Corolla","2":"65","3":"4.22","4":"1.835","5":"19.90","6":"1","7":"1","8":"4","9":"1"},{"1":"Toyota Corona","2":"97","3":"3.70","4":"2.465","5":"20.01","6":"1","7":"0","8":"3","9":"1"},{"1":"Dodge Challenger","2":"150","3":"2.76","4":"3.520","5":"16.87","6":"0","7":"0","8":"3","9":"2"},{"1":"AMC Javelin","2":"150","3":"3.15","4":"3.435","5":"17.30","6":"0","7":"0","8":"3","9":"2"},{"1":"Camaro Z28","2":"245","3":"3.73","4":"3.840","5":"15.41","6":"0","7":"0","8":"3","9":"4"},{"1":"Pontiac Firebird","2":"175","3":"3.08","4":"3.845","5":"17.05","6":"0","7":"0","8":"3","9":"2"},{"1":"Fiat X1-9","2":"66","3":"4.08","4":"1.935","5":"18.90","6":"1","7":"1","8":"4","9":"1"},{"1":"Porsche 914-2","2":"91","3":"4.43","4":"2.140","5":"16.70","6":"0","7":"1","8":"5","9":"2"},{"1":"Lotus Europa","2":"113","3":"3.77","4":"1.513","5":"16.90","6":"1","7":"1","8":"5","9":"2"},{"1":"Ford Pantera L","2":"264","3":"4.22","4":"3.170","5":"14.50","6":"0","7":"1","8":"5","9":"4"},{"1":"Ferrari Dino","2":"175","3":"3.62","4":"2.770","5":"15.50","6":"0","7":"1","8":"5","9":"6"},{"1":"Maserati Bora","2":"335","3":"3.54","4":"3.570","5":"14.60","6":"0","7":"1","8":"5","9":"8"},{"1":"Volvo 142E","2":"109","3":"4.11","4":"2.780","5":"18.60","6":"1","7":"1","8":"4","9":"2"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# note: you need to assign this to a new object to make a new data table excluding the dropped variables
```

## Renaming columns using `setnames()`
`setnames()` in data.table changes the names of a data table by reference.

```r
setnames(mtDT, 'vs', 'engine_type')
names(mtDT) # vs renamed to engine_type
```

```
##  [1] "carname"     "mpg"         "cyl"         "disp"        "hp"         
##  [6] "drat"        "wt"          "qsec"        "engine_type" "am"         
## [11] "gear"        "carb"
```

```r
setnames(mtDT,5:6,c("horse_power","rear_ratio"))
names(mtDT)
```

```
##  [1] "carname"     "mpg"         "cyl"         "disp"        "horse_power"
##  [6] "rear_ratio"  "wt"          "qsec"        "engine_type" "am"         
## [11] "gear"        "carb"
```

```r
setnames(mtDT,5:6,c("hp","drat"))
```

## Assigning and saving new variables
To create a new column, we use this symbol `:=` to assign the new variable.

```r
mtDT[, cyl_gear := cyl + gear]

mtDT[, cyl_gear] # returns a vector
```

```
##  [1] 10 10  8  9 11  9 11  8  8 10 10 11 11 11 11 11 11  8  8  8  7 11 11 11 11
## [26]  8  9  9 13 11 13  8
```

```r
mtDT[, "cyl_gear"] # returns the column
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["cyl_gear"],"name":[1],"type":["int"],"align":["right"]}],"data":[{"1":"10"},{"1":"10"},{"1":"8"},{"1":"9"},{"1":"11"},{"1":"9"},{"1":"11"},{"1":"8"},{"1":"8"},{"1":"10"},{"1":"10"},{"1":"11"},{"1":"11"},{"1":"11"},{"1":"11"},{"1":"11"},{"1":"11"},{"1":"8"},{"1":"8"},{"1":"8"},{"1":"7"},{"1":"11"},{"1":"11"},{"1":"11"},{"1":"11"},{"1":"8"},{"1":"9"},{"1":"9"},{"1":"13"},{"1":"11"},{"1":"13"},{"1":"8"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

We can also create multiple columns like using `mutate()` in `dplyr`. In  `data.table`, we only need to put back single quotes to `:=` and assign new variables.

```r
mtDT[,  `:=`(cyl_gear2 = cyl * gear,
             cyl_gear3 = cyl - gear)]
names(mtDT)
```

```
##  [1] "carname"     "mpg"         "cyl"         "disp"        "hp"         
##  [6] "drat"        "wt"          "qsec"        "engine_type" "am"         
## [11] "gear"        "carb"        "cyl_gear"    "cyl_gear2"   "cyl_gear3"
```

```r
head(mtDT[, list(cyl_gear2, cyl_gear3)])
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["cyl_gear2"],"name":[1],"type":["int"],"align":["right"]},{"label":["cyl_gear3"],"name":[2],"type":["int"],"align":["right"]}],"data":[{"1":"24","2":"2"},{"1":"24","2":"2"},{"1":"16","2":"0"},{"1":"18","2":"3"},{"1":"24","2":"5"},{"1":"18","2":"3"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

# Summarizing data 💻
Summarising data is more readable and easier to type as it takes fewer key strokes compared to `dplyr`. 


```r
mtDT[,(mean_hp = mean(hp))] # returns a vector
```

```
## [1] 146.6875
```

```r
# Notice what happens using "."
mtDT[,.(mean_hp = mean(hp))] # returns name
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["mean_hp"],"name":[1],"type":["dbl"],"align":["right"]}],"data":[{"1":"146.6875"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
mtDT[,.(mean_hp = mean(hp), sd_hp = sd(hp))] # without the "." at the beggining, you will get an error
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["mean_hp"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["sd_hp"],"name":[2],"type":["dbl"],"align":["right"]}],"data":[{"1":"146.6875","2":"68.56287"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
mtDT[,.(mean_hp = mean(hp), sd_hp = sd(hp)), by = .(engine_type)]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["engine_type"],"name":[1],"type":["int"],"align":["right"]},{"label":["mean_hp"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["sd_hp"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"0","2":"189.72222","3":"60.28150"},{"1":"1","2":"91.35714","3":"24.42447"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# you can remove the . if you're only grouping using one variable
mtDT[,.(mean_hp = mean(hp), sd_hp = sd(hp)), by = engine_type]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["engine_type"],"name":[1],"type":["int"],"align":["right"]},{"label":["mean_hp"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["sd_hp"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"0","2":"189.72222","3":"60.28150"},{"1":"1","2":"91.35714","3":"24.42447"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

We can also compute a frequency table by engine type.

```r
mtDT[, .N, engine_type]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["engine_type"],"name":[1],"type":["int"],"align":["right"]},{"label":["N"],"name":[2],"type":["int"],"align":["right"]}],"data":[{"1":"0","2":"18"},{"1":"1","2":"14"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>
# Chaining 📱
Chaining is like piping in `dplyr`. We only need to attach square brackets at the end, with the next method, to add an additional step in the analysis. We can do multiple data table operations one after the other without having to store intermediate results.

For example, we want to return the average mpg, disp, wt, qsec. Then, order the results by cyl.

```r
mtDT[, .(mean_mpg=mean(mpg),
         mean_disp=mean(disp),
         mean_wt=mean(wt),
         mean_qsec=mean(qsec)), by=cyl]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["cyl"],"name":[1],"type":["int"],"align":["right"]},{"label":["mean_mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["mean_disp"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["mean_wt"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["mean_qsec"],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"6","2":"4.440690","3":"183.3143","4":"3.117143","5":"17.97714"},{"1":"4","2":"5.147091","3":"105.1364","4":"2.285727","5":"19.13727"},{"1":"8","2":"3.872129","3":"353.1000","4":"3.999214","5":"16.77214"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
mtDT[, .(mean_mpg=mean(mpg),
         mean_disp=mean(disp),
         mean_wt=mean(wt),
         mean_qsec=mean(qsec)), by=cyl][order(cyl),]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["cyl"],"name":[1],"type":["int"],"align":["right"]},{"label":["mean_mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["mean_disp"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["mean_wt"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["mean_qsec"],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"4","2":"5.147091","3":"105.1364","4":"2.285727","5":"19.13727"},{"1":"6","2":"4.440690","3":"183.3143","4":"3.117143","5":"17.97714"},{"1":"8","2":"3.872129","3":"353.1000","4":"3.999214","5":"16.77214"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Recall that the `dplyr` piping version of this is:

```r
library(dplyr)
mtDT %>%
  group_by(cyl) %>%
  summarise(mean_mpg=mean(mpg),
         mean_disp=mean(disp),
         mean_wt=mean(wt),
         mean_qsec=mean(qsec)) %>%
  arrange(cyl)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["cyl"],"name":[1],"type":["int"],"align":["right"]},{"label":["mean_mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["mean_disp"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["mean_wt"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["mean_qsec"],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"4","2":"5.147091","3":"105.1364","4":"2.285727","5":"19.13727"},{"1":"6","2":"4.440690","3":"183.3143","4":"3.117143","5":"17.97714"},{"1":"8","2":"3.872129","3":"353.1000","4":"3.999214","5":"16.77214"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

We can also use piping in data tables and create new variables, x, y, z.

```r
mtDT[, x := sqrt(mpg)] %>%
        .[, y := gear^2] %>%
        .[, z := paste0(carname , "2")]

head(mtDT[,x:z])
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["x"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["y"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["z"],"name":[3],"type":["chr"],"align":["left"]}],"data":[{"1":"2.140695","2":"16","3":"Mazda RX42"},{"1":"2.140695","2":"16","3":"Mazda RX4 Wag2"},{"1":"2.185162","2":"16","3":"Datsun 7102"},{"1":"2.150817","2":"9","3":"Hornet 4 Drive2"},{"1":"2.079507","2":"9","3":"Hornet Sportabout2"},{"1":"2.062622","2":"9","3":"Valiant2"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

# Joins with `data.table` 👪
## Key
The concept of a "key" is useful in data tables, especially in merging data. We can set a column as a key using `setkey()`.

```r
setkey(mtDT, carname) # setting carname as key
setkey(mtDT, carname, cyl) # setting multiple keys
```

Remember the chaining example a while ago. We grouped and ordered by `cyl`. Let's re-do that using a shortcut by using `keyby`.

```r
# instead of using this chain
mtDT[, .(mean_mpg=mean(mpg),
         mean_disp=mean(disp),
         mean_wt=mean(wt),
         mean_qsec=mean(qsec)), by=cyl][order(cyl), ]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["cyl"],"name":[1],"type":["int"],"align":["right"]},{"label":["mean_mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["mean_disp"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["mean_wt"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["mean_qsec"],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"4","2":"5.147091","3":"105.1364","4":"2.285727","5":"19.13727"},{"1":"6","2":"4.440690","3":"183.3143","4":"3.117143","5":"17.97714"},{"1":"8","2":"3.872129","3":"353.1000","4":"3.999214","5":"16.77214"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# use keyby to group and order by cyl
mtDT[, .(mean_mpg=mean(mpg),
         mean_disp=mean(disp),
         mean_wt=mean(wt),
         mean_qsec=mean(qsec)), keyby=cyl]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["cyl"],"name":[1],"type":["int"],"align":["right"]},{"label":["mean_mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["mean_disp"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["mean_wt"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["mean_qsec"],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"4","2":"5.147091","3":"105.1364","4":"2.285727","5":"19.13727"},{"1":"6","2":"4.440690","3":"183.3143","4":"3.117143","5":"17.97714"},{"1":"8","2":"3.872129","3":"353.1000","4":"3.999214","5":"16.77214"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
key(mtDT)
```

```
## [1] "carname" "cyl"
```

```r
setkey(mtDT, NULL) # to remove the key
```

## Joining data tables
Now that we know what a key is, we can use this concept to merge or join two data tables.

```r
setkey(mtDT, carname)

# we subset 2 data tables from mtDT
dt1 <- mtDT[5:25,.(carname, mpg, cyl)] # 26 rows
dt2 <- mtDT[1:10, .(carname, gear)] # 10 rows

# Inner Join
merge(dt1, dt2, by='carname') # returns 6 rows from row 5 to 10
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[3],"type":["int"],"align":["right"]},{"label":["gear"],"name":[4],"type":["int"],"align":["right"]}],"data":[{"1":"Datsun 710","2":"4.774935","3":"4","4":"4"},{"1":"Dodge Challenger","2":"3.937004","3":"8","4":"3"},{"1":"Duster 360","2":"3.781534","3":"8","4":"3"},{"1":"Ferrari Dino","2":"4.438468","3":"6","4":"5"},{"1":"Fiat 128","2":"5.692100","3":"4","4":"4"},{"1":"Fiat X1-9","2":"5.224940","3":"4","4":"4"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# Left Join
merge(dt1, dt2, by='carname', all.x = T) # returns 21 rows using dt1 as the base
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[3],"type":["int"],"align":["right"]},{"label":["gear"],"name":[4],"type":["int"],"align":["right"]}],"data":[{"1":"Datsun 710","2":"4.774935","3":"4","4":"4"},{"1":"Dodge Challenger","2":"3.937004","3":"8","4":"3"},{"1":"Duster 360","2":"3.781534","3":"8","4":"3"},{"1":"Ferrari Dino","2":"4.438468","3":"6","4":"5"},{"1":"Fiat 128","2":"5.692100","3":"4","4":"4"},{"1":"Fiat X1-9","2":"5.224940","3":"4","4":"4"},{"1":"Ford Pantera L","2":"3.974921","3":"8","4":"NA"},{"1":"Honda Civic","2":"5.513620","3":"4","4":"NA"},{"1":"Hornet 4 Drive","2":"4.626013","3":"6","4":"NA"},{"1":"Hornet Sportabout","2":"4.324350","3":"8","4":"NA"},{"1":"Lincoln Continental","2":"3.224903","3":"8","4":"NA"},{"1":"Lotus Europa","2":"5.513620","3":"4","4":"NA"},{"1":"Maserati Bora","2":"3.872983","3":"8","4":"NA"},{"1":"Mazda RX4","2":"4.582576","3":"6","4":"NA"},{"1":"Mazda RX4 Wag","2":"4.582576","3":"6","4":"NA"},{"1":"Merc 230","2":"4.774935","3":"4","4":"NA"},{"1":"Merc 240D","2":"4.939636","3":"4","4":"NA"},{"1":"Merc 280","2":"4.381780","3":"6","4":"NA"},{"1":"Merc 280C","2":"4.219005","3":"6","4":"NA"},{"1":"Merc 450SE","2":"4.049691","3":"8","4":"NA"},{"1":"Merc 450SL","2":"4.159327","3":"8","4":"NA"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# Outer Join
merge(dt1, dt2, by='carname', all = T)  # returns 25 rows
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["carname"],"name":[1],"type":["chr"],"align":["left"]},{"label":["mpg"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["cyl"],"name":[3],"type":["int"],"align":["right"]},{"label":["gear"],"name":[4],"type":["int"],"align":["right"]}],"data":[{"1":"AMC Javelin","2":"NA","3":"NA","4":"3"},{"1":"Cadillac Fleetwood","2":"NA","3":"NA","4":"3"},{"1":"Camaro Z28","2":"NA","3":"NA","4":"3"},{"1":"Chrysler Imperial","2":"NA","3":"NA","4":"3"},{"1":"Datsun 710","2":"4.774935","3":"4","4":"4"},{"1":"Dodge Challenger","2":"3.937004","3":"8","4":"3"},{"1":"Duster 360","2":"3.781534","3":"8","4":"3"},{"1":"Ferrari Dino","2":"4.438468","3":"6","4":"5"},{"1":"Fiat 128","2":"5.692100","3":"4","4":"4"},{"1":"Fiat X1-9","2":"5.224940","3":"4","4":"4"},{"1":"Ford Pantera L","2":"3.974921","3":"8","4":"NA"},{"1":"Honda Civic","2":"5.513620","3":"4","4":"NA"},{"1":"Hornet 4 Drive","2":"4.626013","3":"6","4":"NA"},{"1":"Hornet Sportabout","2":"4.324350","3":"8","4":"NA"},{"1":"Lincoln Continental","2":"3.224903","3":"8","4":"NA"},{"1":"Lotus Europa","2":"5.513620","3":"4","4":"NA"},{"1":"Maserati Bora","2":"3.872983","3":"8","4":"NA"},{"1":"Mazda RX4","2":"4.582576","3":"6","4":"NA"},{"1":"Mazda RX4 Wag","2":"4.582576","3":"6","4":"NA"},{"1":"Merc 230","2":"4.774935","3":"4","4":"NA"},{"1":"Merc 240D","2":"4.939636","3":"4","4":"NA"},{"1":"Merc 280","2":"4.381780","3":"6","4":"NA"},{"1":"Merc 280C","2":"4.219005","3":"6","4":"NA"},{"1":"Merc 450SE","2":"4.049691","3":"8","4":"NA"},{"1":"Merc 450SL","2":"4.159327","3":"8","4":"NA"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

# Plotting data 💹
We can also quickly present a simple scatter plot using data.table.

```r
mtDT[,plot(mpg, drat, main="mpg vs. drat")]
```

![](DataWrangling_files/figure-html/unnamed-chunk-28-1.png)<!-- -->

# Further readings 👨‍🎓
`data.table` has many other functions including:

- `dcast()` - pivot/wider/spread
- `melt()` - unpivot/longer/gather

# Exercise ☀️

Using `data.table` and the `mtcars.csv` data, please complete the following exercises.

A) Please create a new variable called `mpg_type` that has the value ‘high’ if mpg > 30. If mpg < 30, then the value should be 'low'. 


```r
# write your code here
```

B) Please import the `mtcars_fun` dataset as a data table. Please name this data table `mt_fun`. Then, merge the `mtcars` and `mtcars_fun` data tables as a new data table called `mt_merged`. Compute the mean and standard  deviation of tires variable by transmission variable or am.


```r
# write your code here
```

C) Please convert the mpg variable into kilometres per litre and assign it to kpl. The formula for conversion is 1 mpg = 2.352 kpl. Compute for the mean mpg, kpl, drat, then group and sort the results by cyl using `keyby`()


```r
# write your code here
```


# Sources 👨‍🏫

This tutorial is partly based on [data.table in R – The Complete Beginners Guide](https://www.machinelearningplus.com/data-manipulation/datatable-in-r-complete-guide/) by Selva Prabhakaran and [Data Wrangling — Raw to Clean Transformation](https://towardsdatascience.com/data-wrangling-raw-to-clean-transformation-b30a27bf4b3b) by Suraj Gurav.



