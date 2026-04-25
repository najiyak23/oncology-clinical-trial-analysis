Oncology Clinical Trial Analysis
================
Najiya K

# 🧬 Project Overview

This project evaluates the effectiveness and safety of a treatment
compared to placebo using survival analysis and adverse event data.

------------------------------------------------------------------------

# 📦 Load Packages and Data

``` r
library(haven)
```

    ## Warning: package 'haven' was built under R version 4.3.3

``` r
library(dplyr)
```

    ## Warning: package 'dplyr' was built under R version 4.3.3

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(survival)
```

    ## Warning: package 'survival' was built under R version 4.3.3

``` r
adsl <- read_sas("adsl.sas7bdat")
adtte <- read_sas("adtte.sas7bdat")
adae <- read_sas("adae.sas7bdat")
```

------------------------------------------------------------------------

# 🧪 Overall Survival (OS)

## Prepare Dataset

``` r
adtte_os <- adtte %>%
  filter(PARAMCD == "OS") %>%
  left_join(adsl, by = "USUBJID")
```

------------------------------------------------------------------------

## Kaplan-Meier Curve

``` r
km_fit <- survfit(Surv(AVAL, CNSR == 0) ~ TRT01P, data = adtte_os)

plot(km_fit, col = c("blue", "red"), lwd = 2,
     xlab = "Time", ylab = "Survival Probability")

legend("topright",
       legend = c("Placebo", "Treatment"),
       col = c("blue", "red"),
       lwd = 2)
```

![](oncology_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

### 🧠 Insight

The treatment group shows improved survival compared to placebo.

------------------------------------------------------------------------

## Log-Rank Test

``` r
logrank_test <- survdiff(Surv(AVAL, CNSR == 0) ~ TRT01P, data = adtte_os)
logrank_test
```

    ## Call:
    ## survdiff(formula = Surv(AVAL, CNSR == 0) ~ TRT01P, data = adtte_os)
    ## 
    ##                   N Observed Expected (O-E)^2/E (O-E)^2/V
    ## TRT01P=Placebo   55       18     11.3      3.92      5.98
    ## TRT01P=Treatment 65       21     27.7      1.61      5.98
    ## 
    ##  Chisq= 6  on 1 degrees of freedom, p= 0.01

### 🧠 Insight

The difference between groups is statistically significant.

------------------------------------------------------------------------

## Cox Model (Hazard Ratio)

``` r
cox_os <- coxph(Surv(AVAL, CNSR == 0) ~ TRT01P, data = adtte_os)
summary(cox_os)
```

    ## Call:
    ## coxph(formula = Surv(AVAL, CNSR == 0) ~ TRT01P, data = adtte_os)
    ## 
    ##   n= 120, number of events= 39 
    ## 
    ##                   coef exp(coef) se(coef)      z Pr(>|z|)  
    ## TRT01PTreatment -0.821     0.440    0.344 -2.386    0.017 *
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ##                 exp(coef) exp(-coef) lower .95 upper .95
    ## TRT01PTreatment      0.44      2.273    0.2242    0.8636
    ## 
    ## Concordance= 0.62  (se = 0.042 )
    ## Likelihood ratio test= 5.59  on 1 df,   p=0.02
    ## Wald test            = 5.69  on 1 df,   p=0.02
    ## Score (logrank) test = 5.98  on 1 df,   p=0.01

### 🧠 Insight

The treatment reduces the risk of death compared to placebo.

------------------------------------------------------------------------

# 🔬 Progression-Free Survival (PFS)

## Prepare Dataset

``` r
adtte_pfs <- adtte %>%
  filter(PARAMCD == "PF") %>%
  left_join(adsl, by = "USUBJID")
```

------------------------------------------------------------------------

## Kaplan-Meier Curve

``` r
km_fit_pfs <- survfit(Surv(AVAL, CNSR == 0) ~ TRT01P, data = adtte_pfs)

plot(km_fit_pfs,
     col = c("blue", "red"),
     lwd = 2,
     xlab = "Time",
     ylab = "Progression-Free Survival Probability")

legend("topright",
       legend = c("Placebo", "Treatment"),
       col = c("blue", "red"),
       lwd = 2)
