Oncology Clinical Trial Analysis (Simulated Data)
================
Najiya K

# Project Overview

This project demonstrates an end-to-end analysis workflow for a
**simulated** two-arm oncology trial (Treatment vs Placebo). CDISC
ADaM-style datasets (ADSL, ADTTE, ADAE) were generated in SAS and
analysed in R.

**Important:** all data are simulated. The aim is to show the analysis
workflow (survival analysis and safety summaries), not to make clinical
claims about any real treatment.

# Statistical Analysis Plan (summary)

| Item | Specification |
|----|----|
| Population | All randomised patients in ADSL (efficacy); patients with a treatment arm in ADSL (safety) |
| Efficacy endpoints | Overall survival (OS, `PARAMCD = "OS"`) and progression-free survival (PFS, `PARAMCD = "PF"`) |
| Event definition | Event when `CNSR == 0`; censored otherwise |
| Primary comparison | Treatment vs Placebo, Kaplan-Meier estimates and log-rank test |
| Effect estimate | Hazard ratio (HR) with 95% CI from a Cox proportional hazards model |
| Covariates | Primary models unadjusted for both OS and PFS; PFS additionally adjusted for age and sex as a sensitivity analysis |
| Assumption check | Proportional hazards assessed with `cox.zph` |
| Safety | Number and percentage of patients with at least one adverse event (AE), by maximum severity, and most common AE terms, using arm sizes from ADSL as denominators |

# Load Packages and Data

``` r
# Place the .sas7bdat files in the same folder as this file
adsl  <- read_sas("adsl.sas7bdat")
adtte <- read_sas("adtte.sas7bdat")
adae  <- read_sas("adae.sas7bdat")

# Subject-level variables used throughout
adsl_keep <- adsl %>%
  select(any_of(c("USUBJID", "TRT01P", "TRT01A", "AGE", "SEX")))

# Time unit label (taken from ADTTE if the AVALU variable exists)
time_unit <- if ("AVALU" %in% names(adtte)) unique(adtte$AVALU)[1] else "unit not specified"
time_lab  <- paste0("Time (", time_unit, ")")

# Helper functions for formatting and for extracting a hazard ratio
fmt_p <- function(p) ifelse(p < 0.001, "<0.001", sprintf("%.3f", p))

cox_hr <- function(fit, term = "TRT01PTreatment") {
  s  <- summary(fit)$coefficients
  ci <- summary(fit)$conf.int
  c(hr = ci[term, "exp(coef)"], lo = ci[term, "lower .95"],
    hi = ci[term, "upper .95"], p = s[term, "Pr(>|z|)"])
}

fmt_hr <- function(r) {
  sprintf("HR %.2f, 95%% CI %.2f to %.2f, p = %s", r["hr"], r["lo"], r["hi"], fmt_p(r["p"]))
}

# Build an analysis dataset for one time-to-event parameter
prep_tte <- function(code) {
  adtte %>%
    filter(PARAMCD == code) %>%
    select(-any_of(c("TRT01P", "TRT01A", "AGE", "SEX"))) %>%
    left_join(adsl_keep, by = "USUBJID") %>%
    mutate(TRT01P = factor(TRT01P, levels = c("Placebo", "Treatment")),
           EVENT  = as.integer(CNSR == 0))
}

# Number-at-risk table at chosen time points
risk_table <- function(fit, times) {
  s <- summary(fit, times = times, extend = TRUE)
  data.frame(Arm = sub("TRT01P=", "", as.character(s$strata)),
             Time = s$time, At_risk = s$n.risk)
}
```

## Analysis populations

``` r
adsl_keep %>% count(TRT01P, name = "N patients") %>% kable()
```

| TRT01P    | N patients |
|:----------|-----------:|
| Placebo   |         55 |
| Treatment |         65 |

# Overall Survival (OS)

``` r
adtte_os <- prep_tte("OS")
km_os <- survfit(Surv(AVAL, EVENT) ~ TRT01P, data = adtte_os)
```

## Kaplan-Meier curve

``` r
plot(km_os, col = c("blue", "red"), lwd = 2,
     xlab = time_lab, ylab = "Overall survival probability")
legend("bottomleft", legend = c("Placebo", "Treatment"),
       col = c("blue", "red"), lwd = 2, bty = "n")
```

