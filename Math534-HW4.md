# Math 534
Pan

## Q2

1 Point

Grading comment:

How many rows are available in the Measurements table of the Smith
College Wideband Auditory Immittance database?

``` r
library(RMariaDB)
library(dplyr)
```


    载入程序包：'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

``` r
con <- dbConnect(
  MariaDB(), host = "scidb.smith.edu",
  user = "waiuser", password = "smith_waiDB", 
  dbname = "wai"
)
Measurements <- tbl(con, "Measurements")
Measurements %>% count()
```

    # Source:   SQL [1 x 1]
    # Database: mysql  [waiuser@scidb.smith.edu:3306/wai]
            n
      <int64>
    1 5052304

The Measurements table contains 5052304 rows.

## Q3

1 Point

Grading comment:

Identify what years of data are available in the flights table of the
airlines database.

``` r
library(tidyverse) 
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ forcats   1.0.0     ✔ readr     2.1.5
    ✔ ggplot2   3.5.2     ✔ stringr   1.5.1
    ✔ lubridate 1.9.4     ✔ tibble    3.2.1
    ✔ purrr     1.0.2     ✔ tidyr     1.3.1
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(mdsr) 
library(RMariaDB) 
con <- dbConnect_scidb("airlines")
```

``` r
flights <- tbl(con, "flights")
flights %>% head(10)
```

    # Source:   SQL [10 x 21]
    # Database: mysql  [mdsr_public@mdsr.crcbo51tmesf.us-east-2.rds.amazonaws.com:3306/airlines]
        year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
       <int> <int> <int>    <int>          <int>     <int>    <int>          <int>
     1  2013    10     1        2             10        -8      453            505
     2  2013    10     1        4           2359         5      730            729
     3  2013    10     1       11             15        -4      528            530
     4  2013    10     1       14           2355        19      544            540
     5  2013    10     1       16             17        -1      515            525
     6  2013    10     1       22             20         2      552            554
     7  2013    10     1       29             35        -6      808            816
     8  2013    10     1       29             35        -6      449            458
     9  2013    10     1       31             30         1      519            538
    10  2013    10     1       32             33        -1      557            606
    # ℹ 13 more variables: arr_delay <int>, carrier <chr>, tailnum <chr>,
    #   flight <int>, origin <chr>, dest <chr>, air_time <int>, distance <int>,
    #   cancelled <int>, diverted <int>, hour <int>, minute <int>, time_hour <dttm>

``` r
years_available <- tbl(con, "flights") %>%   
  distinct(year) %>%                           
  arrange(year) %>%                           
  collect()                                    

years_available
```

    # A tibble: 3 × 1
       year
      <int>
    1  2013
    2  2014
    3  2015

The flights table contains data for the years 2013-2015.

## Q4

5 Points

Grading comment:

Use the `dbConnect_scidb` function to connect to the airlines database
to answer the following problem.

### Q4.1

1 Point

Grading comment:

How many domestic flights flew into Dallas-Fort Worth (DFW) on May 14,
2015?

``` r
ans <- tbl(con, "flights") %>% 
  filter(year == 2015,
         month == 5,
         day == 14,
         dest == "DFW") %>% 
  summarise(n = n()) %>%        
  collect()
ans
```

    # A tibble: 1 × 1
            n
      <int64>
    1     737

On May 14, 2015, 737 domestic flights arrived at Dallas-Fort Worth
(DFW).

### Q4.2

1 Point

Grading comment:

Of all the destinations from Chicago O’Hare (ORD), which were the most
common in 2015?

``` r
con <- dbConnect_scidb("airlines")

top_dest <- tbl(con, "flights") %>% 
  filter(year == 2015, origin == "ORD") %>%   
  group_by(dest) %>%                          
  tally(sort = TRUE) %>%                      
  head(10) %>%                                
  collect()

top_dest
```

    # A tibble: 10 × 2
       dest        n
       <chr> <int64>
     1 LGA     10492
     2 LAX      8720
     3 DFW      8384
     4 SFO      8156
     5 BOS      7240
     6 ATL      7104
     7 MSP      6955
     8 DCA      6495
     9 DEN      6120
    10 MKE      5274

### Q4.3

1 Point

Grading comment:

Which airport had the highest average arrival delay time in 2015?