```

![](oncology_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

------------------------------------------------------------------------

## Cox Model (Adjusted)

``` r
cox_pfs <- coxph(Surv(AVAL, CNSR == 0) ~ TRT01P + AGE + SEX, data = adtte_pfs)
summary(cox_pfs)
```

    ## Call:
    ## coxph(formula = Surv(AVAL, CNSR == 0) ~ TRT01P + AGE + SEX, data = adtte_pfs)
    ## 
    ##   n= 120, number of events= 57 
    ## 
    ##                      coef exp(coef)  se(coef)      z Pr(>|z|)
    ## TRT01PTreatment -0.166350  0.846750  0.287227 -0.579    0.562
    ## AGE             -0.004326  0.995684  0.013799 -0.313    0.754
    ## SEXM             0.365247  1.440870  0.279779  1.305    0.192
    ## 
    ##                 exp(coef) exp(-coef) lower .95 upper .95
    ## TRT01PTreatment    0.8467      1.181    0.4822     1.487
    ## AGE                0.9957      1.004    0.9691     1.023
    ## SEXM               1.4409      0.694    0.8327     2.493
    ## 
    ## Concordance= 0.583  (se = 0.047 )
    ## Likelihood ratio test= 2.32  on 3 df,   p=0.5
    ## Wald test            = 2.28  on 3 df,   p=0.5
    ## Score (logrank) test = 2.3  on 3 df,   p=0.5

### 🧠 Insight

No statistically significant improvement in progression-free survival
was observed.

------------------------------------------------------------------------

# 💊 Safety Analysis (ADAE)

## Prepare Dataset

``` r
adae2 <- adae %>%
  left_join(adsl, by = "USUBJID")
```

------------------------------------------------------------------------

## Total Adverse Events

``` r
adae_summary <- adae2 %>%
  group_by(TRT01A) %>%
  summarise(
    total_AE = n(),
    patients = n_distinct(USUBJID)
  )

adae_summary
```

    ## # A tibble: 2 × 3
    ##   TRT01A    total_AE patients
    ##   <chr>        <int>    <int>
    ## 1 Placebo        103       47
    ## 2 Treatment       97       47

### 🧠 Insight

Adverse events are similar between treatment and placebo groups.

------------------------------------------------------------------------

## Severity Distribution

``` r
ae_severity <- adae2 %>%
  group_by(TRT01A, AESEV) %>%
  summarise(count = n())
```

    ## `summarise()` has grouped output by 'TRT01A'. You can override using the
    ## `.groups` argument.

``` r
ae_severity
```

    ## # A tibble: 6 × 3
    ## # Groups:   TRT01A [2]
    ##   TRT01A    AESEV    count
    ##   <chr>     <chr>    <int>
    ## 1 Placebo   MILD        37
    ## 2 Placebo   MODERATE    33
    ## 3 Placebo   SEVERE      33
    ## 4 Treatment MILD        31
    ## 5 Treatment MODERATE    36
    ## 6 Treatment SEVERE      30

### 🧠 Insight

No increase in severe adverse events in the treatment group.

------------------------------------------------------------------------

## Common Adverse Events

``` r
common_ae <- adae2 %>%
  group_by(TRT01A, AEDECOD) %>%
  summarise(count = n()) %>%
  arrange(TRT01A, desc(count)) %>%
  group_by(TRT01A) %>%
  slice_head(n = 5)
```

    ## `summarise()` has grouped output by 'TRT01A'. You can override using the
    ## `.groups` argument.

``` r
common_ae
```

    ## # A tibble: 8 × 3
    ## # Groups:   TRT01A [2]
    ##   TRT01A    AEDECOD  count
    ##   <chr>     <chr>    <int>
    ## 1 Placebo   Headache    39
    ## 2 Placebo   Neutrope    31
    ## 3 Placebo   Fatigue     18
    ## 4 Placebo   Nausea      15
    ## 5 Treatment Headache    30
    ## 6 Treatment Fatigue     27
    ## 7 Treatment Nausea      21
    ## 8 Treatment Neutrope    19

### 🧠 Insight

Common adverse events include headache, fatigue, nausea, and
neutropenia.

------------------------------------------------------------------------

# ⚖️ Final Conclusion

The treatment demonstrated a statistically significant improvement in
overall survival compared to placebo, while no significant difference
was observed in progression-free survival. Safety analysis showed a
comparable incidence and severity of adverse events between treatment
and placebo groups, with no increase in severe adverse events. Overall,
the benefit–risk profile of the treatment appears favorable.