![](oncology_analysis--1-_files/figure-gfm/os-km-1.png)<!-- -->

Number at risk:

``` r
tp <- round(seq(0, max(adtte_os$AVAL, na.rm = TRUE), length.out = 6))
risk_table(km_os, tp) %>%
  tidyr::pivot_wider(names_from = Time, values_from = At_risk) %>%
  kable()
```

| Arm       |   0 |  15 |  30 |  44 |  59 |  74 |
|:----------|----:|----:|----:|----:|----:|----:|
| Placebo   |  55 |  15 |   4 |   1 |   0 |   0 |
| Treatment |  65 |  37 |  16 |   9 |   2 |   0 |

Events, censoring and median survival by arm:

``` r
kable(as.data.frame(summary(km_os)$table)[, c("records", "events", "median", "0.95LCL", "0.95UCL")],
      digits = 1)
```

|                  | records | events | median | 0.95LCL | 0.95UCL |
|:-----------------|--------:|-------:|-------:|--------:|--------:|
| TRT01P=Placebo   |      55 |     18 |   18.5 |    14.5 |      NA |
| TRT01P=Treatment |      65 |     21 |   51.7 |    31.9 |      NA |

## Log-rank test

``` r
lr_os <- survdiff(Surv(AVAL, EVENT) ~ TRT01P, data = adtte_os)
lr_os
```

    ## Call:
    ## survdiff(formula = Surv(AVAL, EVENT) ~ TRT01P, data = adtte_os)
    ## 
    ##                   N Observed Expected (O-E)^2/E (O-E)^2/V
    ## TRT01P=Placebo   55       18     11.3      3.92      5.98
    ## TRT01P=Treatment 65       21     27.7      1.61      5.98
    ## 
    ##  Chisq= 6  on 1 degrees of freedom, p= 0.01

``` r
lr_os_p <- 1 - pchisq(lr_os$chisq, df = length(lr_os$n) - 1)
```

## Cox model (hazard ratio)

``` r
cox_os <- coxph(Surv(AVAL, EVENT) ~ TRT01P, data = adtte_os)
summary(cox_os)
```

    ## Call:
    ## coxph(formula = Surv(AVAL, EVENT) ~ TRT01P, data = adtte_os)
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

``` r
res_os <- cox_hr(cox_os)
```

Proportional hazards check:

``` r
ph_os <- cox.zph(cox_os)
ph_os
```

    ##        chisq df    p
    ## TRT01P  1.32  1 0.25
    ## GLOBAL  1.32  1 0.25

``` r
plot(ph_os)
```

![](oncology_analysis--1-_files/figure-gfm/os-ph-1.png)<!-- -->

**Interpretation.** In this simulated dataset, OS differed between arms
(HR 0.44, 95% CI 0.22 to 0.86, p = 0.017; log-rank p = 0.014). The
confidence interval is wide, so the data are compatible with a range of
effect sizes from modest to large.

# Progression-Free Survival (PFS)

``` r
adtte_pfs <- prep_tte("PF")
km_pfs <- survfit(Surv(AVAL, EVENT) ~ TRT01P, data = adtte_pfs)
```

## Kaplan-Meier curve

``` r
plot(km_pfs, col = c("blue", "red"), lwd = 2,
     xlab = time_lab, ylab = "Progression-free survival probability")
legend("bottomleft", legend = c("Placebo", "Treatment"),
       col = c("blue", "red"), lwd = 2, bty = "n")
```

![](oncology_analysis--1-_files/figure-gfm/pfs-km-1.png)<!-- -->

``` r
kable(as.data.frame(summary(km_pfs)$table)[, c("records", "events", "median", "0.95LCL", "0.95UCL")],
      digits = 1)
```

|                  | records | events | median | 0.95LCL | 0.95UCL |
|:-----------------|--------:|-------:|-------:|--------:|--------:|
| TRT01P=Placebo   |      55 |     22 |   17.6 |     8.9 |      NA |
| TRT01P=Treatment |      65 |     35 |   18.1 |     7.6 |    35.2 |

## Log-rank test and Cox models