``` r
con <- dbConnect_scidb("airlines")
worst_airport <- tbl(con, "flights") %>% 
  filter(year == 2015, !is.na(arr_delay)) %>%   # 只看 2015 有延误记录的航班
  group_by(dest) %>%                            # 按目的地机场分组
  summarise(avg_arr_delay = mean(arr_delay)) %>% 
  arrange(desc(avg_arr_delay)) %>%              # 平均延误最大排最前
  head(1) %>% 
  collect()
```

    Warning: Missing values are always removed in SQL aggregation functions.
    Use `na.rm = TRUE` to silence this warning
    This warning is displayed once every 8 hours.

``` r
worst_airport
```

    # A tibble: 1 × 2
      dest  avg_arr_delay
      <chr>         <dbl>
    1 STC            21.6

In 2015, STC had the highest average arrival delay at 21.6 minutes.

### Q4.4

1 Point

Grading comment:

How many domestic flights came into or flew out of Bradley Airport (BDL)
in 2015?

``` r
con <- dbConnect_scidb("airlines")
bdl_2015 <- tbl(con, "flights") %>% 
  filter(year == 2015,
         (origin == "BDL" | dest == "BDL"),          
        !dest %in% c("SJU", "STT", "STX")) %>%     
  tally() %>% 
  collect()

bdl_2015
```

    # A tibble: 1 × 1
            n
      <int64>
    1   40538

In 2015, 40538 domestic flights either departed from or arrived at
Bradley Airport (BDL).

### Q4.5

1 Point

Grading comment:

List the airline and flight number for all flights between LAX and JFK
on September 26th, 2015.

``` r
con <- dbConnect_scidb("airlines")

lax_jfk_0926 <- tbl(con, "flights") %>% 
  filter(year == 2015,
         month == 9,
         day == 26,
         ((origin == "LAX" & dest == "JFK") |
          (origin == "JFK" & dest == "LAX"))) %>% 
  distinct(carrier, flight) %>%       
  arrange(carrier, flight) %>% 
  mutate(flight_no = str_c(carrier, flight)) %>% 
  collect()
```

    Warning: ORDER BY is ignored in subqueries without LIMIT
    ℹ Do you need to move arrange() later in the pipeline or use window_order() instead?

``` r
lax_jfk_0926
```

    # A tibble: 60 × 3
       carrier flight flight_no
       <chr>    <int> <chr>    
     1 UA         441 UA441    
     2 B6          24 B624     
     3 VX         399 VX399    
     4 DL        1908 DL1908   
     5 B6          23 B623     
     6 AA         118 AA118    
     7 DL         476 DL476    
     8 VX         404 VX404    
     9 AA          34 AA34     
    10 AA          33 AA33     
    # ℹ 50 more rows

## Q5

1 Point

Grading comment:

Wideband acoustic immittance (WAI) is an area of biomedical research
that strives to develop WAI measurements as noninvasive auditory
diagnostic tools. WAI measurements are reported in many related formats,
including absorbance, admittance, impedance, power reflectance, and
pressure reflectance. More information can be found about this public
facing WAI database at
<http://www.science.smith.edu/wai-database/home/about>.

    library(RMariaDB) db <- dbConnect(   MariaDB(),    user = "waiuser",    password = "smith_waiDB",    host = "scidb.smith.edu",    dbname = "wai" )

a\. How many female subjects are there in total across all studies?

``` r
library(tidyverse)
library(RMariaDB)
```

``` r
db <- dbConnect(
  MariaDB(), 
  user = "waiuser", 
  password = "smith_waiDB", 
  host = "scidb.smith.edu", 
  dbname = "wai"
)
dbListTables(db)
```

    [1] "Codebook"             "Measurements"         "Measurements_pre2020"
    [4] "PI_Info"              "PI_Info_OLD"          "Subjects"            
    [7] "Subjects_pre2020"    

``` r
dbGetQuery(db, "DESCRIBE Subjects")[, "Field"]
```

     [1] "Identifier"                     "SubjectNumber"                 
     [3] "SessionTotal"                   "AgeFirstMeasurement"           
     [5] "AgeCategoryFirstMeasurement"    "Sex"                           
     [7] "Race"                           "Ethnicity"                     
     [9] "LeftEarStatusFirstMeasurement"  "RightEarStatusFirstMeasurement"
    [11] "SubjectNotes"                  

``` r
tbl(db, "Subjects") %>% 
  count(Sex) %>% 
  collect()
```

    # A tibble: 4 × 2
      Sex           n
      <chr>   <int64>
    1 Female     4727
    2 Male       3977
    3 Unknown    1441
    4 Femake        1

``` r
n_female <- tbl(db, "Subjects") %>% 
  filter(Sex %in% c("Female", "Femake")) %>%   
  tally() %>% 
  collect()
n_female
```

    # A tibble: 1 × 1
            n
      <int64>
    1    4728

