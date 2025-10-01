# Math 534 （new）
Pan

``` r
library(RMariaDB)
library(DBI)
library(tidyverse)
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ✔ ggplot2   3.5.2     ✔ tibble    3.2.1
    ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ✔ purrr     1.0.2     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(mdsr)
```

Q2 1 Point Grading comment: How many rows are available in the
Measurements table of the Smith College Wideband Auditory Immittance
database?

``` r
con <- dbConnect(
  MariaDB(), host = "scidb.smith.edu",
  user = "waiuser", password = "smith_waiDB", 
  dbname = "wai"
)

n_rows <- dbGetQuery(con, "SELECT COUNT(*) AS n FROM Measurements")[1, 1]
n_rows 
```

    integer64
    [1] 5052304

The Measurements table contains 5052304 rows.

Q3 1 Point Grading comment: Identify what years of data are available in
the flights table of the airlines database.

``` r
con <- dbConnect_scidb("airlines")
tables <- dbGetQuery(con, "SHOW TABLES")
print(tables)
```

      Tables_in_airlines
    1           airports
    2           carriers
    3            flights
    4    flights_summary
    5             planes

``` r
flights_schema <- dbGetQuery(con, "DESCRIBE flights")
print(flights_schema)
```

                Field        Type Null Key Default Extra
    1            year smallint(4)  YES        <NA>      
    2           month smallint(2)  YES        <NA>      
    3             day smallint(2)  YES        <NA>      
    4        dep_time smallint(4)  YES        <NA>      
    5  sched_dep_time smallint(4)  YES        <NA>      
    6       dep_delay smallint(4)  YES        <NA>      
    7        arr_time smallint(4)  YES        <NA>      
    8  sched_arr_time smallint(4)  YES        <NA>      
    9       arr_delay smallint(4)  YES        <NA>      
    10        carrier  varchar(2)   NO                  
    11        tailnum  varchar(6)  YES MUL    <NA>      
    12         flight smallint(4)  YES        <NA>      
    13         origin  varchar(3)   NO MUL              
    14           dest  varchar(3)   NO MUL              
    15       air_time smallint(4)  YES        <NA>      
    16       distance smallint(4)  YES        <NA>      
    17      cancelled  tinyint(1)  YES        <NA>      
    18       diverted  tinyint(1)  YES        <NA>      
    19           hour smallint(2)  YES        <NA>      
    20         minute smallint(2)  YES        <NA>      
    21      time_hour    datetime  YES        <NA>      

``` r
years <- dbGetQuery(con, "
  SELECT DISTINCT year
  FROM   flights
  ORDER  BY year
")$year
years 
```

    [1] 2013 2014 2015

The flights table contains data for the years 2013-2015.

Q4 5 Points Grading comment: Use the dbConnect_scidb function to connect
to the airlines database to answer the following problem. Q4.1 1 Point
Grading comment: How many domestic flights flew into Dallas-Fort Worth
(DFW) on May 14, 2015?

``` r
head_sql <- dbGetQuery(con, "SELECT * FROM flights LIMIT 6")
print(head_sql)
```

      year month day dep_time sched_dep_time dep_delay arr_time sched_arr_time
    1 2013    10   1        2             10        -8      453            505
    2 2013    10   1        4           2359         5      730            729
    3 2013    10   1       11             15        -4      528            530
    4 2013    10   1       14           2355        19      544            540
    5 2013    10   1       16             17        -1      515            525
    6 2013    10   1       22             20         2      552            554
      arr_delay carrier tailnum flight origin dest air_time distance cancelled
    1       -12      AA  N201AA   2400    LAX  DFW      149     1235         0
    2         1      FL  N344AT    710    SFO  ATL      247     2139         0
    3        -2      AA  N3KMAA   1052    SFO  DFW      182     1464         0
    4         4      AA  N3ENAA   2392    SEA  ORD      191     1721         0
    5       -10      UA  N38473   1614    LAX  IAH      157     1379         0
    6        -2      UA  N458UA    291    SFO  IAH      188     1635         0
      diverted hour minute           time_hour
    1        0    0     10 2013-10-01 00:10:00
    2        0   23     59 2013-10-01 23:59:00
    3        0    0     15 2013-10-01 00:15:00
    4        0   23     55 2013-10-01 23:55:00
    5        0    0     17 2013-10-01 00:17:00
    6        0    0     20 2013-10-01 00:20:00

``` r
sql <- "
SELECT COUNT(*) AS n_flights
FROM flights
WHERE dest = 'DFW'
  AND year = 2015
  AND month = 5
  AND day = 14
"
ans <- dbGetQuery(con, sql)[1, "n_flights"]
ans
```

    integer64
    [1] 737

On May 14, 2015, 737 domestic flights arrived at Dallas-Fort Worth
(DFW).

Q4.2 1 Point Grading comment: Of all the destinations from Chicago
O’Hare (ORD), which were the most common in 2015?