``` r
lr_pfs <- survdiff(Surv(AVAL, EVENT) ~ TRT01P, data = adtte_pfs)
lr_pfs_p <- 1 - pchisq(lr_pfs$chisq, df = length(lr_pfs$n) - 1)

# Primary (unadjusted, consistent with OS)
cox_pfs <- coxph(Surv(AVAL, EVENT) ~ TRT01P, data = adtte_pfs)
res_pfs <- cox_hr(cox_pfs)

# Sensitivity analysis (adjusted for age and sex)
cox_pfs_adj <- coxph(Surv(AVAL, EVENT) ~ TRT01P + AGE + SEX, data = adtte_pfs)
res_pfs_adj <- cox_hr(cox_pfs_adj)

summary(cox_pfs_adj)
```

    ## Call:
    ## coxph(formula = Surv(AVAL, EVENT) ~ TRT01P + AGE + SEX, data = adtte_pfs)
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

``` r
cox.zph(cox_pfs)
```

    ##        chisq df   p
    ## TRT01P 0.268  1 0.6
    ## GLOBAL 0.268  1 0.6

**Interpretation.** For PFS the unadjusted estimate was HR 0.82, 95% CI
0.47 to 1.43, p = 0.486 and the age- and sex-adjusted estimate was HR
0.85, 95% CI 0.48 to 1.49, p = 0.562. The confidence intervals include
1, so the data neither show nor rule out a PFS difference: the analysis
is too imprecise to tell. With simulated data, an OS difference without
a clear PFS difference most plausibly reflects how the data were
generated or chance, and should not be read as a biological mechanism.

# Safety Analysis (ADAE)

## Safety population and data checks

``` r
safety_n <- adsl_keep %>% count(TRT01A, name = "N")
safety_n %>% kable()
```

| TRT01A    |   N |
|:----------|----:|
| Placebo   |  55 |
| Treatment |  65 |

``` r
ae <- adae %>%
  select(-any_of("TRT01A")) %>%
  left_join(adsl_keep %>% select(USUBJID, TRT01A), by = "USUBJID")

# Keep treatment-emergent AEs only, if that flag exists in ADAE
if ("TRTEMFL" %in% names(ae)) {
  ae <- ae %>% filter(TRTEMFL == "Y")
  cat("Filtered to treatment-emergent AEs (TRTEMFL == 'Y').\n")
} else {
  cat("No TRTEMFL variable in ADAE: all recorded AEs are summarised.\n")
}
```

    ## Filtered to treatment-emergent AEs (TRTEMFL == 'Y').

``` r
# Check for truncated AE terms (SAS character variables default to 8 characters)
ae %>% filter(nchar(AEDECOD) == 8) %>% distinct(AEDECOD)
```

    ## # A tibble: 2 × 1
    ##   AEDECOD 
    ##   <chr>   
    ## 1 Headache
    ## 2 Neutrope

If a term such as `Neutrope` appears above, the SAS step that created
ADAE truncated the variable. See the SAS fix in the repository notes
(`LENGTH AEDECOD $40;`) and regenerate the dataset before running the
rest.

## Patients with at least one adverse event

``` r
ae_any <- ae %>%
  distinct(USUBJID, TRT01A) %>%
  count(TRT01A, name = "Patients with AE") %>%
  left_join(safety_n, by = "TRT01A") %>%
  left_join(ae %>% count(TRT01A, name = "Total AE events"), by = "TRT01A") %>%
  mutate(`% of arm` = round(100 * `Patients with AE` / N, 1))
kable(ae_any)
```

| TRT01A    | Patients with AE |   N | Total AE events | % of arm |
|:----------|-----------------:|----:|----------------:|---------:|
| Placebo   |               47 |  55 |             103 |     85.5 |
| Treatment |               47 |  65 |              97 |     72.3 |

## Maximum severity per patient

``` r
ae_sev <- ae %>%
  mutate(sev = factor(AESEV, levels = c("MILD", "MODERATE", "SEVERE"), ordered = TRUE)) %>%
  group_by(TRT01A, USUBJID) %>%
  summarise(max_sev = max(sev), .groups = "drop") %>%
  count(TRT01A, max_sev) %>%
  left_join(safety_n, by = "TRT01A") %>%
  mutate(pct = round(100 * n / N, 1))
kable(ae_sev)
```

