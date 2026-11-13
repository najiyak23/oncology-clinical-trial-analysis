# Clinical Study Report (CSR)-Style Summary

## Title

Efficacy and Safety of an Investigational Treatment Compared with Placebo in a Simulated Oncology Clinical Trial Using ADaM Datasets

**Note:** all data in this report are simulated. The report demonstrates an analysis and reporting workflow. It does not provide evidence about any real treatment.

## 1. Introduction

This analysis compares an investigational treatment with placebo in a simulated oncology clinical trial. The data were generated in SAS as CDISC ADaM-structured datasets (ADSL, ADTTE, ADAE) and then read into R for statistical analysis.

Endpoints assessed:

- Overall survival (OS)
- Progression-free survival (PFS)
- Safety, based on adverse events (AEs)

## 2. Objectives

**Primary objective**

- To compare overall survival between the investigational treatment and placebo.

**Secondary objectives**

- To compare progression-free survival (PFS).
- To describe the incidence and severity of adverse events.
- To give a descriptive overview of efficacy and safety together.

## 3. Data Sources and Populations

The analysis used simulated datasets created in SAS following CDISC ADaM conventions for oncology trials.

Datasets used:

- **ADSL:** subject-level data (treatment assignment, age, sex, population flags)
- **ADTTE:** time-to-event data for OS and PFS (`AVAL` for time, `CNSR` for censoring)
- **ADAE:** adverse event data (preferred term, severity, treatment-emergent flag)

The datasets were exported as `.sas7bdat` files and read into R with the `haven` package.

**Analysis populations**

- **Efficacy:** intention-to-treat (ITT) population, `ITTFL = "Y"`
- **Safety:** safety population, `SAFFL = "Y"`, using treatment-emergent adverse events (`TRTEMFL = "Y"`)

The trial had 120 patients: 55 on placebo and 65 on treatment.

## 4. Methodology

### 4.1 Efficacy analysis

OS and PFS were analysed as time-to-event endpoints. A patient with `CNSR == 0` had the event, and all others were censored.

- Kaplan-Meier estimates of survival by treatment arm, with median survival and 95% confidence intervals
- Log-rank test to compare the arms
- Cox proportional hazards regression to estimate hazard ratios (HR) with 95% confidence intervals
- The proportional hazards assumption was checked with `cox.zph`
- OS and PFS were both analysed with an unadjusted model. For PFS, a second model adjusted for age and sex was run as a sensitivity analysis.

### 4.2 Safety analysis

Safety was summarised from ADAE for the safety population:

- Number and percentage of patients with at least one AE, using the number of patients in each arm as the denominator
- Total number of AE events per arm
- Maximum severity per patient (mild, moderate, severe), counting each patient once
- Most common AE terms, counted by patient

All analyses were done in R with `survival` and `dplyr`. Plots used base R graphics. The results are descriptive, with no formal testing of safety outcomes.

## 5. Results

### 5.1 Overall survival (OS)

- 39 deaths occurred among the 120 patients.
- Hazard ratio (HR): 0.44 (95% CI 0.22 to 0.86), meaning a 56% lower hazard of death on treatment than on placebo.
- Log-rank p = 0.01; Cox model p = 0.017.
- Median OS was 18.5 on placebo and 51.7 on treatment (some confidence limits for the medians could not be estimated).
- The proportional hazards check did not reject the assumption (p = 0.25).

**Interpretation:** In this simulated trial, treatment was associated with longer overall survival than placebo. The confidence interval is wide, so the data are compatible with anything from a modest to a large effect.

### 5.2 Progression-free survival (PFS)

- Unadjusted HR: 0.82 (95% CI 0.47 to 1.43).
- HR adjusted for age and sex: 0.85 (95% CI 0.48 to 1.49), p = 0.56.
- Adjusting for age and sex did not materially change the estimate.

**Interpretation:** Both confidence intervals include 1, so the analysis does not show a PFS difference. It is also too imprecise to rule one out.

### 5.3 Safety analysis

#### 5.3.1 Overall adverse events

| Group | Patients in arm | Patients with at least one AE | Percentage | Total AE events |
|---|---|---|---|---|
| Placebo | 55 | 47 | 85.5% | 103 |
| Treatment | 65 | 47 | 72.3% | 97 |

**Interpretation:** The same number of patients had an AE in each arm (47), but the arms differ in size, so a smaller percentage of treatment patients had an AE. Percentages are therefore more informative than raw counts.

#### 5.3.2 Severity of adverse events

- A severe AE was the worst event for 49.1% of placebo patients and 32.3% of treatment patients.

**Interpretation:** There is no sign of more severe adverse events on treatment in these simulated data.

#### 5.3.3 Common adverse events

The most frequent AE terms were headache, fatigue, nausea and neutropenia. Patient counts and percentages by arm are in the full report.

**Interpretation:** No unexpected AE terms appeared in these simulated data. The tables are descriptive, so they cannot show that the safety profiles are equivalent.

## 6. Overview of Efficacy and Safety

- OS: HR 0.44 (95% CI 0.22 to 0.86) in favour of treatment.
- PFS: HR 0.82 (95% CI 0.47 to 1.43), no clear difference.
- Safety: lower percentages of patients with any AE and with severe AEs on treatment, descriptively.

**Overall:** In this simulated trial the treatment arm had longer overall survival and no sign of more toxicity. Because the confidence intervals are wide and the data are simulated, this should not be read as evidence of a favourable benefit-risk profile for a real treatment.

## 7. Discussion

In this simulated trial, treatment was associated with a lower hazard of death than placebo (HR 0.44, 95% CI 0.22 to 0.86). PFS did not show a clear difference (HR 0.82, 95% CI 0.47 to 1.43). The PFS interval is wide, so the analysis cannot tell no effect from a modest one. In simulated data, an OS difference without a clear PFS difference most likely reflects how the data were generated or chance, so no biological explanation is drawn from it.

The AE summaries show no obvious safety signal on treatment. They are descriptive and based on a small sample, so they do not establish that the safety profiles are equivalent.

**Limitations**

- The data are simulated and the sample is small (120 patients).
- Confidence intervals for OS and PFS are wide.
- Safety results are descriptive, with no formal testing.
- AE severity uses the categories in the data (mild, moderate, severe), not CTCAE grades.
- SAS cut the AE term "Neutropenia" to 8 characters ("Neutrope"). It was restored to the full term in R, and the full term is used throughout this report.

## 8. Conclusion

In this simulated oncology trial, treatment was associated with longer overall survival than placebo (HR 0.44, 95% CI 0.22 to 0.86). PFS showed no clear difference (HR 0.82, 95% CI 0.47 to 1.43), and the AE summaries showed no sign of more toxicity on treatment. The results are imprecise and come from simulated data, so they illustrate the analysis rather than support a conclusion about a real treatment.

## 9. Tools and Programming

- **SAS:** creation of the ADaM datasets (ADSL, ADTTE, ADAE) following CDISC conventions for oncology trials
- **R:**
  - `survival`: Kaplan-Meier, log-rank test, Cox models, proportional hazards check
  - `dplyr`: data preparation and summaries
  - `haven`: reading SAS `.sas7bdat` files into R
