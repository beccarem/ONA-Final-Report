Final Report
================
Chelsea, Rebecca, Diwei, Dany

# 1. Introduction

# 2. Methodology

Load the applications data from ‘app\_data\_sample.parquet’ and edges
data from ‘edges\_sample.csv’

``` r
applications <- read_parquet("app_data_sample.parquet")
edges <- read_csv("edges_sample.csv")
```

    ## Rows: 32906 Columns: 4

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (1): application_number
    ## dbl  (2): ego_examiner_id, alter_examiner_id
    ## date (1): advice_date

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

### Get gender for examiners

We’ll get gender based on the first name of the examiner, which is
recorded in the field `examiner_name_first`. We’ll use library `gender`
for that, relying on a modified version of their own
[example](https://cran.r-project.org/web/packages/gender/vignettes/predicting-gender.html).

Note that there are over 2 million records in the applications table –
that’s because there are many records for each examiner, as many as the
number of applications that examiner worked on during this time frame.
Our first step therefore is to get all *unique* names in a separate list
`examiner_names`. We will then guess gender for each one and will join
this table back to the original dataset. So, let’s get names without
repetition:

``` r
library(gender)
#install_genderdata_package() # only run this line the first time you use the package, to get data for it
# get a list of first names without repetitions
examiner_names <- applications %>% 
  distinct(examiner_name_first)
#examiner_names
```

Now let’s use function `gender()` as shown in the example for the
package to attach a gender and probability to each name and put the
results into the table `examiner_names_gender`

``` r
# get a table of names and gender
examiner_names_gender <- examiner_names %>% 
  do(results = gender(.$examiner_name_first, method = "ssa")) %>% 
  unnest(cols = c(results), keep_empty = TRUE) %>% 
  select(
    examiner_name_first = name,
    gender,
    proportion_female
  )
examiner_names_gender
```

    ## # A tibble: 1,822 × 3
    ##    examiner_name_first gender proportion_female
    ##    <chr>               <chr>              <dbl>
    ##  1 AARON               male              0.0082
    ##  2 ABDEL               male              0     
    ##  3 ABDOU               male              0     
    ##  4 ABDUL               male              0     
    ##  5 ABDULHAKIM          male              0     
    ##  6 ABDULLAH            male              0     
    ##  7 ABDULLAHI           male              0     
    ##  8 ABIGAIL             female            0.998 
    ##  9 ABIMBOLA            female            0.944 
    ## 10 ABRAHAM             male              0.0031
    ## # … with 1,812 more rows

Finally, let’s join that table back to our original applications data
and discard the temporary tables we have just created to reduce clutter
in our environment.

``` r
# remove extra colums from the gender table
examiner_names_gender <- examiner_names_gender %>% 
  select(examiner_name_first, gender)
# joining gender back to the dataset
applications <- applications %>% 
  left_join(examiner_names_gender, by = "examiner_name_first")
# cleaning up
rm(examiner_names)
rm(examiner_names_gender)
#gc()
```

### Guess the examiner’s race

We’ll now use package `wru` to estimate likely race of an examiner. Just
like with gender, we’ll get a list of unique names first, only now we
are using surnames.

``` r
library(wru)
examiner_surnames <- applications %>% 
  select(surname = examiner_name_last) %>% 
  distinct()
#examiner_surnames
```

We’ll follow the instructions for the package outlined here
<https://github.com/kosukeimai/wru>.

``` r
examiner_race <- predict_race(voter.file = examiner_surnames, surname.only = T) %>% 
  as_tibble()
```

    ## [1] "Proceeding with surname-only predictions..."

    ## Warning in merge_surnames(voter.file): Probabilities were imputed for 698
    ## surnames that could not be matched to Census list.

``` r
#examiner_race
```

As you can see, we get probabilities across five broad US Census
categories: white, black, Hispanic, Asian and other. (Some of you may
correctly point out that Hispanic is not a race category in the US
Census, but these are the limitations of this package.)

Our final step here is to pick the race category that has the highest
probability for each last name and then join the table back to the main
applications table. See this example for comparing values across
columns: <https://www.tidyverse.org/blog/2020/04/dplyr-1-0-0-rowwise/>.
And this one for `case_when()` function:
<https://dplyr.tidyverse.org/reference/case_when.html>.

``` r
examiner_race <- examiner_race %>% 
  mutate(max_race_p = pmax(pred.asi, pred.bla, pred.his, pred.oth, pred.whi)) %>% 
  mutate(race = case_when(
    max_race_p == pred.asi ~ "Asian",
    max_race_p == pred.bla ~ "black",
    max_race_p == pred.his ~ "Hispanic",
    max_race_p == pred.oth ~ "other",
    max_race_p == pred.whi ~ "white",
    TRUE ~ NA_character_
  ))
examiner_race
```

    ## # A tibble: 3,806 × 8
    ##    surname    pred.whi pred.bla pred.his pred.asi pred.oth max_race_p race 
    ##    <chr>         <dbl>    <dbl>    <dbl>    <dbl>    <dbl>      <dbl> <chr>
    ##  1 HOWARD       0.643   0.295    0.0237   0.005     0.0333      0.643 white
    ##  2 YILDIRIM     0.861   0.0271   0.0609   0.0135    0.0372      0.861 white
    ##  3 HAMILTON     0.702   0.237    0.0245   0.0054    0.0309      0.702 white
    ##  4 MOSHER       0.947   0.00410  0.0241   0.00640   0.0185      0.947 white
    ##  5 BARR         0.827   0.117    0.0226   0.00590   0.0271      0.827 white
    ##  6 GRAY         0.687   0.251    0.0241   0.0054    0.0324      0.687 white
    ##  7 MCMILLIAN    0.359   0.574    0.0189   0.00260   0.0463      0.574 black
    ##  8 FORD         0.620   0.32     0.0237   0.0045    0.0313      0.620 white
    ##  9 STRZELECKA   0.666   0.0853   0.137    0.0797    0.0318      0.666 white
    ## 10 KIM          0.0252  0.00390  0.00650  0.945     0.0198      0.945 Asian
    ## # … with 3,796 more rows

Let’s join the data back to the applications table.

``` r
# removing extra columns
examiner_race <- examiner_race %>% 
  select(surname,race)
applications <- applications %>% 
  left_join(examiner_race, by = c("examiner_name_last" = "surname"))
rm(examiner_race)
rm(examiner_surnames)
#gc()
```

### Examiner’s tenure

To figure out the timespan for which we observe each examiner in the
applications data, let’s find the first and the last observed date for
each examiner. We’ll first get examiner IDs and application dates in a
separate table, for ease of manipulation. We’ll keep examiner ID (the
field `examiner_id`), and earliest and latest dates for each application
(`filing_date` and `appl_status_date` respectively). We’ll use functions
in package `lubridate` to work with date and time values.

``` r
library(lubridate) # to work with dates
examiner_dates <- applications %>% 
  select(examiner_id, filing_date, appl_status_date) 
#examiner_dates
```

The dates look inconsistent in terms of formatting. Let’s make them
consistent. We’ll create new variables `start_date` and `end_date`.

``` r
examiner_dates <- examiner_dates %>% 
  mutate(start_date = ymd(filing_date), end_date = as_date(dmy_hms(appl_status_date)))
```

Let’s now identify the earliest and the latest date for each examiner
and calculate the difference in days, which is their tenure in the
organization.

``` r
examiner_dates <- examiner_dates %>% 
  group_by(examiner_id) %>% 
  summarise(
    earliest_date = min(start_date, na.rm = TRUE), 
    latest_date = max(end_date, na.rm = TRUE),
    tenure_days = interval(earliest_date, latest_date) %/% days(1)
    ) %>% 
  filter(year(latest_date)<2018)
#examiner_dates

## plot tenure
library(skimr)
examiner_dates %>% 
  select(earliest_date,latest_date,tenure_days) %>% 
  skim()
```

|                                                  |            |
|:-------------------------------------------------|:-----------|
| Name                                             | Piped data |
| Number of rows                                   | 5625       |
| Number of columns                                | 3          |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   |            |
| Column type frequency:                           |            |
| Date                                             | 2          |
| numeric                                          | 1          |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ |            |
| Group variables                                  | None       |

Data summary

**Variable type: Date**

| skim\_variable | n\_missing | complete\_rate | min        | max        | median     | n\_unique |
|:---------------|-----------:|---------------:|:-----------|:-----------|:-----------|----------:|
| earliest\_date |          0 |              1 | 2000-01-02 | 2016-03-03 | 2003-01-21 |      2322 |
| latest\_date   |          0 |              1 | 2000-09-14 | 2017-12-06 | 2017-05-19 |       870 |

**Variable type: numeric**

| skim\_variable | n\_missing | complete\_rate |    mean |      sd |  p0 |  p25 |  p50 |  p75 | p100 | hist  |
|:---------------|-----------:|---------------:|--------:|--------:|----:|-----:|-----:|-----:|-----:|:------|
| tenure\_days   |          0 |              1 | 4429.51 | 1795.54 |  27 | 3096 | 4912 | 6091 | 6518 | ▂▂▂▅▇ |

Joining back to the applications data.

``` r
applications <- applications %>% 
  left_join(examiner_dates, by = "examiner_id")
#rm(examiner_dates)
#gc()
```

### Create application processing time variable

``` r
attach(applications)
library(lubridate)

# compute the final decision date as either abandon date or patent issue date
application_dates <- applications %>% 
    mutate(decision_date = coalesce(abandon_date,patent_issue_date)) %>%
    select(application_number,filing_date, abandon_date, patent_issue_date, decision_date, examiner_id, examiner_art_unit, gender, race, tenure_days) %>%
    filter(!is.na(decision_date))

head(application_dates)
```

    ## # A tibble: 6 × 10
    ##   application_number filing_date abandon_date patent_issue_date decision_date
    ##   <chr>              <date>      <date>       <date>            <date>       
    ## 1 08284457           2000-01-26  NA           2003-02-18        2003-02-18   
    ## 2 08413193           2000-10-11  NA           2002-08-27        2002-08-27   
    ## 3 08531853           2000-05-17  NA           1997-03-04        1997-03-04   
    ## 4 08637752           2001-07-20  NA           2005-08-09        2005-08-09   
    ## 5 08682726           2000-04-10  2000-12-27   NA                2000-12-27   
    ## 6 08687412           2000-04-28  NA           2001-07-31        2001-07-31   
    ## # … with 5 more variables: examiner_id <dbl>, examiner_art_unit <dbl>,
    ## #   gender <chr>, race <chr>, tenure_days <dbl>

``` r
# compute the application processing time as the difference of filing date and decision date
application_dates <- application_dates %>% 
    #mutate(app_proc_time = decision_date - filing_date)
    mutate(app_proc_time = difftime(decision_date, filing_date, units = "days"))

head(application_dates) #1,688,716 applications
```

    ## # A tibble: 6 × 11
    ##   application_number filing_date abandon_date patent_issue_date decision_date
    ##   <chr>              <date>      <date>       <date>            <date>       
    ## 1 08284457           2000-01-26  NA           2003-02-18        2003-02-18   
    ## 2 08413193           2000-10-11  NA           2002-08-27        2002-08-27   
    ## 3 08531853           2000-05-17  NA           1997-03-04        1997-03-04   
    ## 4 08637752           2001-07-20  NA           2005-08-09        2005-08-09   
    ## 5 08682726           2000-04-10  2000-12-27   NA                2000-12-27   
    ## 6 08687412           2000-04-28  NA           2001-07-31        2001-07-31   
    ## # … with 6 more variables: examiner_id <dbl>, examiner_art_unit <dbl>,
    ## #   gender <chr>, race <chr>, tenure_days <dbl>, app_proc_time <drtn>

It seems some application processing time have negative value
abnormally. Let’s take a look at the distribution.

``` r
# plot the data distribution of application processing time
application_dates %>%
  ggplot(aes(sample = app_proc_time)) +
  geom_qq()
```

    ## Don't know how to automatically pick scale for object of type difftime. Defaulting to continuous.
    ## Don't know how to automatically pick scale for object of type difftime. Defaulting to continuous.

![](final_report_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
# filter out negative and outlying application processing time
application_dates <- application_dates %>% 
    filter(app_proc_time>ddays(0)) %>% 
    filter(app_proc_time<ddays(10000))

head(application_dates) #1,688,672 applications
```

    ## # A tibble: 6 × 11
    ##   application_number filing_date abandon_date patent_issue_date decision_date
    ##   <chr>              <date>      <date>       <date>            <date>       
    ## 1 08284457           2000-01-26  NA           2003-02-18        2003-02-18   
    ## 2 08413193           2000-10-11  NA           2002-08-27        2002-08-27   
    ## 3 08637752           2001-07-20  NA           2005-08-09        2005-08-09   
    ## 4 08682726           2000-04-10  2000-12-27   NA                2000-12-27   
    ## 5 08687412           2000-04-28  NA           2001-07-31        2001-07-31   
    ## 6 08765941           2000-06-23  2001-08-22   NA                2001-08-22   
    ## # … with 6 more variables: examiner_id <dbl>, examiner_art_unit <dbl>,
    ## #   gender <chr>, race <chr>, tenure_days <dbl>, app_proc_time <drtn>

``` r
# plot again the data distribution of application processing time after cleaning
application_dates %>%
  ggplot(aes(sample = app_proc_time)) +
  geom_qq()
```

    ## Don't know how to automatically pick scale for object of type difftime. Defaulting to continuous.
    ## Don't know how to automatically pick scale for object of type difftime. Defaulting to continuous.

![](final_report_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->
Outliers are removed successfully.

### Get work group from art unit

``` r
# before we begin, get the workgroup from art unit as rounding down to digit tenth.
application_dates <- application_dates %>%
  mutate(wg = (application_dates$examiner_art_unit%/%10) * 10)

# Find out which is the dominating workgroup an examiner handled the applications for.
library(plyr)
```

    ## ------------------------------------------------------------------------------

    ## You have loaded plyr after dplyr - this is likely to cause problems.
    ## If you need functions from both plyr and dplyr, please load plyr first, then dplyr:
    ## library(plyr); library(dplyr)

    ## ------------------------------------------------------------------------------

    ## 
    ## Attaching package: 'plyr'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     arrange, count, desc, failwith, id, mutate, rename, summarise,
    ##     summarize

    ## The following object is masked from 'package:purrr':
    ## 
    ##     compact

``` r
library(dplyr)
library(lubridate)
application_dates <- mutate(
  application_dates,
  period = case_when(
    filing_date<ymd("2007-01-01") ~ NA_character_,
    filing_date<ymd("2008-01-01") ~ "t0",
    filing_date<ymd("2009-01-01") ~ "t1",
    filing_date<ymd("2010-01-01") ~ "t2",
    filing_date<ymd("2011-01-01") ~ "t3",
    filing_date<ymd("2012-01-01") ~ "t4",
    filing_date<ymd("2013-01-01") ~ "t5",
    filing_date<ymd("2014-01-01") ~ "t6",
    filing_date<ymd("2015-01-01") ~ "t7",
    filing_date<ymd("2016-01-01") ~ "t8",
    TRUE~ NA_character_)
  )

# get number of applications
library(plyr)
examiner_wg_napp <- ddply(application_dates, .(examiner_id, period, wg), nrow)
names(examiner_wg_napp) <- c("examiner_id","period", "wg", "n_applications")

# assume an examiner belong to the wg he/she most frequently handled applications for, if tie take the greater wg
examiner_wg_napp <- examiner_wg_napp[order(examiner_wg_napp$examiner_id, examiner_wg_napp$period, -(examiner_wg_napp$n_applications), -(examiner_wg_napp$wg)), ] ### sort first
examiner_wg <- examiner_wg_napp [!duplicated(examiner_wg_napp[c(1,2)]),]
examiner_wg <- select(examiner_wg, c("examiner_id","wg","period"))
examiner_wg <- drop_na(examiner_wg)

rm(examiner_wg_napp)
```

### Get seniority at each period

Let’s assume a time period of *t*<sub>0</sub> = 2007 (the year we first
get senior examiners, according to our definition),
*t*<sub>1</sub> = 2008, *t*<sub>2</sub> = 2009, *t*<sub>3</sub> = 2010,
*t*<sub>4</sub> = 2011, *t*<sub>5</sub> = 2012, *t*<sub>6</sub> = 2013,
*t*<sub>7</sub> = 2014, *t*<sub>8</sub> = 2015

``` r
# Get tenure & state at each period
examiner_dates <- examiner_dates %>% 
  mutate(
    tenure_t0 = ifelse(as.duration(earliest_date %--% ymd("2007-01-01")) / dyears(1)<0,0,as.duration(earliest_date %--% ymd("2007-01-01"))/dyears(1)),
    tenure_t1 = ifelse(as.duration(earliest_date %--% ymd("2008-01-01")) / dyears(1)<0,0,as.duration(earliest_date %--% ymd("2008-01-01"))/dyears(1)),
    tenure_t2 = ifelse(as.duration(earliest_date %--% ymd("2009-01-01")) / dyears(1)<0,0,as.duration(earliest_date %--% ymd("2009-01-01"))/dyears(1)),
    tenure_t3 = ifelse(as.duration(earliest_date %--% ymd("2010-01-01")) / dyears(1)<0,0,as.duration(earliest_date %--% ymd("2010-01-01"))/dyears(1)),
    tenure_t4 = ifelse(as.duration(earliest_date %--% ymd("2011-01-01")) / dyears(1)<0,0,as.duration(earliest_date %--% ymd("2011-01-01"))/dyears(1)),
    tenure_t5 = ifelse(as.duration(earliest_date %--% ymd("2012-01-01")) / dyears(1)<0,0,as.duration(earliest_date %--% ymd("2012-01-01"))/dyears(1)),
    tenure_t6 = ifelse(as.duration(earliest_date %--% ymd("2013-01-01")) / dyears(1)<0,0,as.duration(earliest_date %--% ymd("2013-01-01"))/dyears(1)),
    tenure_t7 = ifelse(as.duration(earliest_date %--% ymd("2014-01-01")) / dyears(1)<0,0,as.duration(earliest_date %--% ymd("2014-01-01"))/dyears(1)),
    tenure_t8 = ifelse(as.duration(earliest_date %--% ymd("2015-01-01")) / dyears(1)<0,0,as.duration(earliest_date %--% ymd("2015-01-01"))/dyears(1)),

    t0_state = case_when(
      tenure_t0<6 & tenure_t0>0 ~ "Junior"  , # Jr; not those yet to be hired!
      tenure_t0>=6              ~ "Senior"  , # Sr
      TRUE                      ~ NA_character_ # not yet hired
    ),
    t1_state = case_when(
      latest_date<ymd("2008-12-31")        ~ "Exit",
      earliest_date>ymd("2007-01-01") 
        & earliest_date<ymd("2008-01-01") ~ "New hire",
      tenure_t1<6 & tenure_t1>0          ~ "Junior"  , # Jr; not those yet to be hired!
      tenure_t1>=6                       ~ "Senior"  , # Sr
      TRUE                               ~ NA_character_ # not yet hired
      ),
    t2_state = case_when(
      t1_state=="Exit"                   ~ NA_character_,
      latest_date<ymd("2009-12-31")        ~ "Exit",
      earliest_date>ymd("2008-01-01") 
        & earliest_date<ymd("2009-01-01") ~ "New hire",
      tenure_t2<6 & tenure_t2>0          ~ "Junior"  , # Jr; not those yet to be hired!
      tenure_t2>=6                       ~ "Senior"  , # Sr
      TRUE                               ~ NA_character_ # not yet hired or already exit in previous period
      ),
    t3_state = case_when(
      t1_state=="Exit"|t2_state=="Exit"                   ~ NA_character_,
      latest_date<ymd("2010-12-31")        ~ "Exit",
      earliest_date>ymd("2009-01-01") 
        & earliest_date<ymd("2010-01-01") ~ "New hire",
      tenure_t3<6 & tenure_t3>0          ~ "Junior"  , # Jr; not those yet to be hired!
      tenure_t3>=6                       ~ "Senior"  , # Sr
      TRUE                               ~ NA_character_ # not yet hired or already exit in previous period
      ),
    t4_state = case_when(
      t1_state=="Exit"|t2_state=="Exit"|t3_state=="Exit"                   ~ NA_character_,
      latest_date<ymd("2011-12-31")        ~ "Exit",
      earliest_date>ymd("2010-01-01") 
        & earliest_date<ymd("2011-01-01") ~ "New hire",
      tenure_t4<6 & tenure_t4>0          ~ "Junior"  , # Jr; not those yet to be hired!
      tenure_t4>=6                       ~ "Senior"  , # Sr
      TRUE                               ~ NA_character_ # not yet hired or already exit in previous period
      ),
    t5_state = case_when(
      t1_state=="Exit"|t2_state=="Exit"|t3_state=="Exit"|t4_state=="Exit"                   ~ NA_character_,
      latest_date<ymd("2012-12-31")        ~ "Exit",
      earliest_date>ymd("2011-01-01") 
        & earliest_date<ymd("2012-01-01") ~ "New hire",
      tenure_t5<6 & tenure_t5>0          ~ "Junior"  , # Jr; not those yet to be hired!
      tenure_t5>=6                       ~ "Senior"  , # Sr
      TRUE                               ~ NA_character_ # not yet hired or already exit in previous period
      ),
    t6_state = case_when(
      t1_state=="Exit"|t2_state=="Exit"|t3_state=="Exit"|t4_state=="Exit"|t5_state=="Exit"                   ~ NA_character_,
      latest_date<ymd("2013-12-31")        ~ "Exit",
      earliest_date>ymd("2012-01-01") 
        & earliest_date<ymd("2013-01-01") ~ "New hire",
      tenure_t6<6 & tenure_t6>0          ~ "Junior"  , # Jr; not those yet to be hired!
      tenure_t6>=6                       ~ "Senior"  , # Sr
      TRUE                               ~ NA_character_ # not yet hired or already exit in previous period
      ),
    t7_state = case_when(
      t1_state=="Exit"|t2_state=="Exit"|t3_state=="Exit"|t4_state=="Exit"|t5_state=="Exit"|t6_state=="Exit"                   ~ NA_character_,
      latest_date<ymd("2014-12-31")        ~ "Exit",
      earliest_date>ymd("2013-01-01") 
        & earliest_date<ymd("2014-01-01") ~ "New hire",
      tenure_t7<6 & tenure_t7>0          ~ "Junior"  , # Jr; not those yet to be hired!
      tenure_t7>=6                       ~ "Senior"  , # Sr
      TRUE                               ~ NA_character_ # not yet hired or already exit in previous period
      ),
    t8_state = case_when(
      t1_state=="Exit"|t2_state=="Exit"|t3_state=="Exit"|t4_state=="Exit"|t5_state=="Exit"|t6_state=="Exit"|t7_state=="Exit"                   ~ NA_character_,
      latest_date<ymd("2015-12-31")        ~ "Exit",
      earliest_date>ymd("2014-01-01") 
        & earliest_date<ymd("2015-01-01") ~ "New hire",
      tenure_t8<6 & tenure_t8>0          ~ "Junior"  , # Jr; not those yet to be hired!
      tenure_t8>=6                       ~ "Senior"  , # Sr
      TRUE                               ~ NA_character_ # not yet hired or already exit in previous period
      )
    )

examiner_dates <- examiner_dates %>% 
  select(examiner_id, t0_state, t1_state, t2_state, t3_state, t4_state, t5_state, t6_state, t7_state, t8_state)

## plot seniority
library(ggplot2)
library(scales)  
```

    ## 
    ## Attaching package: 'scales'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     discard

    ## The following object is masked from 'package:readr':
    ## 
    ##     col_factor

``` r
library(gridExtra)
```

    ## 
    ## Attaching package: 'gridExtra'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

``` r
plot1 <- ggplot(examiner_dates, aes(factor(t1_state, levels = c("New hire", "Junior", "Senior", "Exit", NA)))) + 
          geom_bar(aes(y = (..count..)/sum(..count..))) + 
          scale_y_continuous(labels=scales::percent) +
          ylab("Relative Frequencies") +
          ggtitle("Seniority distribution for USPTO at t1")
plot1
```

![](final_report_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

Joining back to the applications dates data.

``` r
application_dates <- application_dates %>% 
  left_join(examiner_dates, by = "examiner_id")
rm(examiner_dates)
rm(plot1)
#gc()
```

### Generate examiner panel dataset

``` r
# compute average application processing time

cols <- c("examiner_id","period", "wg", "examiner_art_unit","gender", "race", "tenure_days",
          "t0_state","t1_state","t2_state","t3_state","t4_state","t5_state","t6_state","t7_state","t8_state")

examiners <- application_dates %>%
    group_by(across(all_of(cols))) %>%
    dplyr::summarize(mean_app_proc_time = mean(app_proc_time, na.rm=TRUE), n_app = n()) %>%
    drop_na()
```

    ## `summarise()` has grouped output by 'examiner_id', 'period', 'wg', 'examiner_art_unit', 'gender', 'race', 'tenure_days', 't0_state', 't1_state', 't2_state', 't3_state', 't4_state', 't5_state', 't6_state', 't7_state'. You can override using the `.groups` argument.

``` r
head(data.frame(examiners))
```

    ##   examiner_id period   wg examiner_art_unit gender  race tenure_days t0_state
    ## 1       59012     t0 1710              1716   male white        4013   Junior
    ## 2       59012     t0 1710              1717   male white        4013   Junior
    ## 3       59012     t1 1710              1716   male white        4013   Junior
    ## 4       59012     t1 1710              1717   male white        4013   Junior
    ## 5       59012     t3 1710              1716   male white        4013   Junior
    ## 6       59056     t0 2120              2124   male Asian        6268   Senior
    ##   t1_state t2_state t3_state t4_state t5_state t6_state t7_state t8_state
    ## 1   Junior   Junior   Junior   Senior   Senior   Senior   Senior     Exit
    ## 2   Junior   Junior   Junior   Senior   Senior   Senior   Senior     Exit
    ## 3   Junior   Junior   Junior   Senior   Senior   Senior   Senior     Exit
    ## 4   Junior   Junior   Junior   Senior   Senior   Senior   Senior     Exit
    ## 5   Junior   Junior   Junior   Senior   Senior   Senior   Senior     Exit
    ## 6   Senior   Senior   Senior   Senior   Senior   Senior   Senior   Senior
    ##   mean_app_proc_time n_app
    ## 1      1232.312 days    32
    ## 2      1379.429 days    14
    ## 3      1003.286 days     7
    ## 4      1131.000 days    10
    ## 5       273.000 days     1
    ## 6      1740.000 days     1

#### Create panel dataset of examiners for analysis

``` r
# subset examiners for time period t1 for our analysis, as advice dates are all in 2008 
examiner_aus <- data.frame(examiners) %>%
    filter(period == "t1") %>% 
    #filter(wg == 2450 | wg == 2480) %>%
    select(wg, examiner_art_unit, examiner_id, gender, race, t1_state, tenure_days, mean_app_proc_time, n_app) %>%
    distinct(examiner_id, .keep_all=TRUE) %>% 
    drop_na() 

head(examiner_aus) #2591
```

    ##     wg examiner_art_unit examiner_id gender  race t1_state tenure_days
    ## 1 1710              1716       59012   male white   Junior        4013
    ## 2 2120              2124       59056   male Asian   Senior        6268
    ## 3 2450              2455       59130   male Asian   Senior        6323
    ## 4 1780              1787       59133   male white   Junior        4573
    ## 5 2160              2169       59141 female Asian   Junior        4582
    ## 6 2160              2162       59181 female black   Senior        6331
    ##   mean_app_proc_time n_app
    ## 1     1003.2857 days     7
    ## 2     1445.5000 days     2
    ## 3      998.4737 days    38
    ## 4     1934.8065 days    31
    ## 5     1839.5556 days    18
    ## 6     1324.6038 days    53

### Compute centrality of examiners

``` r
# separate from edges examiners seek and give advice
edges_aus <- edges %>%
  filter(ego_examiner_id %in% examiner_aus$examiner_id) %>%
  filter(alter_examiner_id %in% examiner_aus$examiner_id) %>%
  drop_na() #8824

# merge work group information
network <- left_join(edges_aus, examiner_aus, by = c("ego_examiner_id" = "examiner_id"))
colnames(network)[5] <- "ego_examiner_wg"
colnames(network)[6] <- "ego_examiner_au"
colnames(network)[7] <- "ego_examiner_gender"
colnames(network)[8] <- "ego_examiner_race"
colnames(network)[9] <- "ego_examiner_t1_state"
colnames(network)[10] <- "ego_examiner_tenure"
colnames(network)[11] <- "ego_examiner_appprooctime"
colnames(network)[12] <- "ego_examiner_napp"
#network <- subset(network, select = -c(period))
network <- left_join(network, examiner_aus, by = c("alter_examiner_id" = "examiner_id"))
colnames(network)[13] <- "alter_examiner_wg"
colnames(network)[14] <- "alter_examiner_au"
colnames(network)[15] <- "alter_examiner_gender"
colnames(network)[16] <- "alter_examiner_race"
colnames(network)[17] <- "alter_examiner_t1_state"
#colnames(network)[18] <- "alter_examiner_tenure"
colnames(network)[19] <- "alter_examiner_appprooctime"
colnames(network)[20] <- "alter_examiner_napp"
#network <- subset(network, select = -c(period))

head(network)
```

    ## # A tibble: 6 × 20
    ##   application_number advice_date ego_examiner_id alter_examiner_… ego_examiner_wg
    ##   <chr>              <date>                <dbl>            <dbl>           <dbl>
    ## 1 09402488           2008-11-17            84356            63519            1650
    ## 2 09445135           2008-08-21            92953            91818            2420
    ## 3 09484331           2008-02-07            72253            61519            1630
    ## 4 09484331           2008-02-07            72253            72253            1630
    ## 5 09489652           2008-01-10            67078            75772            2190
    ## 6 09489652           2008-01-10            67078            97328            2190
    ## # … with 15 more variables: ego_examiner_au <dbl>, ego_examiner_gender <chr>,
    ## #   ego_examiner_race <chr>, ego_examiner_t1_state <chr>,
    ## #   ego_examiner_tenure <dbl>, ego_examiner_appprooctime <drtn>,
    ## #   ego_examiner_napp <int>, alter_examiner_wg <dbl>, alter_examiner_au <dbl>,
    ## #   alter_examiner_gender <chr>, alter_examiner_race <chr>,
    ## #   alter_examiner_t1_state <chr>, tenure_days <dbl>,
    ## #   alter_examiner_appprooctime <drtn>, alter_examiner_napp <int>

``` r
# Visualize the advice seeking volume by examiner seniority in period t1
  network %>% 
  group_by(ego_examiner_t1_state, alter_examiner_t1_state) %>% 
  dplyr::summarise(count = n())
```

    ## `summarise()` has grouped output by 'ego_examiner_t1_state'. You can override using the `.groups` argument.

    ## # A tibble: 4 × 3
    ## # Groups:   ego_examiner_t1_state [2]
    ##   ego_examiner_t1_state alter_examiner_t1_state count
    ##   <chr>                 <chr>                   <int>
    ## 1 Junior                Junior                    827
    ## 2 Junior                Senior                   3862
    ## 3 Senior                Junior                    373
    ## 4 Senior                Senior                   3762

There are more junior seeking advice from senior than peer advice
seeking (junior to junior, senior to senior). It is the fewest for
senior to seek advice from junior.

``` r
# create edge list
edge_list <- select(network, c("ego_examiner_id","alter_examiner_id")) #8824

# create node list
ego <- select(network, c("ego_examiner_id","ego_examiner_wg")) %>%
    dplyr::rename(id=ego_examiner_id, wg=ego_examiner_wg)
alter <- select(network, c("alter_examiner_id","alter_examiner_wg")) %>%
    dplyr::rename(id=alter_examiner_id, wg=alter_examiner_wg)
nodes <- rbind(ego, alter) %>%
  select(id) %>%
  distinct() %>%
  drop_na() #1447

# create advice net
library(igraph)
```

    ## 
    ## Attaching package: 'igraph'

    ## The following objects are masked from 'package:lubridate':
    ## 
    ##     %--%, union

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     as_data_frame, groups, union

    ## The following objects are masked from 'package:purrr':
    ## 
    ##     compose, simplify

    ## The following object is masked from 'package:tidyr':
    ## 
    ##     crossing

    ## The following object is masked from 'package:tibble':
    ## 
    ##     as_data_frame

    ## The following objects are masked from 'package:stats':
    ## 
    ##     decompose, spectrum

    ## The following object is masked from 'package:base':
    ## 
    ##     union

``` r
advice_net = graph_from_data_frame(d=edge_list, vertices=nodes, directed=TRUE)
```

``` r
# calculate Degree Centrality, a measure for a node in a network is just its degree, the number of edges connected to it. 
V(advice_net)$dc <- degree(advice_net)
# calculate Betweenness Centrality, which measures the extent to which a node lies on paths between other nodes.
V(advice_net)$bc <- betweenness(advice_net)
# calculate Eigenvector Centrality, which awards a number of points proportional to the centrality scores of the neighbors
V(advice_net)$ec <- evcent(advice_net)$vector
V(advice_net)$cc <- closeness(advice_net) # dropped since closeness centrality is not well-defined for disconnected graphs
```

    ## Warning in closeness(advice_net): At centrality.c:2874 :closeness centrality is
    ## not well-defined for disconnected graphs

``` r
# combine the centrality scores
centrality <- data.frame(cbind(nodes$id, V(advice_net)$dc, V(advice_net)$bc, V(advice_net)$ec, V(advice_net)$cc)) 
colnames(centrality)[1] <- "examiner_id"
colnames(centrality)[2] <- "degree_centrality"
colnames(centrality)[3] <- "betweenness_centrality"
colnames(centrality)[4] <- "eigenvector_centrality"
colnames(centrality)[5] <- "closeness_centrality"
head(centrality)
```

    ##   examiner_id degree_centrality betweenness_centrality eigenvector_centrality
    ## 1       84356                17             22.0000000           3.117146e-10
    ## 2       92953                 1              0.0000000           0.000000e+00
    ## 3       72253                28             94.0000000           7.153516e-11
    ## 4       67078                 2              0.0000000           1.177571e-08
    ## 5       91688                12              0.7936508           5.901232e-10
    ## 6       61797                25              0.0000000           4.146497e-07
    ##   closeness_centrality
    ## 1         4.802518e-07
    ## 2         4.785900e-07
    ## 3         4.795848e-07
    ## 4         4.795853e-07
    ## 5         4.782593e-07
    ## 6         5.569789e-07

``` r
# merge centrality to examiners
examiner_joined <- left_join(examiner_aus, centrality, by = c("examiner_id" = "examiner_id"))
examiner_joined <- examiner_joined %>%
  drop_na(degree_centrality)
head(examiner_joined) #1447
```

    ##     wg examiner_art_unit examiner_id gender  race t1_state tenure_days
    ## 1 2120              2124       59056   male Asian   Senior        6268
    ## 2 2160              2169       59141 female Asian   Junior        4582
    ## 3 2160              2162       59181 female black   Senior        6331
    ## 4 1640              1644       59211   male white   Senior        6332
    ## 5 1730              1734       59227 female white   Senior        6349
    ## 6 1650              1652       59236 female white   Senior        6331
    ##   mean_app_proc_time n_app degree_centrality betweenness_centrality
    ## 1      1445.500 days     2                 9                      0
    ## 2      1839.556 days    18                 7                      0
    ## 3      1324.604 days    53                17                      0
    ## 4      1044.125 days    56                21                      1
    ## 5      1270.418 days    91                 4                      0
    ## 6      1369.680 days    50                 2                      0
    ##   eigenvector_centrality closeness_centrality
    ## 1           1.951991e-07         4.779288e-07
    ## 2           1.221788e-06         4.785903e-07
    ## 3           6.445086e-05         4.779288e-07
    ## 4           2.212934e-10         4.782593e-07
    ## 5           5.042804e-13         4.779288e-07
    ## 6           4.941852e-12         4.779288e-07

``` r
# housekeeping
rm(examiner_wg)
rm(alter)
rm(ego)
rm(examiner_aus)
rm(edges_aus)
rm(edge_list)
rm(edges)
rm(nodes)
rm(centrality)
```

### Work groups selection (applicable for analysis zoom-in)

``` r
# select the workgroups under the same technology centre with most examiners at t1 for our analysis, as advice dates are all in 2008
examiner_joined %>% 
  #dplyr::filter(period == "t1") %>% 
  count("wg") %>% 
  arrange(desc(freq)) %>%
  head(4)
```

    ##     wg freq
    ## 1 2440   84
    ## 2 1780   79
    ## 3 1640   71
    ## 4 1770   66

Hence, we’re selecting work groups 1780 and 1770 under the same
technology centre 1700 for further analysis.

``` r
examiner_joined_1780 = examiner_joined[examiner_joined$wg==1780,]
examiner_joined_1770 = examiner_joined[examiner_joined$wg==1770,]

examiner_joined_2wg <- examiner_joined %>%
  filter(wg == 1780 | wg == 1770)
```

# 3. Analysis and Results

### 3.1 Organization and social factors associated with the length of patent application prosecution

#### 3.1.1 Impacts of examiners’ gender

#### 3.1.2 Impacts of examiners’ seniority

### 3.2 The role of race and ethnicity in the processes described

#### 3.2.1 Impacts of examiners’ race

# 4. Conclusions and Recommendations