| TRT01A    | max_sev  |   n |   N |  pct |
|:----------|:---------|----:|----:|-----:|
| Placebo   | MILD     |   8 |  55 | 14.5 |
| Placebo   | MODERATE |  12 |  55 | 21.8 |
| Placebo   | SEVERE   |  27 |  55 | 49.1 |
| Treatment | MILD     |   9 |  65 | 13.8 |
| Treatment | MODERATE |  17 |  65 | 26.2 |
| Treatment | SEVERE   |  21 |  65 | 32.3 |

## Most common adverse events (patients, by preferred term)

``` r
ae_common <- ae %>%
  distinct(USUBJID, TRT01A, AEDECOD) %>%
  count(TRT01A, AEDECOD, name = "Patients") %>%
  left_join(safety_n, by = "TRT01A") %>%
  mutate(`% of arm` = round(100 * Patients / N, 1)) %>%
  group_by(TRT01A) %>%
  arrange(desc(Patients), .by_group = TRUE) %>%
  slice_head(n = 5)
kable(ae_common)
```

| TRT01A    | AEDECOD  | Patients |   N | % of arm |
|:----------|:---------|---------:|----:|---------:|
| Placebo   | Headache |       31 |  55 |     56.4 |
| Placebo   | Neutrope |       24 |  55 |     43.6 |
| Placebo   | Fatigue  |       17 |  55 |     30.9 |
| Placebo   | Nausea   |       13 |  55 |     23.6 |
| Treatment | Headache |       25 |  65 |     38.5 |
| Treatment | Fatigue  |       22 |  65 |     33.8 |
| Treatment | Nausea   |       17 |  65 |     26.2 |
| Treatment | Neutrope |       16 |  65 |     24.6 |

**Interpretation.** Compare the percentages of patients per arm above,
not the raw event counts, because the arms are different sizes. These
summaries are descriptive; with this sample size they cannot establish
that the safety profiles are equivalent.

# Conclusion

In this simulated trial, treatment was associated with a lower hazard of
death than placebo (HR 0.44, 95% CI 0.22 to 0.86, p = 0.017), while PFS
showed no clear difference (HR 0.82, 95% CI 0.47 to 1.43, p = 0.486),
with wide confidence intervals for both. The AE summaries show no
obvious safety signal in the simulated data, but they are descriptive
only. Because the data are simulated, none of this is evidence about a
real treatment; the project demonstrates the analysis workflow.

# Reproducibility

``` r
sessionInfo()
```

    ## R version 4.3.1 (2023-06-16 ucrt)
    ## Platform: x86_64-w64-mingw32/x64 (64-bit)
    ## Running under: Windows 11 x64 (build 26300)
    ## 
    ## Matrix products: default
    ## 
    ## 
    ## locale:
    ## [1] LC_COLLATE=English_India.utf8  LC_CTYPE=English_India.utf8   
    ## [3] LC_MONETARY=English_India.utf8 LC_NUMERIC=C                  
    ## [5] LC_TIME=English_India.utf8    
    ## 
    ## time zone: Asia/Calcutta
    ## tzcode source: internal
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] knitr_1.45     survival_3.8-3 dplyr_1.1.4    haven_2.5.4   
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] vctrs_0.6.5       cli_3.6.4         rlang_1.1.5       xfun_0.52        
    ##  [5] highr_0.10        forcats_1.0.0     purrr_1.0.2       generics_0.1.3   
    ##  [9] glue_1.6.2        htmltools_0.5.8.1 hms_1.1.3         fansi_1.0.5      
    ## [13] rmarkdown_2.25    grid_4.3.1        evaluate_1.0.5    tibble_3.2.1     
    ## [17] tzdb_0.5.0        fastmap_1.2.0     yaml_2.3.7        lifecycle_1.0.5  
    ## [21] compiler_4.3.1    pkgconfig_2.0.3   tidyr_1.3.0       rstudioapi_0.15.0
    ## [25] lattice_0.21-8    digest_0.6.33     R6_2.6.1          readr_2.1.4      
    ## [29] tidyselect_1.2.0  utf8_1.2.3        splines_4.3.1     pillar_1.9.0     
    ## [33] magrittr_2.0.3    Matrix_1.5-4.1    withr_3.0.2       tools_4.3.1