Across all studies, the WAI database contains 4728 female subjects
(including one record with a typo “Femake”).

b\. Find the average absorbance for participants for each study, ordered
by highest to lowest value.

``` r
db <- dbConnect(
  MariaDB(), 
  user = "waiuser", 
  password = "smith_waiDB", 
  host = "scidb.smith.edu", 
  dbname = "wai"
)
dbGetQuery(db, "DESCRIBE Measurements")[, "Field"]
```

     [1] "Identifier"     "SubjectNumber"  "Session"        "Ear"           
     [5] "Instrument"     "Age"            "AgeCategory"    "EarStatus"     
     [9] "TPP"            "AreaCanal"      "PressureCanal"  "SweepDirection"
    [13] "Frequency"      "Absorbance"     "Zmag"           "Zang"          

``` r
dbGetQuery(db, "DESCRIBE Subjects")[, "Field"]
```

     [1] "Identifier"                     "SubjectNumber"                 
     [3] "SessionTotal"                   "AgeFirstMeasurement"           
     [5] "AgeCategoryFirstMeasurement"    "Sex"                           
     [7] "Race"                           "Ethnicity"                     
     [9] "LeftEarStatusFirstMeasurement"  "RightEarStatusFirstMeasurement"
    [11] "SubjectNotes"                  

``` r
subj <- tbl(db, "Subjects") %>% select(SubjectNumber, Identifier) %>% collect()
meas <- tbl(db, "Measurements") %>% select(SubjectNumber, Absorbance) %>% collect()
avg_abs_by_study <-
  meas %>% 
  inner_join(
    subj,
    by = "SubjectNumber",
    relationship = "many-to-many"   
  ) %>% 
  group_by(Identifier) %>%          
  summarise(
    n_measurements = n(),
    avg_abs        = mean(Absorbance, na.rm = TRUE)
  ) %>% 
  arrange(desc(avg_abs))

avg_abs_by_study
```

    # A tibble: 44 × 3
       Identifier    n_measurements avg_abs
       <chr>                  <int>   <dbl>
     1 Keefe_2003            106384   0.549
     2 Sun_2023              598130   0.529
     3 Hunter_2016            49005   0.502
     4 Keefe_2017             10596   0.451
     5 Merchant_2010         618307   0.432
     6 Sanford_2014          605285   0.430
     7 Merchant_2020         895435   0.428
     8 Downing_2022         1798603   0.424
     9 Rosowski_2012         826637   0.421
    10 Nakajima_2012         636986   0.421
    # ℹ 34 more rows

c\. Write a query to count all the measurements with a calculated
absorbance of less than 0.

``` r
n_neg <- tbl(db, "Measurements") %>% 
  filter(Absorbance < 0) %>% 
  tally() %>% 
  collect()

n_neg
```

    # A tibble: 1 × 1
            n
      <int64>
    1   28694

A total of 28694 measurements in the database have a calculated
absorbance less than zero.

## Q6

1 Point

Grading comment:

The following open-ended question may require more than one query and a
thoughtful response. Based on data from 2013 only, and assuming that
transportation to the airport is not an issue, would you rather fly out
of JFK, LaGuardia (LGA), or Newark (EWR)? Why or why not? Use the
dbConnect_scidb function to connect to the airlines database.

``` r
con <- dbConnect_scidb("airlines")
summary_2013 <- tbl(con, "flights") %>% 
  filter(year == 2013, origin %in% c("JFK", "LGA", "EWR")) %>% 
  mutate(
    on_time = dep_delay <= 15,               
    cancelled = is.na(dep_delay) & is.na(arr_delay)  
  ) %>% 
  group_by(origin) %>% 
  summarise(
    n_flights   = n(),
    avg_delay   = mean(dep_delay, na.rm = TRUE),
    cancel_rate = mean(cancelled, na.rm = TRUE),
    on_time_rate = mean(on_time, na.rm = TRUE)
  ) %>% 
  arrange(avg_delay) %>%   
  collect()

summary_2013
```

    # A tibble: 3 × 5
      origin n_flights avg_delay cancel_rate on_time_rate
      <chr>    <int64>     <dbl>       <dbl>        <dbl>
    1 LGA       104662      10.0           0        0.817
    2 JFK       111279      11.9           0        0.796
    3 EWR       120835      14.7           0        0.760

In 2013, LGA had the lowest average departure delay (10 min), the lowest
cancellation rate (0), and the highest on-time rate (81.6 %).  
Therefore, I would choose to fly on LGA in 2013, assuming ground
transportation is equally convenient.
