# Patient Health & Lifestyle Demographics Dashboard

**Tool:** Power BI | **Dataset:** Synthetic Healthcare Data | **Pages:** 3 | **Records:** 6,500

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Project Objectives](#project-objectives)
3. [Methodology](#methodology)
4. [Key Pages in This Report](#key-pages-in-this-report)
    - [Demographics and risk overview](#Demographics-and-Risk-Overview)
    - [Organ damage risk drivers](#Organ-damaga-risk-drivers)
    - [Key Influencers](#Key-Influencers)
6. [Detailed Insights](#detailed-insights)
7. [Disclaimer](#disclaimer)
8. [Actionable Recommendations](#actionable-recommendations)
9. [Conclusion](#conclusion)
10. [How to View the Report](#how-to-view-the-report)

---

## Project Overview

This Power BI dashboard explores the relationship between patient demographics, lifestyle factors, and clinical risk indicators and their association with organ damage across a synthetic patient population of 6,500 records spanning five organ types: Heart, Kidney, Liver, Lungs, and Systemic(ie... general body system).

The project was built independently as a portfolio piece, with an original analytical layer. It demonstrates end-to-end dashboard development: from data cleaning and feature engineering through DAX measure development, multi-page report design, and AI-assisted visual analysis using Power BI's Key Influencers feature.

The dashboard is structured across three pages, moving from population-level description to risk factor analysis to machine-learning-assisted segment discovery.

---

## Project Objectives

- Profile the patient population across demographic and lifestyle dimensions
- Examine how smoking behavior, BMI, blood pressure, and cholesterol vary across age groups and gender
- Identify which factors are most strongly associated with organ damage
- Use Power BI's Key Influencers visual to surface patient profiles with the highest damage prevalence
- Present findings in a clear, accessible format suitable for both clinical and non-clinical audiences
- Document analytical decisions transparently, including the limitations of working with synthetic data

---

## Methodology

### Data Cleaning

The dataset was assessed for logical inconsistencies before analysis. Key issues identified and resolved:

| Issue | Records Affected | Action Taken |
|---|---|---|
| Never smokers with Years of Smoking > 0 | 1,847 | Reset to 0 |
| Current smokers with Cigarettes per Day = 0 | 1,267 | Imputed with median CPD of current smokers (17) |
| Current smokers with Years of Smoking = 0 | 18 | Imputed with median YOS of current smokers (25 years) |
| Former smokers with Years of Smoking = 0 | 14 | Retained: plausible short-term smokers |

Median values were used for imputation rather than means because the median is less sensitive to extreme values and better represents the typical smoker in this dataset.

### Feature Engineering

Five new variables were created to enrich the analytical layer:

**Age Group** → patients bucketed into 10-year bands (18–28 through 79+) for age-level trend analysis.

**BMI Category** → raw BMI values categorized using WHO classification standards:

| BMI Range | Category |
|---|---|
| < 18.5 | Underweight |
| 18.5 – 24.9 | Normal |
| 25.0 – 29.9 | Overweight |
| 30.0 – 34.9 | Obese Class I |
| 35.0 – 39.9 | Obese Class II |
| ≥ 40.0 | Obese Class III |

**Cholesterol Category** → cholesterol values categorized using standard clinical thresholds (assumed Total Cholesterol in mg/dL):

| Cholesterol (mg/dL) | Category |
|---|---|
| < 200 | Desirable |
| 200 – 239 | Borderline High |
| ≥ 240 | High |

**Risk Score** → a composite index summarizing cumulative risk exposure. Each qualifying condition contributes 1 point (range: 0–6):

| Risk Factor | Scoring Condition |
|---|---|
| Smoking | Current smoker |
| BMI | Obese Class I or above (BMI ≥ 30) |
| Blood Pressure | High BP Risk |
| Cholesterol | High (≥ 240 mg/dL) |
| Family History | Yes |
| Alcohol Consumption | High |

**Risk Category** → Risk Score grouped into three tiers for dashboard readability:

| Risk Score | Category |
|---|---|
| 0 – 1 | Low Risk |
| 2 – 3 | Medium Risk |
| 4 – 6 | High Risk |

> **Note:** Risk Category was excluded from the Key Influencers analysis to avoid circular reasoning, as it was derived from the same predictor variables used in the model.

### DAX Measures

**vs Avg Age** → directional KPI comparator showing current filtered average age against the overall average.
```dax
vs Avg Age =
VAR _CurrentAge = AVERAGE(health_dataset[Age])
VAR _OverallAge = CALCULATE(AVERAGE(health_dataset[Age]), ALL(health_dataset))
VAR _Diff = _CurrentAge - _OverallAge
RETURN
SWITCH(
    TRUE(),
    _Diff > 0, UNICHAR(9650) & " " & FORMAT(_CurrentAge, "0.0"),
    _Diff < 0, UNICHAR(9660) & " " & FORMAT(_CurrentAge, "0.0"),
    FORMAT(_CurrentAge, "0.0")
)
```

**vs Avg BMI** → directional KPI comparator showing current filtered average BMI against the overall average.
```dax
vs Avg BMI =
VAR _CurrentBMI = AVERAGE(health_dataset[BMI])
VAR _OverallBMI = CALCULATE(AVERAGE(health_dataset[BMI]), ALL(health_dataset))
VAR _Diff = _CurrentBMI - _OverallBMI
RETURN
SWITCH(
    TRUE(),
    _Diff > 0, UNICHAR(9650) & " " & FORMAT(_CurrentBMI, "0.0"),
    _Diff < 0, UNICHAR(9660) & " " & FORMAT(_CurrentBMI, "0.0"),
    FORMAT(_CurrentBMI, "0.0")
)
```

**Rate of Damage** → primary outcome measure. The numerator is fixed to Organ_Condition = "Damaged" while the denominator remains context-sensitive, meaning the measure always reflects damage prevalence within whatever patient subgroup is currently in view.
```dax
Rate of Damage =
DIVIDE(
    CALCULATE(COUNTROWS(health_dataset), health_dataset[Organ_Condition] = "Damaged"),
    COUNTROWS(health_dataset)
)
```

**Family History Damage Rate** → conditional damage rate restricted to patients flagged with family history risk. Both numerator and denominator are anchored to Family_History_Risk = "Yes".
```dax
Family History Risk =
DIVIDE(
    CALCULATE(COUNTROWS(health_dataset),
        health_dataset[Organ_Condition] = "Damaged",
        health_dataset[Family_History_Risk] = "Yes"),
    CALCULATE(COUNTROWS(health_dataset),
        health_dataset[Family_History_Risk] = "Yes")
)
```
**High_Risk%**
```dax
High_risk % = DIVIDE(COUNTROWS(FILTER(health_dataset,health_dataset[Risk_category] = "High_Risk")),
                COUNTROWS(ALL(health_dataset)))
```
**Medium_Risk**
```dax
Medium_risk % = DIVIDE(COUNTROWS(FILTER(health_dataset,health_dataset[Risk_category] = "Medium_Risk")),
                COUNTROWS(ALL(health_dataset)))
```
**Low_Risk%**
```dax
Low_risk % = DIVIDE(COUNTROWS(FILTER(health_dataset,health_dataset[Risk_category] = "Low_Risk")),
                COUNTROWS(ALL(health_dataset)))
```
**% population with high cholesterol**
```dax
High Cholesterol = DIVIDE(CALCULATE(COUNTROWS(health_dataset),health_dataset[Cholesterol_category] = "High"),
                   CALCULATE(COUNTROWS(health_dataset)))
```

### Calculated Columns
**Risk Score**
```dax
Risk_Score = 
VAR SmokingRisk =
    IF(health_dataset[Smoking_Status] = "Current", 1, 0)

VAR BPRisk =
    IF(health_dataset[BP_Risk] = "High", 1, 0)

VAR CholRisk =
    IF(health_dataset[Cholesterol_Level] >= 240, 1, 0)

VAR FamilyRisk =
    IF(health_dataset[Family_History_Risk] = "Yes", 1, 0)

VAR AlcoholRisk =
    IF(health_dataset[Alcohol_Consumption] = "High", 1, 0)

VAR BMIRisk =
    IF(
         health_dataset[BMI_Category] = "Obese Class I" ||
        health_dataset[BMI_Category] = "Obese Class II" ||
        health_dataset[BMI_Category] = "Obese Class III",
        1,
        0
    )

RETURN
SmokingRisk + BPRisk + CholRisk + FamilyRisk + AlcoholRisk + BMIRisk
```

**Risk_Category**
```dax
Risk_category = IF([Risk_Score] <=1,"Low_Risk",
                IF([Risk_Score] <= 3, "Medium_Risk",
                "High_Risk"
                ))
```
---

## Key Pages in This Report

### Page 1 → Demographics & Risk Overview

*53% of patients have a smoking history; high BP peaks in middle age and BMI is uniformly elevated across all age groups*

This page establishes who the patients are, their smoking behavior, BMI distribution, gender breakdown, and blood pressure risk profile. It is the descriptive foundation of the dashboard, answering population-level questions before moving into outcomes.

**Visuals on this page:**
- Total Patients KPI with vs Avg Age and vs Avg BMI comparators
- Avg BMI Across Age Groups (clustered column chart)
- Avg Smoking Duration & Daily Intake across age groups (line chart)
- Smoking Status Distribution Among Patients (donut chart)
- Smoking Status by Gender (Ribbon chart)
- BP Risk Distribution Across Age Groups (100% stacked bar chart)

**Organ toggle:** Users can switch between the Damaged and Healthy patient views using the toggle in the header, dynamically filtering all visuals on the page.

---

### Page 2 — Organ Damage Risk Drivers

*Organ damage affects 42% of patients uniformly across age, organ type, and BMI, with underweight patients and males showing the highest damage rates*

This page shifts from description to analysis, examining organ damage rates across multiple risk dimensions to identify which factors are and are not meaningfully associated with damage outcomes.

**Visuals on this page:**
- KPI cards: Total Damage Rate, Family History Damage Rate, % Population at High Risk, % Population at Medium Risk
- Rate of Damage by Gender (donut chart)
- Organ Damage Rate by Smoking Status and Blood Pressure Risk (matrix)
- Organ Damage vs BMI Category (bar chart)
- Organ Damage Across Age Groups (clustered column chart with average reference line)
- Rate of Damage by Organs (stacked bar chart)

---

### Page 3 — Key Influencers & Risk Factor Combinations

*Organ damage affects 42% of patients with no clear link to age, cholesterol, or family history. male gender, underweight BMI, and high alcohol consumption emerge as the strongest associated risk factors*

This page uses Power BI's built-in Key Influencers AI visual to move beyond single-variable analysis and identify combinations of risk factors associated with the highest organ damage prevalence. It answers the question: which type of patient is most likely to have a damaged organ?

**Visuals on this page:**
- KPI cards: Total Damage Rate, % Population with High Cholesterol, % Population at High Risk, % Population at Medium Risk
- Combinations of Risk Factors Associated with Organ Damage (Key Influencers / Top Segments visual)
- Organ Damage vs Cholesterol Category (bar chart)
- Rate of Damage by Alcohol Consumption (donut chart)

**Top Segments identified:**

| Segment | Defining Characteristics | Damage Rate | Population |
|---|---|---|---|
| Segment 1 | Male, Obese Class III, non-moderate alcohol consumption | 51.3% | 437 |
| Segment 2 | Underweight BMI | 50.9% | 530 |
| Segment 3 | Male, Normal BMI, non-moderate alcohol consumption | 47.8% | 414 |

> **Analytical note:** A fourth segment identified by Power BI involved the Risk Category variable and was excluded from reporting. Because Risk Category is derived from other predictor variables already included in the model, retaining it would introduce circular reasoning into the analysis.

---

## Detailed Insights

> All findings below describe patterns within this synthetic dataset only and should not be interpreted as clinical evidence or medical conclusions.

### Smoking & Demographics (Page 1)

- Over half the patient population (53.52%) has a smoking history. 30.77% are current smokers and 22.75% are former smokers
- Male patients show consistently higher smoking rates across all categories, with the widest gap among current smokers (19% male vs 12% female)
- BMI is uniformly elevated across all age groups, ranging narrowly from 31 to 32. Every age group falls within the Obese Class I range with no meaningful age-related variation
- High BP peaks at 43–44% in the 39–48 and 49–58 age groups, with the youngest group (18–28) showing the lowest proportion at 33%
- Both cigarettes per day and years of smoking trend upward toward the 79+ age group, with the sharpest rise in cigarettes per day in the oldest cohort

### Organ Damage Outcomes (Page 2)

- The overall organ damage rate is 41.6% across the full patient population
- Organ damage is evenly distributed across all five organ types; Liver leads at 41%, Heart/Lungs/Kidney at 40%, Systemic at 37% — a spread of only 4 percentage points
- Age is not a predictor of organ damage in this dataset, rates fluctuate between 38.1% and 45.0% with no directional trend.
- Underweight patients show the highest damage rate at 50.9%, higher than every obesity class. Obesity shows no consistent upward association with organ damage
- Smoking status and BP risk show no linear interaction pattern. Damage rates cluster tightly between 39.7% and 46.6% across all nine combinations. Notably, Those who has Never smoked and with Low BP show the highest rate in the matrix at 46.6%
- Male patients experience organ damage at a higher rate than females, 43.8% vs 38.9%
- Only 7.94% of patients are classified as High Risk, despite 41.6% having organ damage. Suggesting that damage is concentrated in the large medium-risk population (55.95%) rather than among the most extreme risk cases

### Risk Factor Combinations (Page 3)

- Cholesterol category shows no meaningful association with organ damage, rates are virtually identical across Borderline High (42.4%), Desirable (41.6%), and High (41.3%). Counterintuitively, patients with the highest cholesterol show the lowest damage rate
- Family history damage rate (42.2%) is only 0.6 percentage points above the overall rate (41.6%), indicating that family history alone does not substantially elevate organ damage in this dataset
- High alcohol consumption is associated with the highest damage rate at 44.5%, though the pattern is non-linear. Low consumption (43.4%) exceeds Moderate (38.7%), which prevents a clean dose-response conclusion
- The three highest-risk patient segments all involve male gender and/or non-moderate alcohol consumption, reinforcing these as the most consistently associated factors across the dataset

### Why Some Findings Are Counterintuitive

Several findings in this dashboard challenge expected clinical patterns. Never smokers showing higher damage rates than current smokers in certain combinations, underweight patients having more organ damage than obese patients, high cholesterol patients showing lower damage than desirable cholesterol patients, and family history barely moving the needle. These results are most likely attributable to the way the data was generated rather than genuine health phenomena, and should be read in that context.

---

## Disclaimer

This project uses a **synthetic dataset** generated for analytical and learning purposes. The data does not represent real patients, real clinical records, or real health outcomes.

All findings, patterns, and insights presented in this dashboard are illustrative of analytical methodology only. They should not be used to draw medical conclusions, inform clinical decisions, or support any health-related claims.

Several findings in this report are counterintuitive when viewed against established clinical knowledge. This is expected behavior in synthetic data, which does not always replicate the complex interdependencies present in real-world health data.

---

## Actionable Recommendations

The following recommendations are framed as analytical directions for exploration in a real dataset. They are not clinical prescriptions.

**1. Investigate the medium-risk population more closely.**
With 55.95% of patients classified as Medium Risk and a total damage rate of 41.6%, the bulk of organ damage in this dataset occurs in patients who are not extreme outliers. In a real dataset, this would suggest that moderate multi-factor risk accumulation deserves as much attention as high-risk classification.

**2. Examine alcohol consumption patterns more granularly.**
The non-linear relationship between alcohol consumption and organ damage (High > Low > Moderate) warrants deeper investigation in a real dataset. Understanding what distinguishes Low from Moderate consumers could surface important confounding variables.

**3. Disaggregate the underweight finding.**
Underweight patients showing the highest damage rate (50.9%) is a striking result worth exploring further. In a real dataset, underweight status is often associated with chronic illness or malnutrition, both of which could independently elevate organ damage risk.

**4. Revisit family history as a composite variable.**
The near-zero incremental impact of family history on damage rate may reflect how family history was encoded in this dataset. In a real analytical context, family history is typically more informative when combined with the specific condition (e.g., family history of cardiovascular disease vs. liver disease) rather than as a single binary flag.

**5. Explore gender as a modifying variable.**
Male gender appears in two of the three highest-risk patient segments and shows a 5 percentage point higher damage rate overall. In a real dataset, gender-stratified analysis of individual risk factors would be a logical next step.

---

## Conclusion

This dashboard demonstrates that organ damage in this synthetic patient population does not follow a simple, single-factor risk model. No individual variable — age, cholesterol, BMI class, smoking status, or family history — shows a strong, clean association with damage outcomes on its own. Instead, damage is distributed broadly across the population, with combinations of moderate risk factors — particularly male gender, non-moderate alcohol consumption, and underweight BMI — emerging as the most consistently associated profiles.

The project showcases end-to-end Power BI development including data cleaning, feature engineering, DAX measure development, multi-page report design, and AI-assisted segment analysis. The analytical decisions, including what was excluded from the Key Influencers model and why, are documented to reflect a transparent and defensible approach to the work.

---

## How to View the Report

1. Download the `.pbix` file from this repository
2. Open it in **Power BI Desktop** (free download from Microsoft)
3. Use the **Damaged / Healthy toggle** in the report header to filter all visuals by organ condition
4. Use the **organ icons** in the left navigation to filter by organ type
5. Navigate between pages using the **Next Page / Previous Page** buttons or the page tabs at the bottom of the Power BI window
6. On Page 3, click individual segments in the **Top Segments** view to explore the defining characteristics of each patient profile

---

*Analysis by Shalom Norberts | Healthcare Data Analytics Portfolio*
*Tool: Power BI Desktop | Dataset: Synthetic*
