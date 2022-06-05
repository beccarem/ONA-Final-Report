Final Report
================
Chelsea, Rebecca, Diwei, Dany

# 1. Introduction

The U.S. Patent and Trademark Office (USPTO) is the agency within the
U.S. Chamber of Commerce that issues patents and trademarks. It is one
in few organizations around the world to guarantee the intellectual
property rights or inventors. Although the organization plays an
important role in the capitalist society, it is now faced with many
challenges and criticisms that it needs to address.

One of the challenges is the time it takes examiners to process an
application. As the agency is fully funded by patent applications fees,
the focus should be to ensure a satisfactory “customer experience.”
However, when examiners take a long time to process applications, this
can be a frustrating experience for applicants. USPTO is now taking
action to improve its entire process by identifying the causes of
application backlogs.

Our goal is to understand what causes delays in processing times, which
characteristics makes examiners more efficient in their work and how can
network analysis solve the organizational difficulties that USPTO is
facing. We will focus on determining the organizational and social
factors associated with the length of patent application prosecution. We
will look at how examiner’s demographics (gender, race, tenure) are
related to their application processing time. We will also look at the
social advice network within the organization plays a role in improving
the patent application times and outcomes.

# 2. Methodology

1.  Data Pre-processing

<!-- -->

1.  Data Engineering
2.  Data Exploration
3.  Data Cleaning

<!-- -->

2.  Examining Gender Effect
3.  Examining Race Effect
4.  Examining Tenure Effect

# 3. Analysis and Results

Load the applications data from ‘app\_data\_sample.parquet’ and edges
data from ‘edges\_sample.csv’

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

Now let’s use function `gender()` as shown in the example for the
package to attach a gender and probability to each name and put the
results into the table `examiner_names_gender`

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

### Guess the examiner’s race

We’ll now use package `wru` to estimate likely race of an examiner. Just
like with gender, we’ll get a list of unique names first, only now we
are using surnames.

We’ll follow the instructions for the package outlined here
<https://github.com/kosukeimai/wru>.

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
    ## 10 KIM          0.0252  0.00390  0.00650  0.945     0.0198      0.945 asian
    ## # … with 3,796 more rows

Let’s join the data back to the applications table.

### Examiner’s tenure

To figure out the timespan for which we observe each examiner in the
applications data, let’s find the first and the last observed date for
each examiner. We’ll first get examiner IDs and application dates in a
separate table, for ease of manipulation. We’ll keep examiner ID (the
field `examiner_id`), and earliest and latest dates for each application
(`filing_date` and `appl_status_date` respectively). We’ll use functions
in package `lubridate` to work with date and time values.

The dates look inconsistent in terms of formatting. Let’s make them
consistent. We’ll create new variables `start_date` and `end_date`.

Let’s now identify the earliest and the latest date for each examiner
and calculate the difference in days, which is their tenure in the
organization.

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

### Create application processing time variable

Compute the final decision date as either abandon date or patent issue
date.

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