``` r
sql <- "
SELECT dest,
       COUNT(*) AS n_flights
FROM   flights
WHERE  origin = 'ORD'
  AND  year   = 2015
GROUP  BY dest
ORDER  BY n_flights DESC
LIMIT  10            
"
top_dest <- dbGetQuery(con, sql)
print(top_dest)
```

       dest n_flights
    1   LGA     10492
    2   LAX      8720
    3   DFW      8384
    4   SFO      8156
    5   BOS      7240
    6   ATL      7104
    7   MSP      6955
    8   DCA      6495
    9   DEN      6120
    10  MKE      5274

Q4.3 1 Point Grading comment: Which airport had the highest average
arrival delay time in 2015?

``` r
sql <- "
SELECT dest,
       AVG(arr_delay) AS avg_arr_delay
FROM   flights
WHERE  year = 2015
       AND arr_delay IS NOT NULL        
GROUP  BY dest
ORDER  BY avg_arr_delay DESC
LIMIT  1
"

ans <- dbGetQuery(con, sql)
ans
```

      dest avg_arr_delay
    1  STC        21.622

In 2015, STC had the highest average arrival delay at 21.6 minutes.

Q4.4 1 Point Grading comment: How many domestic flights came into or
flew out of Bradley Airport (BDL) in 2015?

``` r
sql <- "
SELECT SUM(cnt) AS total_flights
FROM (
    SELECT COUNT(*) AS cnt
    FROM flights
    WHERE year = 2015 AND dest = 'BDL'
    UNION ALL
    SELECT COUNT(*) AS cnt
    FROM flights
    WHERE year = 2015 AND origin = 'BDL'
) AS u
"

dbGetQuery(con, sql)[1, "total_flights"]
```

    [1] 41025

In 2015, 41025 domestic flights either departed from or arrived at
Bradley Airport (BDL).

Q4.5 1 Point Grading comment: List the airline and flight number for all
flights between LAX and JFK on September 26th, 2015.

``` r
sql <- "
SELECT DISTINCT carrier, flight
FROM flights
WHERE year = 2015
  AND month = 9
  AND day = 26
  AND ((origin = 'LAX' AND dest = 'JFK') OR
       (origin = 'JFK' AND dest = 'LAX'))
ORDER BY carrier, flight
"

dbGetQuery(con, sql)
```

       carrier flight
    1       AA      2
    2       AA      3
    3       AA      4
    4       AA     19
    5       AA     21
    6       AA     22
    7       AA     30
    8       AA     32
    9       AA     33
    10      AA     34
    11      AA    117
    12      AA    118
    13      AA    180
    14      AA    255
    15      AA    293
    16      B6     23
    17      B6     24
    18      B6    123
    19      B6    124
    20      B6    223
    21      B6    224
    22      B6    323
    23      B6    324
    24      B6    423
    25      B6    424
    26      B6    523
    27      B6    524
    28      B6    623
    29      B6    624
    30      DL    412
    31      DL    420
    32      DL    423
    33      DL    447
    34      DL    464
    35      DL    472
    36      DL    476
    37      DL    477
    38      DL    920
    39      DL   1162
    40      DL   1262
    41      DL   1908
    42      UA    275
    43      UA    441
    44      UA    535
    45      UA    703
    46      UA    779
    47      UA    841
    48      UA    912
    49      UA   1752
    50      UA   1985
    51      VX    399
    52      VX    404
    53      VX    406
    54      VX    407
    55      VX    411
    56      VX    412
    57      VX    413
    58      VX    415
    59      VX    416
    60      VX    420

Q5 1 Point Grading comment: Wideband acoustic immittance (WAI) is an
area of biomedical research that strives to develop WAI measurements as
noninvasive auditory diagnostic tools. WAI measurements are reported in
many related formats, including absorbance, admittance, impedance, power
reflectance, and pressure reflectance. More information can be found
about this public facing WAI database at
http://www.science.smith.edu/wai-database/home/about.

``` r
library(RMariaDB) 
db <- dbConnect(
  MariaDB(),    user = "waiuser",    password = "smith_waiDB",    host = "scidb.smith.edu",    dbname = "wai" )
```

1.  How many female subjects are there in total across all studies?

``` r
dbListTables(db)
```

    [1] "Codebook"             "Measurements"         "Measurements_pre2020"
    [4] "PI_Info"              "PI_Info_OLD"          "Subjects"            
    [7] "Subjects_pre2020"    

``` r
head_subjects <- dbGetQuery(db, "SELECT * FROM Subjects LIMIT 6")
print(head_subjects)
```

      Identifier SubjectNumber SessionTotal AgeFirstMeasurement
    1  Abur_2014             1            7                  20
    2  Abur_2014             3            8                  19
    3  Abur_2014             4            7                  21
    4  Abur_2014             6            8                  21
    5  Abur_2014             7            5                  20
    6  Abur_2014             8            5                  19
      AgeCategoryFirstMeasurement    Sex    Race Ethnicity
    1                       Adult Female Unknown   Unknown
    2                       Adult Female Unknown   Unknown
    3                       Adult Female Unknown   Unknown
    4                       Adult Female Unknown   Unknown
    5                       Adult Female Unknown   Unknown
    6                       Adult Female Unknown   Unknown
      LeftEarStatusFirstMeasurement RightEarStatusFirstMeasurement
    1                        Normal                         Normal
    2                        Normal                         Normal
    3                        Normal                         Normal
    4                        Normal                         Normal
    5                        Normal                         Normal
    6                        Normal                         Normal
                                    SubjectNotes
    1                                           
    2 Session 5 not included do to acoustic leak
    3                                           
    4                                           
    5                                           
    6                                           

