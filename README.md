# 🧬 Clinical Study Report (CSR)–Style Summary

## Title  
**Evaluation of Efficacy and Safety of an Investigational Treatment Compared to Placebo in a Simulated Oncology Clinical Trial Using ADaM Datasets**

---

## 📌 1. Introduction

This analysis evaluates the efficacy and safety of an investigational treatment versus placebo in a **simulated oncology clinical trial**. The underlying data were generated in **SAS** as **CDISC ADaM‑compliant datasets** (ADSL, ADTTE, ADAE) and then read into **R** for statistical analysis.

**Key endpoints assessed were:**  
- Overall Survival (OS)  
- Progression‑Free Survival (PFS)  
- Safety via adverse events (AEs)

---

## 🎯 2. Objectives

**Primary Objective**  
- To evaluate the **effectiveness and safety** of the investigational treatment compared to placebo.

**Secondary Objectives**  
- Assess improvement in **Overall Survival (OS)**.  
- Evaluate **Progression‑Free Survival (PFS)**.  
- Analyze the **incidence and severity** of adverse events.  
- Determine the **overall benefit–risk profile** of the treatment.

---

## 📊 3. Data Sources

The analysis was based on simulated datasets created in **SAS** following the **CDISC ADaM standards** for oncology trials.

**Datasets used:**  
- **ADSL**: Subject‑level analysis dataset (demographics, treatment assignment, baseline characteristics)  
- **ADTTE**: Time‑to‑event analysis dataset (OS and PFS endpoints, with analysis variables such as `AVAL`, `STARTDT`, and `CNSR`)  
- **ADAE**: Adverse events dataset (serious and non‑serious AEs, coded terms, severity, reporter, etc.)

These datasets were exported to `.sas7bdat` files and read into **R** for analysis using the `haven` package.

---

## 🔬 4. Methodology

### 4.1 Efficacy Analysis

Time‑to‑event endpoints (OS and PFS) were analyzed using:  
- **Kaplan–Meier method** for survival probability estimation by treatment group.  
- **Log‑rank test** to compare survival curves between treatment and placebo arms.  
- **Cox proportional hazards regression** to estimate **hazard ratios (HR)**, with the PFS model adjusted for **age and sex** as covariates.


### 4.2 Safety Analysis

Safety was summarized using the ADAE dataset:  
- **Total number of adverse events** by treatment group.  
- **Number of patients with ≥1 adverse event**.  
- **Distribution of events by severity** (Mild, Moderate, Severe).  
- **Listing of most common adverse events** (e.g., headache, fatigue, nausea, neutropenia).

Figures and tables were generated in R using `dplyr` for data manipulation and base plotting / `ggplot2`‑style summaries (if applicable).

---

## 📈 5. Results

### 5.1 Overall Survival (OS)

- **Hazard Ratio (HR): 0.44**  
- Approximate **56% reduction in the risk of death** in the treatment arm versus placebo.  
- **Log‑rank test p‑value: 0.01** (statistically significant).

**Interpretation:**  
The treatment provides a **clinically meaningful survival benefit** as compared with placebo, with a clear improvement in OS over the follow‑up period.

### 5.2 Progression‑Free Survival (PFS)

- **Hazard Ratio (HR): 0.85**  
- **p‑value: 0.56** (not statistically significant).  
- Adjustment for **age and sex** did not materially change the estimate.

**Interpretation:**  
There was **no statistically significant improvement in PFS** with the investigational treatment, suggesting that the OS benefit may not be fully explained by a delay in disease progression.

### 5.3 Safety Analysis

#### 5.3.1 Overall Adverse Events

| Group      | Total AEs | Patients with ≥1 AE |
|-----------|-----------|---------------------|
| Placebo   | 103       | 47                  |
| Treatment | 97        | 47                  |

**Interpretation:**  
There is **no increase in the overall number of adverse events** in the treatment group compared with placebo.

#### 5.3.2 Severity of Adverse Events

- **Severe AEs**: Placebo (33) vs Treatment (30)  
- **Moderate AEs**: Similar across groups  
- **Mild AEs**: Slightly lower in the treatment group  

**Interpretation:**  
The **burden of severe adverse events is comparable** between groups, with no evidence of increased toxicity with treatment.

#### 5.3.3 Common Adverse Events

Most frequently observed AEs:  
- Headache  
- Fatigue  
- Nausea  
- Neutropenia  

**Interpretation:**  
The AE profile is **consistent with expectations for oncology trials**, and no unexpected or off‑target safety signals were identified.

---

## ⚖️ 6. Benefit–Risk Assessment

- **Significant improvement in Overall Survival (OS)** (HR = 0.44, p = 0.01).  
- **No significant improvement in Progression‑Free Survival (PFS)** (HR = 0.85, p = 0.56).  
- **Comparable safety profile** between treatment and placebo, with similar rates and severity of AEs.

**Overall Assessment:**  
The treatment demonstrates a **favorable benefit–risk profile**, driven primarily by a survival advantage without a meaningful increase in toxicity.

---

## 🧠 7. Discussion

In this simulated oncology trial, the investigational treatment was associated with a **substantial reduction in the risk of death**, as reflected by a hazard ratio of 0.44 for OS. The absence of a statistically significant effect on PFS suggests that the survival benefit may arise from factors beyond simple disease‑progression delay (e.g., post‑progression survival, treatment‑related effects on competing risks, or other non‑tumor‑related mechanisms).

Critically, the **safety profile remained comparable** to placebo, with similar total AEs, severity distribution, and no unexpected events. This supports the **clinical tolerability** of the regimen and underpins its potential for further development in oncology indications.

---

## ✅ 8. Conclusion

The treatment demonstrated:  
- **Significant improvement in Overall Survival**.  
- **No significant effect on Progression‑Free Survival**.  
- **Comparable safety profile** to placebo.

**Final Conclusion:**  
The **overall benefit–risk profile** supports the clinical value of the investigational treatment in this simulated oncology trial.

---

## 🛠️ 9. Tools and Programming

- **SAS**: Generation of **ADaM datasets** (ADSL, ADTTE, ADAE) following CDISC standards for oncology trials.  
- **R Programming Language**:  
  - Packages:  
    - `survival` (Kaplan–Meier, log‑rank, Cox models)  
    - `dplyr` (data summarization and manipulation)  
    - `haven` (reading SAS `.sas7bdat` files into R)