![](final_report_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

![](final_report_files/figure-gfm/unnamed-chunk-5-1.png)<!-- --> Filter
out negative and outlying application processing time. Outliers are
removed successfully.

### Get work group from art unit

### Get seniority at each period

Let’s assume a time period of *t*<sub>0</sub> = 2007 (the year we first
get senior examiners, according to our definition),
*t*<sub>1</sub> = 2008, *t*<sub>2</sub> = 2009, *t*<sub>3</sub> = 2010,
*t*<sub>4</sub> = 2011, *t*<sub>5</sub> = 2012, *t*<sub>6</sub> = 2013,
*t*<sub>7</sub> = 2014, *t*<sub>8</sub> = 2015

![](final_report_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

Joining back to the applications dates data.

### Generate examiner panel dataset

Compute average application processing time

    ## `summarise()` has grouped output by 'examiner_id', 'period', 'wg', 'examiner_art_unit', 'gender', 'race', 'tenure_days', 't0_state', 't1_state', 't2_state', 't3_state', 't4_state', 't5_state', 't6_state', 't7_state'. You can override using the `.groups` argument.

    ##   examiner_id period   wg examiner_art_unit gender  race tenure_days t0_state
    ## 1       59012     t0 1710              1716   male white        4013   Junior
    ## 2       59012     t0 1710              1717   male white        4013   Junior
    ## 3       59012     t1 1710              1716   male white        4013   Junior
    ## 4       59012     t1 1710              1717   male white        4013   Junior
    ## 5       59012     t3 1710              1716   male white        4013   Junior
    ## 6       59056     t0 2120              2124   male asian        6268   Senior
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

    ##     wg examiner_art_unit examiner_id gender  race t1_state tenure_days
    ## 1 1710              1716       59012   male white   Junior        4013
    ## 2 2120              2124       59056   male asian   Senior        6268
    ## 3 2450              2455       59130   male asian   Senior        6323
    ## 4 1780              1787       59133   male white   Junior        4573
    ## 5 2160              2169       59141 female asian   Junior        4582
    ## 6 2160              2162       59181 female black   Senior        6331
    ##   mean_app_proc_time n_app
    ## 1     1003.2857 days     7
    ## 2     1445.5000 days     2
    ## 3      998.4737 days    38
    ## 4     1934.8065 days    31
    ## 5     1839.5556 days    18
    ## 6     1324.6038 days    53

### Compute centrality of examiners

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

### Work groups selection (applicable for analysis zoom-in)

    ##     wg freq
    ## 1 2440   84
    ## 2 1780   79
    ## 3 1640   71
    ## 4 1770   66

Hence, we’re selecting work groups 1780 and 1770 under the same
technology centre 1700 for further analysis.

### 3.1 Organization and social factors associated with the length of patent application prosecution

Before we begin, let’s take a look at the distribution of application
processing time for the selected examiners.
![](final_report_files/figure-gfm/unnamed-chunk-22-1.png)<!-- -->

We can see that the two selected work groups under the same technology
centre have lower application processing time than USTPO company as a
whole. Comparatively, 1780 demonstrates more similar distribution as
company-wide, while work group 1770 has even shorter application proc
time in general but slightly more outliers towards the higher end. Given
the low sample size (79 examiners for 1780 and 66 examiners for 1770),
the data limitation shall be acknowledged.

Also we would like to understand the network for the two selected work
groups.

![](final_report_files/figure-gfm/unnamed-chunk-23-1.png)<!-- -->

#### 3.1.1 Impacts of centrality

As a consulting team, we would like to analyze how examiners’ gender is
associated with the length of the patent application processing time.
Here we would focus on both the USPTO organizational level and selected
work groups levels in t1 period.

Firstly, we run 4 linear regression models on organizational level to
understand the effect of each centrality measure to the application
process time separately.

    ## 
    ## Please cite as:

    ##  Hlavac, Marek (2018). stargazer: Well-Formatted Regression and Summary Statistics Tables.

    ##  R package version 5.2.2. https://CRAN.R-project.org/package=stargazer

    ## 
    ## Impacts of Each Centrality Measure to Application Processing Time (USPTO level)
    ## ==========================================================================================
    ##                                                    Dependent variable:                    
    ##                                 ----------------------------------------------------------
    ##                                               as.numeric(mean_app_proc_time)              
    ##                                     (1)          (2)          (3)              (4)        
    ## ------------------------------------------------------------------------------------------
    ## degree_centrality                  -0.101                                                 
    ##                                   (0.629)                                                 
    ##                                                                                           
    ## betweenness_centrality                          -0.013                                    
    ##                                                (0.014)                                    
    ##                                                                                           
    ## eigenvector_centrality                                      272.252                       
    ##                                                            (467.882)                      
    ##                                                                                           
    ## closeness_centrality                                                   1,312,375,648.000**
    ##                                                                         (542,030,078.000) 
    ##                                                                                           
    ## Constant                        1,365.962*** 1,366.274*** 1,364.533***     726.228***     
    ##                                   (14.502)     (12.400)     (12.308)        (263.998)     
    ##                                                                                           
    ## ------------------------------------------------------------------------------------------
    ## Observations                       1,447        1,447        1,447            1,447       
    ## R2                                0.00002       0.001        0.0002           0.004       
    ## Adjusted R2                        -0.001      -0.00003     -0.0005           0.003       
    ## Residual Std. Error (df = 1445)   468.042      467.891      467.992          467.100      
    ## F Statistic (df = 1; 1445)         0.026        0.960        0.339           5.862**      
    ## ==========================================================================================
    ## Note:                                                          *p<0.1; **p<0.05; ***p<0.01

Both degree centrality and betweenness centrality have a negative
relation to the mean application processing time. Adding one more unit
in degree centrality and betweenness centrality subtract, on average,
mean application processing time by 0.101 days and 0.013 days
respectively, if holding everything else equal. The higher these
centrality scores, the faster the application processing.

Both eigenvector centrality and closesness centrality (ranged from 0 to
1 ) have a positive relation to the mean application processing time.
Adding 0.1 more unit in eigenvector centrality adds mean application
processing time by 27.2 days and adding 0.1 more unit in eigenvector
centrality adds mean application processing time by 1.3e08 days, if
holding everything else equal. The higher these centrality scores, the
slower the application processing. This means having relationship with
examiners who have high scores would take longer processing time,
potentially due to more workload assigned, and the more distant an
examiner is with other examiners, the more the longer the processing
time, potentially due to lack of peer support.

Now, let’s take a look at the correlation of the 4 centrality measures
and run a regression to understand the effect of the centrality measure
to the application process time together.

![](final_report_files/figure-gfm/unnamed-chunk-25-1.png)<!-- --> From
the correlation matrix we can see that the target variable
app\_proc\_time has no strong correlation with other numeric variables.
The centrality measures have strong correlation with each other -
relatively stronger for degree centrality with all other measures and
slightly stronger correlation between betweenness centrality and
closeness centrality.

Then, we look into all measures in one linear regression model and
observed consistent results - an increase in degree centrality and
betweenness centrality reduces application processing time while an in
crease in eigenvector centrality and closeness centrality increases
application processing time.

    ## 
    ## Impacts of Centrality Measure Separately and Combined to Application Processing Time (USPTO level)
    ## ==================================================================================================================================
    ##                                                                    Dependent variable:                                            
    ##                        -----------------------------------------------------------------------------------------------------------
    ##                                                              as.numeric(mean_app_proc_time)                                       
    ##                                (1)                  (2)                  (3)                   (4)                    (5)         
    ## ----------------------------------------------------------------------------------------------------------------------------------
    ## degree_centrality             -0.101                                                                                -0.223        
    ##                              (0.629)                                                                                (0.677)       
    ##                                                                                                                                   
    ## betweenness_centrality                             -0.013                                                           -0.023        
    ##                                                   (0.014)                                                           (0.015)       
    ##                                                                                                                                   
    ## eigenvector_centrality                                                 272.252                                      317.346       
    ##                                                                       (467.882)                                    (474.120)      
    ##                                                                                                                                   
    ## closeness_centrality                                                                   1,312,375,648.000**   1,591,292,250.000*** 
    ##                                                                                         (542,030,078.000)      (567,266,041.000)  
    ##                                                                                                                                   
    ## Constant                   1,365.962***         1,366.274***         1,364.533***           726.228***             595.606**      
    ##                              (14.502)             (12.400)             (12.308)             (263.998)              (274.896)      
    ##                                                                                                                                   
    ## ----------------------------------------------------------------------------------------------------------------------------------
    ## Observations                  1,447                1,447                1,447                 1,447                  1,447        
    ## R2                           0.00002               0.001                0.0002                0.004                  0.006        
    ## Adjusted R2                   -0.001              -0.00003             -0.0005                0.003                  0.004        
    ## Residual Std. Error    468.042 (df = 1445)  467.891 (df = 1445)  467.992 (df = 1445)   467.100 (df = 1445)    467.048 (df = 1442) 
    ## F Statistic            0.026 (df = 1; 1445) 0.960 (df = 1; 1445) 0.339 (df = 1; 1445) 5.862** (df = 1; 1445) 2.296* (df = 4; 1442)
    ## ==================================================================================================================================
    ## Note:                                                                                                  *p<0.1; **p<0.05; ***p<0.01

Next, we will repeat the steps for the two selected work groups.

    ## 
    ## Impacts of Centrality Measure Separately and Combined to Application Processing Time (2wg level)
    ## ==========================================================================================================================
    ##                                                                Dependent variable:                                        
    ##                        ---------------------------------------------------------------------------------------------------
    ##                                                          as.numeric(mean_app_proc_time)                                   
    ##                                (1)                 (2)                 (3)                 (4)                 (5)        
    ## --------------------------------------------------------------------------------------------------------------------------
    ## degree_centrality            -2.070                                                                          -1.844       
    ##                              (1.406)                                                                         (1.475)      
    ##                                                                                                                           
    ## betweenness_centrality                            0.053                                                       0.129       
    ##                                                  (1.061)                                                     (1.066)      
    ##                                                                                                                           
    ## eigenvector_centrality                                            -761,768.900                            -733,900.600    
    ##                                                                  (2,758,323.000)                         (2,765,540.000)  
    ##                                                                                                                           
    ## closeness_centrality                                                               -1,763,717,185.000  -1,136,894,823.000 
    ##                                                                                    (1,844,022,311.000) (1,922,483,285.000)
    ##                                                                                                                           
    ## Constant                  1,313.860***        1,289.150***        1,290.221***         2,140.211**         1,859.320**    
    ##                             (33.791)            (30.620)            (29.793)            (889.912)           (923.550)     
    ##                                                                                                                           
    ## --------------------------------------------------------------------------------------------------------------------------
    ## Observations                   145                 145                 145                 145                 145        
    ## R2                            0.015              0.00002              0.001               0.006               0.018       
    ## Adjusted R2                   0.008              -0.007              -0.006              -0.001              -0.010       
    ## Residual Std. Error    354.887 (df = 143)  357.564 (df = 143)  357.471 (df = 143)  356.429 (df = 143)  358.119 (df = 140) 
    ## F Statistic            2.168 (df = 1; 143) 0.003 (df = 1; 143) 0.076 (df = 1; 143) 0.915 (df = 1; 143) 0.640 (df = 4; 140)
    ## ==========================================================================================================================
    ## Note:                                                                                          *p<0.1; **p<0.05; ***p<0.01

Unlike company level, all centrality measures except betweenness
centrality reduce application process time for the two selected work
groups. This finding is consistent across running regression separately
and combined.

Degree centrality, eigenvector centrality and closeness centrality have
a negative relation to the mean application processing time. Adding one
more unit of degree centrality subtract, on average, mean application
processing time by 1.84 days and adding 0.1 unit of eigenvector
centrality and closeness centrality add 7.33e04 days and 1.14e08
respectively, if holding everything else equal. The higher these
centrality scores, the faster the application processing.

Only betweenness centrality have a positive relation to the mean
application processing time. Adding 0.1 more unit in eigenvector
centrality adds mean application processing time by 0.129 days, if
holding everything else equal. This means having relationship with
examiners who have the greatest influence over the flow of information
would take longer processing time, potentially due to some bottleneck or
centralized review needed within the work groups.

Overall, the effect of centrality is greater for work groups 2450 and
2480 than in the entire USPTO organization. This is potentially due to
the nature of applications that require more communications,
collaborations and advice seeking in specific domain subjects. This is
in line with the work group shortlisting approach taken in our
methodology.

#### 3.1.2 Impacts of examiners’ gender

Now that we have a general idea on the relationship of centrality
measures to application processing, we can overlay different dimensions
on top and evaluate if there is different level of impacts. We will
focus on the 2 work groups 1780 and 1770 shortlisted and begin with
examiner’s gender in t1 period.

    ## 
    ## Impacts of Centrality Measure Separately and Combined to Application Processing Time (2wg level, male)
    ## ========================================================================================================================
    ##                                                               Dependent variable:                                       
    ##                        -------------------------------------------------------------------------------------------------
    ##                                                         as.numeric(mean_app_proc_time)                                  
    ##                               (1)                (2)                 (3)                 (4)                 (5)        
    ## ------------------------------------------------------------------------------------------------------------------------
    ## degree_centrality            -2.165                                                                        -1.894       
    ##                             (1.558)                                                                        (1.697)      
    ##                                                                                                                         
    ## betweenness_centrality                          0.684                                                       0.859       
    ##                                                (1.182)                                                     (1.194)      
    ##                                                                                                                         
    ## eigenvector_centrality                                       -2,174,175,763.000                      -1,507,154,670.000 
    ##                                                              (2,231,984,954.000)                     (2,345,267,960.000)
    ##                                                                                                                         
    ## closeness_centrality                                                             -1,310,550,071.000   -388,030,633.000  
    ##                                                                                  (2,256,989,357.000) (2,349,671,706.000)
    ##                                                                                                                         
    ## Constant                  1,289.814***       1,261.027***       1,273.747***         1,897.334*           1,473.949     
    ##                             (38.318)           (35.235)           (35.298)           (1,088.681)         (1,128.982)    
    ##                                                                                                                         
    ## ------------------------------------------------------------------------------------------------------------------------
    ## Observations                   95                 95                 95                  95                  95         
    ## R2                           0.020              0.004               0.010               0.004               0.030       
    ## Adjusted R2                  0.010              -0.007             -0.001              -0.007              -0.013       
    ## Residual Std. Error    332.245 (df = 93)  335.075 (df = 93)   333.978 (df = 93)   335.070 (df = 93)   336.005 (df = 90) 
    ## F Statistic            1.931 (df = 1; 93) 0.334 (df = 1; 93) 0.949 (df = 1; 93)  0.337 (df = 1; 93)  0.705 (df = 4; 90) 
    ## ========================================================================================================================
    ## Note:                                                                                        *p<0.1; **p<0.05; ***p<0.01

    ## 
    ## Impacts of Centrality Measure Separately and Combined to Application Processing Time (2wg level, female)
    ## =======================================================================================================================
    ##                                                              Dependent variable:                                       
    ##                        ------------------------------------------------------------------------------------------------
    ##                                                         as.numeric(mean_app_proc_time)                                 
    ##                               (1)                (2)                (3)                 (4)                 (5)        
    ## -----------------------------------------------------------------------------------------------------------------------
    ## degree_centrality            -2.031                                                                       -1.487       
    ##                             (2.944)                                                                       (3.134)      
    ##                                                                                                                        
    ## betweenness_centrality                          -1.581                                                    -1.549       
    ##                                                (2.183)                                                    (2.237)      
    ##                                                                                                                        
    ## eigenvector_centrality                                         -1,133,026.000                         -1,271,400.000   
    ##                                                               (3,096,228.000)                         (3,159,762.000)  
    ##                                                                                                                        
    ## closeness_centrality                                                            -2,506,844,397.000  -1,992,146,951.000 
    ##                                                                                 (3,205,125,562.000) (3,422,545,629.000)
    ##                                                                                                                        
    ## Constant                  1,361.097***       1,347.836***       1,338.181***         2,545.286           2,331.557     
    ##                             (67.569)           (58.790)           (56.951)          (1,548.189)         (1,641.995)    
    ##                                                                                                                        
    ## -----------------------------------------------------------------------------------------------------------------------
    ## Observations                   50                 50                 50                 50                  50         
    ## R2                           0.010              0.011              0.003               0.013               0.031       
    ## Adjusted R2                  -0.011             -0.010             -0.018             -0.008              -0.055       
    ## Residual Std. Error    397.121 (df = 48)  396.922 (df = 48)  398.528 (df = 48)   396.564 (df = 48)   405.756 (df = 45) 
    ## F Statistic            0.476 (df = 1; 48) 0.524 (df = 1; 48) 0.134 (df = 1; 48) 0.612 (df = 1; 48)  0.359 (df = 4; 45) 
    ## =======================================================================================================================
    ## Note:                                                                                       *p<0.1; **p<0.05; ***p<0.01

It is interesting to observe that the degree centrality effect in
reducing application processing time is more in males (adding one more
unit subtracts application processing time by 1.89 days) than females
(subtracts 1.49 days) and the betweenness centrality effect adds
application processing for male but reduces application processing time
for female. This shows the different strengths and preferences on how to
get applications processed by gender. Male examiners are more good at
building cohesive network that all examiners know each other well, while
female examiners are more good at bridging network that they have close
examiners in different groups who don’t know each other well. Also,
bridging network seems to be more effective in reducing application
processing time for female examiners while it could be
counter-productive for male.

To better understand potential reasons and control for other
characteristics of examiner that might influence the relationship, let’s
take a look at the distribution of gender on company level and work
group level.

    ## Scale for 'y' is already present. Adding another scale for 'y', which will
    ## replace the existing scale.
    ## Scale for 'y' is already present. Adding another scale for 'y', which will
    ## replace the existing scale.
    ## Scale for 'y' is already present. Adding another scale for 'y', which will
    ## replace the existing scale.
    ## Scale for 'y' is already present. Adding another scale for 'y', which will
    ## replace the existing scale.

![](final_report_files/figure-gfm/unnamed-chunk-30-1.png)<!-- -->

It is observed that there is systematic bias in gender for USPTO and
selected work groups that there are more male examiners than female.
Among the 2 selected work group, 1780 has better gender balance and 1770
has worse gender balance when compared to company-level. Gender shall be
taken into consideration in the formulation of regression overall.

Let’s repeat running the regression with interaction terms of gender and
centrality. This time we’re going to take a more micro-view per each of
the selected work groups as their gender distributions are so different.

    ## 
    ## Impacts of Centrality Measure Separately and Combined to Application Processing Time (1780 wg level, centrality x gender)
    ## ===============================================================================================================================================
    ##                                                                                     Dependent variable:                                        
    ##                                              --------------------------------------------------------------------------------------------------
    ##                                                                                as.numeric(mean_app_proc_time)                                  
    ##                                                     (1)                 (2)                 (3)                 (4)                 (5)        
    ## -----------------------------------------------------------------------------------------------------------------------------------------------
    ## degree_centrality                                  -2.103                                                                         -0.261       
    ##                                                   (4.027)                                                                         (4.496)      
    ##                                                                                                                                                
    ## betweenness_centrality                                               -12.228**                                                   -12.012*      
    ##                                                                       (5.812)                                                     (6.140)      
    ##                                                                                                                                                
    ## eigenvector_centrality                                                                 -969,034.100                           -1,481,198.000   
    ##                                                                                       (2,896,574.000)                         (2,901,398.000)  
    ##                                                                                                                                                
    ## closeness_centrality                                                                                    -2,300,268,739.000   -895,906,878.000  
    ##                                                                                                         (3,038,678,695.000) (3,412,273,921.000)
    ##                                                                                                                                                
    ## as.factor(gender)male                             -81.692            -169.128*            -87.309           -1,156.557          -1,692.842     
    ##                                                  (102.450)           (87.792)            (86.849)           (2,069.662)         (2,394.764)    
    ##                                                                                                                                                
    ## degree_centrality:as.factor(gender)male            -0.818                                                                         -4.140       
    ##                                                   (4.621)                                                                         (5.870)      
    ##                                                                                                                                                
    ## betweenness_centrality:as.factor(gender)male                         13.354**                                                    13.349**      
    ##                                                                       (5.960)                                                     (6.291)      
    ##                                                                                                                                                
    ## eigenvector_centrality:as.factor(gender)male                                        -4,060,405,843.000                       1,923,445,209.000 
    ##                                                                                     (5,006,380,986.000)                     (7,685,871,745.000)
    ##                                                                                                                                                
    ## closeness_centrality:as.factor(gender)male                                                               2,185,261,518.000   3,245,876,917.000 
    ##                                                                                                         (4,273,146,757.000) (5,010,232,173.000)
    ##                                                                                                                                                
    ## Constant                                        1,339.215***       1,373.178***        1,317.188***          2,427.751           1,815.083     
    ##                                                   (80.164)           (68.066)            (64.610)           (1,473.418)         (1,627.677)    
    ##                                                                                                                                                
    ## -----------------------------------------------------------------------------------------------------------------------------------------------
    ## Observations                                         79                 79                  79                  79                  79         
    ## R2                                                 0.042               0.081               0.027               0.025               0.114       
    ## Adjusted R2                                        0.004               0.044              -0.011              -0.014              -0.001       
    ## Residual Std. Error                          368.157 (df = 75)   360.689 (df = 75)   370.976 (df = 75)   371.455 (df = 75)   369.063 (df = 69) 
    ## F Statistic                                  1.101 (df = 3; 75) 2.193* (df = 3; 75) 0.706 (df = 3; 75)  0.639 (df = 3; 75)  0.991 (df = 9; 69) 
    ## ===============================================================================================================================================
    ## Note:                                                                                                               *p<0.1; **p<0.05; ***p<0.01

Looking into work group 1780 which has better gender balance, the effect
of different centrality measures to mean application time separately in
the 1st to 4th regression is consistent with combining them together for
evaluation in the 5th regression. Increase in any of the four centrality
measures can reduce the application processing time. Being a male
examiner subtracts, on average, mean application processing time by 1693
days, if holding everything else equal. The interaction terms of gender
x centrality has shown that being a male examiner, the effect of degree
centrality is negative and the effect of betweenness centrality is
positive in relation to the mean application processing time - this
supports the robustness of our above finding per gender that male
examiners are more good at building cohesive network while female
examiners are more good at bridging network.

    ## 
    ## Impacts of Centrality Measure Separately and Combined to Application Processing Time (1770 wg level, centrality x gender)
    ## ===================================================================================================================================================
    ##                                                                                       Dependent variable:                                          
    ##                                              ------------------------------------------------------------------------------------------------------
    ##                                                                                  as.numeric(mean_app_proc_time)                                    
    ##                                                     (1)                (2)                  (3)                  (4)                   (5)         
    ## ---------------------------------------------------------------------------------------------------------------------------------------------------
    ## degree_centrality                                  -2.091                                                                            -2.348        
    ##                                                   (3.485)                                                                            (3.773)       
    ##                                                                                                                                                    
    ## betweenness_centrality                                                -0.439                                                         -1.255        
    ##                                                                      (2.057)                                                         (4.496)       
    ##                                                                                                                                                    
    ## eigenvector_centrality                                                              -48,691,658,828.000                        122,649,961,465.000 
    ##                                                                                    (298,629,786,168.000)                      (656,665,395,524.000)
    ##                                                                                                                                                    
    ## closeness_centrality                                                                                       -668,763,612.000    -3,166,592,296.000  
    ##                                                                                                          (47,488,018,473.000) (50,501,007,694.000) 
    ##                                                                                                                                                    
    ## as.factor(gender)male                             -96.222            -68.288              -71.332             1,176.078             -274.341       
    ##                                                  (112.294)          (104.892)            (108.328)           (22,841.610)         (24,305.100)     
    ##                                                                                                                                                    
    ## degree_centrality:as.factor(gender)male            1.652                                                                              2.003        
    ##                                                   (4.322)                                                                            (4.621)       
    ##                                                                                                                                                    
    ## betweenness_centrality:as.factor(gender)male                          -2.443                                                         -1.524        
    ##                                                                      (5.508)                                                         (6.972)       
    ##                                                                                                                                                    
    ## eigenvector_centrality:as.factor(gender)male                                        47,059,356,177.000                        -123,737,432,522.000 
    ##                                                                                    (298,641,591,923.000)                      (656,671,722,712.000)
    ##                                                                                                                                                    
    ## closeness_centrality:as.factor(gender)male                                                                -2,590,487,171.000     399,146,585.000   
    ##                                                                                                          (47,676,905,724.000) (50,710,133,578.000) 
    ##                                                                                                                                                    
    ## Constant                                        1,410.319***       1,387.850***        1,388.340***           1,701.823             2,931.854      
    ##                                                   (98.811)           (91.403)            (96.110)            (22,750.360)         (24,204.380)     
    ##                                                                                                                                                    
    ## ---------------------------------------------------------------------------------------------------------------------------------------------------
    ## Observations                                         66                 66                  66                    66                   66          
    ## R2                                                 0.014              0.014                0.015                0.018                 0.033        
    ## Adjusted R2                                        -0.033             -0.034              -0.033                -0.030               -0.123        
    ## Residual Std. Error                          345.167 (df = 62)  345.239 (df = 62)    345.126 (df = 62)    344.611 (df = 62)     359.773 (df = 56)  
    ## F Statistic                                  0.301 (df = 3; 62) 0.292 (df = 3; 62)  0.306 (df = 3; 62)    0.369 (df = 3; 62)   0.211 (df = 9; 56)  
    ## ===================================================================================================================================================
    ## Note:                                                                                                                   *p<0.1; **p<0.05; ***p<0.01

Looking into work group 1770 which has more severe gender imbalance
towards male, the overall findings are similar to work group 1780 - the
effect of different centrality measures to mean application time
separately in the 1st to 4th regression is consistent with combining
them together for evaluation in the 5th regression. Increase in any of
the four centrality measures can reduce the application processing time.
Being a male examiner subtracts, on average, mean application processing
time by 274 days, if holding everything else equal.

However, the interaction terms of gender x centrality has shown that
being a male examiner, the effect of degree centrality is positive and
the effect of betweenness centrality is negative in relation to the mean
application processing time - this can potentially be explained by the
hypothesis that even male examiners are more good at building cohesive
network it may be a cons than pros in a male dominating work group, and
that even female examiners are more good at bridging network they might
not perform as well when they are the minority. These warrant further
analysis beyond application processing time on how the gender
distributions impact examiners’ networking among different work groups.

#### 3.1.3 Impacts of examiners’ seniority

We are interested in how examiners’ seniority influences the application
process time. Here, we focus on the seniority state of examiners in t1
period.

Firstly, We fit a linear regression between the mean application
processing time and the centralities for all junior examiners.

Based on the summary, for junior examiners, an increase of 1 unit of
degree centrality, betweenness centrality, and eigen vector centrality,
means a decrease of process time by 2.76 days, 8.63 days and 1.32e+98
days respectively. Only the closeness centrality has positive
relationship with the processing time, which means the higher the
closeness centrality, the longer the processing time. Unfortunately, all
of the coefficients are not statistically significant.

Then, we fit the similar linear regression based on data points from
senior examiners to see if seniority impose some impacts.

Different from the juniors, for senior examiners, an increase of 1 unit
of degree centrality and eigen vector centrality means a decrease of
process time by 2.86 days (larger than the amount of change for juniors)
and 1.33e+08 days(smaller than the amount of change for juniors)
respectively. Contrastingly, an increase of 1 unit of betweenness
centrality and closeness centrality causes an increase of process time
by 6.14 days and 4.15 days respectively. Similarly, all of the
coefficients are not statistically significant.

In the next model, we include seniority as interaction term for each of
the centrality predictors.

    ## 
    ## Impacts of Seniority to Application Processing Time
    ## =================================================================================================
    ##                                                           Dependent variable:                    
    ##                                       -----------------------------------------------------------
    ##                                                     as.numeric(mean_app_proc_time)               
    ##                                               (1)                 (2)                 (3)        
    ## -------------------------------------------------------------------------------------------------
    ## t1_stateSenior                                                                      -9.602       
    ##                                                                                   (1,995.795)    
    ##                                                                                                  
    ## degree_centrality                           -2.761              -1.381              -2.761       
    ##                                             (2.509)             (1.904)             (2.377)      
    ##                                                                                                  
    ## betweenness_centrality                      -8.628               0.544              -8.628       
    ##                                            (13.578)             (1.055)            (12.864)      
    ##                                                                                                  
    ## eigenvector_centrality                -1,107,117,631.000     -368,935.000     -1,107,117,631.000 
    ##                                       (3,012,160,700.000)   (2,689,044.000)   (2,853,691,116.000)
    ##                                                                                                  
    ## closeness_centrality                  -1,324,153,448.000  -1,667,472,201.000  -1,324,153,448.000 
    ##                                       (2,884,571,627.000) (3,044,851,915.000) (2,732,814,496.000)
    ##                                                                                                  
    ## t1_stateSenior:degree_centrality                                                     1.380       
    ##                                                                                     (3.078)      
    ##                                                                                                  
    ## t1_stateSenior:betweenness_centrality                                                9.171       
    ##                                                                                    (12.909)      
    ##                                                                                                  
    ## t1_stateSenior:eigenvector_centrality                                          1,106,748,696.000 
    ##                                                                               (2,853,692,452.000)
    ##                                                                                                  
    ## t1_stateSenior:closeness_centrality                                            -343,318,753.000  
    ##                                                                               (4,152,505,479.000)
    ##                                                                                                  
    ## Constant                                   2,068.548           2,058.946           2,068.548     
    ##                                           (1,385.821)         (1,463.893)         (1,312.913)    
    ##                                                                                                  
    ## -------------------------------------------------------------------------------------------------
    ## Observations                                  47                  98                  145        
    ## R2                                           0.068               0.011               0.060       
    ## Adjusted R2                                 -0.021              -0.032              -0.003       
    ## Residual Std. Error                    376.629 (df = 42)   347.496 (df = 93)  356.815 (df = 135) 
    ## F Statistic                           0.769 (df = 4; 42)  0.252 (df = 4; 93)  0.956 (df = 9; 135)
    ## =================================================================================================
    ## Note:                                                                 *p<0.1; **p<0.05; ***p<0.01

According to this model, a junior examiner usually process applications
9.60 days longer than a senior examiner. Also, the same as the previous
junior model, an increase in any of the centrality predictor results in
shorter processing time for junior examiners, except for the closeness
centrality. If the examiner is a senior, an increase in closeness
centrality results in a shortening of 3.43e08 days compared to juniors;
and increase in degree centrality or betweenness centrality or eigen
vector centrality incurs a elongation of 1.38 days, 9.17 days or 1.11e09
days compared to juniors.

#### 3.1.3 Impacts of examiners’ race and ethnicity

    ## 
    ## Impacts of Race to Application Processing Time
    ## ==========================================================================================
    ##                                                Dependent variable:                        
    ##                        -------------------------------------------------------------------
    ##                                          as.numeric(mean_app_proc_time)                   
    ##                                (1)            (2)       (3)      (4)           (5)        
    ## ------------------------------------------------------------------------------------------
    ## degree_centrality             2.470         27.216                           -2.206       
    ##                             (11.563)                                         (1.586)      
    ##                                                                                           
    ## betweenness_centrality       -0.685                                           0.116       
    ##                              (6.898)                                         (1.130)      
    ##                                                                                           
    ## eigenvector_centrality    -753,913.300                                  -166,134,831.000  
    ##                          (2,331,264.000)                                (182,578,152.000) 
    ##                                                                                           
    ## closeness_centrality    -989,640,401.000                                -899,423,122.000  
    ##                        (3,750,979,097.000)                             (2,246,032,850.000)
    ##                                                                                           
    ## Constant                    1,727.745      1,441.996 1,839.000 636.000      1,758.653     
    ##                            (1,808.341)                                     (1,079.066)    
    ##                                                                                           
    ## ------------------------------------------------------------------------------------------
    ## Observations                   27              2         1        1            114        
    ## R2                            0.010          1.000     0.000    0.000         0.031       
    ## Adjusted R2                  -0.170                    0.000    0.000        -0.005       
    ## Residual Std. Error     269.603 (df = 22)                              374.716 (df = 109) 
    ## F Statistic            0.057 (df = 4; 22)                              0.859 (df = 4; 109)
    ## ==========================================================================================
    ## Note:                                                          *p<0.1; **p<0.05; ***p<0.01

Adding one more unit in degree centrality subtracts, on average, an
asian’s mean application processing time by 5.4 days, if holding
everything else equal. Adding one more unit in betweenness centrality
subtracts, on average, an asian’s mean application processing time by
4.7 days, if holding everything else equal.

Adding one more unit in degree centrality subtracts, on average, a
white’s mean application processing time by 2.4 days, if holding
everything else equal. Adding one more unit in betweenness centrality
adds, on average, a white’s mean application processing time by 0.03
days, if holding everything else equal.

    ##      value count
    ## 1    asian    27
    ## 2    black     2
    ## 3 hispanic     1
    ## 4    other     1
    ## 5    white   114

This distribution explains why the regressions made for races with not
enough datapoints didn’t work.

To better understand potential reasons and control for other
characteristics of examiner that might influence the relationship, let’s
take a look at the distribution of race for the 2 work groups combined.

    ## Don't know how to automatically pick scale for object of type difftime. Defaulting to continuous.

![](final_report_files/figure-gfm/unnamed-chunk-44-1.png)<!-- -->

Let’s consider race at the both workgroups level now.

    ## 
    ## Call:
    ## lm(formula = as.numeric(mean_app_proc_time) ~ degree_centrality + 
    ##     betweenness_centrality + eigenvector_centrality + closeness_centrality + 
    ##     as.factor(race) + as.factor(race) * degree_centrality + as.factor(race) * 
    ##     betweenness_centrality, data = examiner_joined_2wg)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -892.52 -208.00   -5.06  160.39 1291.90 
    ## 
    ## Coefficients: (5 not defined because of singularities)
    ##                                                  Estimate Std. Error t value
    ## (Intercept)                                     1.685e+03  9.545e+02   1.766
    ## degree_centrality                               2.623e+00  1.513e+01   0.173
    ## betweenness_centrality                         -7.896e-01  8.236e+00  -0.096
    ## eigenvector_centrality                         -8.093e+05  3.087e+06  -0.262
    ## closeness_centrality                           -9.022e+08  1.969e+09  -0.458
    ## as.factor(race)black                            1.888e+02  6.511e+02   0.290
    ## as.factor(race)hispanic                         5.062e+02  5.256e+02   0.963
    ## as.factor(race)other                           -6.417e+02  3.735e+02  -1.718
    ## as.factor(race)white                            7.060e+01  1.146e+02   0.616
    ## degree_centrality:as.factor(race)black          2.439e+01  1.693e+02   0.144
    ## degree_centrality:as.factor(race)hispanic              NA         NA      NA
    ## degree_centrality:as.factor(race)other                 NA         NA      NA
    ## degree_centrality:as.factor(race)white         -4.800e+00  1.523e+01  -0.315
    ## betweenness_centrality:as.factor(race)black            NA         NA      NA
    ## betweenness_centrality:as.factor(race)hispanic         NA         NA      NA
    ## betweenness_centrality:as.factor(race)other            NA         NA      NA
    ## betweenness_centrality:as.factor(race)white     9.245e-01  8.318e+00   0.111
    ##                                                Pr(>|t|)  
    ## (Intercept)                                      0.0797 .
    ## degree_centrality                                0.8626  
    ## betweenness_centrality                           0.9238  
    ## eigenvector_centrality                           0.7936  
    ## closeness_centrality                             0.6476  
    ## as.factor(race)black                             0.7723  
    ## as.factor(race)hispanic                          0.3373  
    ## as.factor(race)other                             0.0881 .
    ## as.factor(race)white                             0.5390  
    ## degree_centrality:as.factor(race)black           0.8857  
    ## degree_centrality:as.factor(race)hispanic            NA  
    ## degree_centrality:as.factor(race)other               NA  
    ## degree_centrality:as.factor(race)white           0.7531  
    ## betweenness_centrality:as.factor(race)black          NA  
    ## betweenness_centrality:as.factor(race)hispanic       NA  
    ## betweenness_centrality:as.factor(race)other          NA  
    ## betweenness_centrality:as.factor(race)white      0.9117  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 357.7 on 133 degrees of freedom
    ## Multiple R-squared:  0.06913,    Adjusted R-squared:  -0.007859 
    ## F-statistic: 0.8979 on 11 and 133 DF,  p-value: 0.5443

Adding one more unit in degree centrality subtracts, on average, mean
application processing time by 2.8 days, if holding everything else
equal. Adding one more unit in betweenness centrality subtracts, on
average, mean application processing time by 1 days, if holding
everything else equal. Being a black examiner adds, on average, mean
application processing time by 185 days, if holding everything else
equal. The interaction terms of race x centrality has shown that being a
white examiner, the effect of degree centrality is positive and the
effect of betweenness centrality is positive in relation to the mean
application processing time.

# 4. Conclusions and Recommendations

To USPTO, it is important to understand the fundamental reasons of
gender imbalance within the organization and why male examiners in
general takes 274 less days to process applications in selected work
groups. With centrality, we can conclude that a cohesive network around
an examiner is important to the efficiency of processing patent
applications, and implies that application processing is more an
non-divergent organizational change than a divergent change. From gender
impact analysis, we are also aware of the strengths and working style of
females and males that male examiners are more good at building cohesive
network that all examiners know each other well, while female examiners
are more good at bridging network that they have close examiners in
different groups who don’t know each other well - this applies when
there is better gender balance, and less of a benefit under gender
imbalance.

With that being said, since the main duty of examiners are processing
applications, it makes sense that naturally more male examiners with
network building capabilities are acquired to perform the day-to-day
task. However, from a gender equity perspective, it is more beneficial
to strike a gender balance to promote organization diversity &
inclusion. To USPTO business, there could be other transformation
projects (divergent changes perhaps) which might require involvement of
examiners on top of business as usual.

Considering the impact of seniority to the mean application processing
time, we found that junior examiners take 9.60 days longer to process
applications than senior and that for senior, having higher centrality
score actually hinders their speed in processing the patent
applications. This could be explained by the greater responsibility they
take being involved in cohesive and bridging networks under the context
of an advice net. This is supported by the fact that there more senior
being consulted than seeking advice in the directed network, and
pointing towards the need to review workload per seniority level and the
coaching and mentoring approach to enhance operational efficiency.

Imbalance of examiners’ race and ethnicity is also seen which lack of
Black and Hispanic data points are too few for us to draw useful
conclusion on the company level. Under similar limitations, a closer
look into the selected work groups at least shed us some light on
potential racial inequality issue that being a black examiner takes 185
days more in mean processing time while networking for white examiners
helps reducing mean application processing time. Further study on how
the racial distribution looks like against the demographic of US might
help evaluation of inclusion and diversity policy and formulation of
targeted internal training and team building to break the obstacles to
bond and perform efficiently.

Last but not least, analysis on specific work group level and individual
level are strongly recommended to evaluate examiner performance more
fairly in quarterly/yearly reviews. Different metrics apart from
application processing time / efficiency in handling applications shall
be included, such as the quality of applications processing, tenure and
promotion, etc. In this way, we can fit the context of organizational
network analysis into the bigger picture of human resource strategies.