``` r
n_female <- dbGetQuery(db,
  "SELECT COUNT(*) AS n_female
   FROM Subjects
   WHERE Sex = 'Female'")[1, "n_female"]

n_female
```

    integer64
    [1] 4727

Across all studies, the WAI database contains 4727 female subjects.

2.  Find the average absorbance for participants for each study, ordered
    by highest to lowest value.

``` r
head_measurements <- dbGetQuery(db, "SELECT * FROM Measurements LIMIT 6")
print(head_measurements)
```

      Identifier SubjectNumber Session  Ear Instrument Age AgeCategory EarStatus
    1  Abur_2014             1       1 Left     HearID  20       Adult    Normal
    2  Abur_2014             1       1 Left     HearID  20       Adult    Normal
    3  Abur_2014             1       1 Left     HearID  20       Adult    Normal
    4  Abur_2014             1       1 Left     HearID  20       Adult    Normal
    5  Abur_2014             1       1 Left     HearID  20       Adult    Normal
    6  Abur_2014             1       1 Left     HearID  20       Adult    Normal
      TPP AreaCanal PressureCanal SweepDirection Frequency Absorbance      Zmag
    1  -5  4.42e-05             0        Ambient   210.938  0.0333379 113780000
    2  -5  4.42e-05             0        Ambient   234.375  0.0315705 103585000
    3  -5  4.42e-05             0        Ambient   257.812  0.0405751  92951696
    4  -5  4.42e-05             0        Ambient   281.250  0.0438399  86058000
    5  -5  4.42e-05             0        Ambient   304.688  0.0486400  79492800
    6  -5  4.42e-05             0        Ambient   328.125  0.0527801  73326200
           Zang
    1 -0.233504
    2 -0.235778
    3 -0.233482
    4 -0.233421
    5 -0.232931
    6 -0.232837

``` r
sql <- "
SELECT SUBSTRING_INDEX(s.Identifier,'_',1) AS studyID,
       AVG(m.Absorbance)                   AS avg_abs
FROM   Subjects s
JOIN   Measurements m ON s.SubjectNumber = m.SubjectNumber
WHERE  m.Absorbance IS NOT NULL
GROUP  BY studyID
ORDER  BY avg_abs DESC
"
dbGetQuery(db, sql)
```

         studyID   avg_abs
    1     Hunter 0.5015484
    2        Sun 0.4391928
    3    Downing 0.4237462
    4   Rosowski 0.4214976
    5   Nakajima 0.4208143
    6   Merchant 0.4155671
    7     Shaver 0.4145624
    8     Werner 0.4119051
    9      Myers 0.4119042
    10       Liu 0.4109574
    11 AlMakadma 0.4104801
    12    Aithal 0.4077721
    13   Sanford 0.4054889
    14   Shahnaz 0.4019397
    15     Sliwa 0.4013881
    16    Pitaro 0.3969542
    17   Ellison 0.3956476
    18     Keefe 0.3914554
    19     Lewis 0.3899812
    20 Scheperle 0.3888696
    21     Groon 0.3876838
    22      Voss 0.3762544
    23    Feeney 0.3614771
    24      Abur 0.3284249

3.  Write a query to count all the measurements with a calculated
    absorbance of less than 0.

``` r
n_low_abs <- dbGetQuery(db,
  "SELECT COUNT(*) AS n_low_abs
   FROM Measurements
   WHERE Absorbance < 0")[1, "n_low_abs"]
n_low_abs
```

    integer64
    [1] 28694

A total of 28694 measurements in the database have a calculated
absorbance less than zero.

Q6 1 Point Grading comment: The following open-ended question may
require more than one query and a thoughtful response. Based on data
from 2013 only, and assuming that transportation to the airport is not
an issue, would you rather fly out of JFK, LaGuardia (LGA), or Newark
(EWR)? Why or why not? Use the dbConnect_scidb function to connect to
the airlines database.

``` r
sql <- "
SELECT origin,
       COUNT(*) AS n_flights,
       AVG(arr_delay) AS avg_arr_delay,
       SUM(arr_delay <= 15) / COUNT(*) AS pct_ontime
FROM flights
WHERE year = 2013
  AND origin IN ('JFK','LGA','EWR')
  AND arr_delay IS NOT NULL
GROUP BY origin
ORDER BY pct_ontime DESC
"

dbGetQuery(con, sql)
```

      origin n_flights avg_arr_delay pct_ontime
    1    LGA    104662        5.5889     0.7840
    2    JFK    111279        5.4417     0.7749
    3    EWR    120835        8.8276     0.7520

Based on 2013 arrival statistics, LaGuardia (LGA) offers the best
on-time performance (78.4 %) and the shortest average arrival delay (5.6
min). Therefore, I would choose to fly on LGA in 2013, assuming ground
transportation is equally convenient.
