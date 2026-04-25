# 🧬 Clinical Study Report (CSR Style)

## 📌 Title
**Evaluation of Efficacy and Safety of Treatment Compared to Placebo in an Oncology Clinical Trial Using ADaM Datasets**

---

## 📖 1. Introduction

This analysis evaluates the efficacy and safety of an investigational treatment compared to placebo in a simulated oncology clinical trial. CDISC-compliant ADaM datasets were used to perform survival and safety analyses. Key endpoints include:

- Overall Survival (OS)
- Progression-Free Survival (PFS)
- Adverse Events (AEs)

---

## 🎯 2. Objectives

### Primary Objective
- To evaluate the effectiveness and safety of the treatment compared to placebo

### Secondary Objectives
- Assess improvement in Overall Survival (OS)
- Evaluate Progression-Free Survival (PFS)
- Analyze incidence and severity of adverse events
- Determine overall benefit–risk profile

---

## 📊 3. Data Sources

The analysis used the following datasets:

- **ADSL**: Subject-level dataset (demographics, treatment assignment)
- **ADTTE**: Time-to-event dataset (OS and PFS)
- **ADAE**: Adverse events dataset (safety outcomes)

---

## 🔬 4. Methodology

### 4.1 Efficacy Analysis

- Kaplan–Meier method used for survival estimation
- Log-rank test used to compare treatment groups
- Cox proportional hazards model used to estimate hazard ratios (HR)
- PFS model adjusted for covariates (Age and Sex)

---

### 4.2 Safety Analysis

Safety evaluation included:

- Total number of adverse events
- Number of patients with ≥1 adverse event
- Distribution by severity (Mild, Moderate, Severe)
- Identification of most common adverse events

---

## 📈 5. Results

### 5.1 Overall Survival (OS)

- Hazard Ratio (HR): **0.44**
- ~56% reduction in risk of death
- Log-rank test: **p = 0.01 (statistically significant)**

**👉 Interpretation:**  
The treatment provides a clinically meaningful survival benefit.

---

### 5.2 Progression-Free Survival (PFS)

- Hazard Ratio (HR): **0.85**
- p-value: **0.56 (not statistically significant)**
- Adjustment for age and sex: no major impact

**👉 Interpretation:**  
No statistically significant improvement in progression-free survival.

---

### 5.3 Safety Analysis

#### 5.3.1 Overall Adverse Events

| Group     | Total AEs | Patients with AE |
|----------|----------|------------------|
| Placebo  | 103      | 47               |
| Treatment| 97       | 47               |

**👉 Interpretation:**  
No increase in overall adverse events in the treatment group.

---

#### 5.3.2 Severity of Adverse Events

- Severe: Placebo (33) vs Treatment (30)
- Moderate: Similar between groups
- Mild: Slightly lower in treatment group

**👉 Interpretation:**  
No increase in severe adverse events.

---

#### 5.3.3 Common Adverse Events

Most frequent AEs:

- Headache  
- Fatigue  
- Nausea  
- Neutropenia  

**👉 Interpretation:**  
No unexpected safety signals observed.

---

## ⚖️ 6. Benefit–Risk Assessment

- Significant improvement in Overall Survival
- No significant change in Progression-Free Survival
- Comparable safety profile across groups

**👉 Overall Assessment:**  
The treatment demonstrates a favorable benefit–risk profile.

---

## 🧠 7. Discussion

The treatment improves overall survival in oncology patients.  
However, the lack of PFS improvement suggests survival benefit may not be directly linked to disease progression delay.  

Importantly, the safety profile remains comparable to placebo, supporting treatment tolerability.

---

## ✅ 8. Conclusion

The treatment demonstrated:

- Significant improvement in Overall Survival
- Comparable safety profile vs placebo
- No significant effect on Progression-Free Survival

**👉 Final Conclusion:**  
The overall benefit–risk profile supports the clinical value of the treatment.

---

## 🛠️ 9. Tools and Programming

- **R Programming Language**
- Packages:
  - `survival`
  - `dplyr`
  - `haven`

---

## 💻 10. Project Files

- `oncology_analysis.Rmd` → Full reproducible analysis  
- `oncology_analysis.md` → GitHub-rendered report  
- `oncology_analysis_files/` → Plots and figures  

---

## 🚀 Author

**Najiya K**
