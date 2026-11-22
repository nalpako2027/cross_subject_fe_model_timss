# <center> ✨ Clearer Instruction, Bigger Learning Gains? Student Fixed-Effects Evidence on SES Differences from Two-Wave Multi-country Panel Data (TIMSS 2023 Longitudinal, Grades 8-9) </center>

> **⚠️ Status: Research Plan (pre-analysis draft)**

---

## 1. 📝 Summary

This project estimates how changes in students' perceived instructional clarity relate to changes in their mathematics and science achievement over one school year, and whether that relationship differs by socioeconomic status (SES). It uses the TIMSS 2023 Longitudinal panel (Grade 8 → Grade 9) with student fixed-effects (first-difference) models, plus a cross-subject comparison, to get closer to causal estimates than cross-sectional studies allow.

---

## 2. 🎯 Objectives

### Business case and motivation

Organizations constantly ask a version of the same question: *does better guidance (coaching, onboarding, manager clarity, customer-success support) improve performance, and for whom?* The naive answer is confounded, because the people who receive better guidance usually differ in ability, motivation, and resources. This project shows a reusable analytical pattern for that problem: **compare each unit with itself**, remove everything stable about the unit, and then remove shocks shared across its two "products" (here, two school subjects). The same design applies to employee, customer, or store-level panel data, for example one employee under two managers, or one customer using two product lines.

The policy reading is direct. If clarity gains raise achievement more for low-SES students, improving instructional clarity is an equity lever that is cheaper than structural reform.

### Research questions

| # | Question | Predictor | Outcome | Moderator |
|---|----------|-----------|---------|-----------|
| RQ1 | Within students, are gains in perceived instructional clarity in a subject associated with larger achievement growth in that subject? | Change in perceived instructional clarity (math, science) | Change in achievement (math, science) | none |
| RQ2 | Is that association stronger for lower-SES students? | Same as RQ1 | Same as RQ1 | Baseline (2023) SES |
| RQ3 | Does the association differ across Jordan, Korea, and Sweden? | Same as RQ1 | Same as RQ1 | Country (separate models) |
| RQ4 | Does 2023 clarity predict 2024 achievement beyond 2023 achievement (temporal ordering)? | 2023 instructional clarity | 2024 achievement | Baseline SES |
| Exploratory | Does baseline digital self-efficacy change the SES pattern in RQ2? | Same as RQ1 | Same as RQ1 | Baseline SES × baseline digital self-efficacy |

### Hypotheses

- **H1.** Within-student clarity gains are positively associated with achievement growth in the same subject.
- **H2.** The association is larger for lower-SES students, following the argument that instructional quality can have differential effects by SES (Gustafsson et al., 2018).
- **H3 (exploratory).** Digital self-efficacy is not assumed to moderate clarity effects; it is tested as exploratory because theory is thin and its distribution shows ceiling effects at Grade 8 (see References).

### Technical goals

1. Build a student × subject × wave panel from TIMSS-L, with plausible values and replicate weights handled correctly and missing data imputed and diagnosed using my MICE-PMM diagnostics package: [A Diagnostics Package for Multiple Imputation by Chained Equations (using PMM)](https://github.com/nalpako2027/mice_imputation_diagnostics_helper.git).
2. Estimate a ladder of fixed-effects specifications and report what each added layer of control changes.
3. State, test where possible, and bound the strict exogeneity assumption instead of assuming it.

---

## 3. 📂 Dataset

- **Source:** TIMSS 2023 Longitudinal International Database (IEA / TIMSS & PIRLS International Study Center, Boston College). Students were assessed in Grade 8 (2023) and Grade 9 (2024) with the same achievement instruments, using booklets assigned so that students did not see the same items twice (Fishbein et al., 2026).
- **Countries:** Jordan, Republic of Korea, Sweden (the three Grade 8–9 participants). Analyses run **separately by country**, because replicate-weight structures differ.
- **Link:** <https://timss2023.org/longitudinal/data>
- **Files used:** student achievement and student context and school files (2023 and 2024 variables in one record).
- **Existing pipeline:** [`To be added!`] notebooks `01_merge_and_rweights`, `02_multicountry_data_cleaning_imputation`, `03_fixed_effects_timss`. The Swedish sample in the current pipeline has 5,303 students `[UPDATE: confirm N per country after science variables are added]`.

### Variables

| Role | Construct | Variable(s) | Notes |
|------|-----------|-------------|-------|
| Outcome | Math achievement | 2023: `BSMMAT11–15`; 2024: `BSMMAT21–25`; change: `BSMCHG01–05` | Five plausible values per wave; use the official change PVs for change models |
| Outcome | Science achievement | 2023: `BSSSCI11–15`; 2024: `BSSSCI21–25`; change: `BSSCHG01–05` | Same handling as math |
| Predictor | Perceived instructional clarity, math lessons | `<codebook name, 2023 and 2024>` | Rasch scale score on a common cross-wave scale (per technical report) |
| Predictor | Perceived instructional clarity, science lessons | `<codebook name, 2023 and 2024>` | **Cross-country comparability differs; see Limitations** |
| Moderator | SES | baseline 2023 `ses` (from existing pipeline) | Time-invariant by construction |
| Moderator (exploratory) | Digital self-efficacy | baseline 2023 `digseff` | Used only as a time-invariant moderator |
| Controls | Liking, confidence, and disorderly behavior, per subject | `<codebook names>` | Subject-specific; verify availability for science |
| Design | Sampling weights | `TOTWGT`; JK2 replicate weights | Defined per country |

### Filtering and preparation

- Keep students with valid achievement and clarity data in **both subjects and both waves**; report attrition by SES.
- Standardize clarity per subject using a **pooled-wave mean and SD**, so that wave-to-wave change is preserved.
- Missing data handled with the MICE-PMM pipeline, with a complete-case sensitivity check (see Limitations).

---

## 4. 🔧 Methodology

### 4.1 Pipeline

1. Merge and weight (notebook 01); clean and impute (notebook 02); add science clarity and science achievement.
2. Reshape to long format: one row per **student × subject × wave** (up to four rows per student).
3. Standardize predictors as above; keep SES and digital self-efficacy at their 2023 values.
4. Estimate the specification ladder below for each country, each plausible value, and each imputed dataset; combine with Rubin's rules and replicate-weight variance.

### 4.2 Specification ladder

Let *i* be a student, *s* ∈ {math, science}, *t* ∈ {2023, 2024}; *A* is achievement, *C* is perceived clarity, *S_i* is baseline SES, and Δ means 2024 minus 2023.

**M1, within student-subject, first-difference model** ("does a change in clarity predict a change in achievement?")

$$\Delta A_{is} = \beta\,\Delta C_{is} + \delta\,(\Delta C_{is}\times S_i) + \theta\,S_i + \tau_s + \varepsilon_{is}$$

Differencing removes everything stable about the student and the student's stable subject-specific traits. With two waves, fixed effects and first differences are equivalent. The term θ·S_i lets growth differ by SES, so the interaction is not credited with differential trends. τ_s captures average growth by subject.

**M2, cross-subject model** ("is the subject taught more clearly the subject where the student does better?")

$$A_{ist} = \beta\,C_{ist} + \delta\,(C_{ist}\times S_i) + \alpha_{it} + \mu_{st} + \varepsilon_{ist}$$

The student-by-wave effect α_it removes ability, motivation, family background, school, mood, response style, and any wave-specific shock shared across subjects. Identification comes only from the math-versus-science contrast within a student and wave. μ_st absorbs subject-by-wave differences, such as a change in how the science scale is built.

**M3, double-difference model** (strictest; the preferred estimate if variation permits)

$$\Delta\big(A_{i,\text{math}}-A_{i,\text{sci}}\big)=\beta\,\Delta\big(C_{i,\text{math}}-C_{i,\text{sci}}\big)+\delta\,(\cdots\times S_i)+\varepsilon_i$$

This removes student-by-wave shocks and stable student-by-subject traits together. It asks whether a student whose math lessons became clearer *relative to* science also grew more in math *relative to* science.

**Decision rule:** report M1, M2, and M3 side by side; use M3 as the headline estimate if the within-student variation check (Table below) shows enough usable change, otherwise M2 and M1 with the limitation stated.

**M4, robustness: temporal ordering (RQ4).** Cross-subject, lagged form:

$$A_{i,\text{math},2024}-A_{i,\text{sci},2024}=\rho\,(A_{i,\text{math},2023}-A_{i,\text{sci},2023})+\beta\,(C_{i,\text{math},2023}-C_{i,\text{sci},2023})+\varepsilon_i$$

This gives time order (clarity measured before the outcome) while the subject contrast still removes stable student traits. The usual dynamic-panel bias (Nickell, 1981) concerns lagged outcomes combined with fixed effects over time; this form uses a subject contrast instead, but it is not formally established in the literature `[UPDATE: validate with a methodologist]`.

### 4.3 How the assumptions are handled

| Assumption | Why it can fail here | Handling |
|---|---|---|
| **Strict exogeneity**: Clarity ratings must be unrelated to past, present, and future achievement shocks, conditional on the student effect (Wooldridge, 2010) | A student who has a bad term in math may rate math lessons as less clear (feedback / reverse causality); self-reports share method variance | (1) Design: the subject contrast removes shared moods and shocks (Clotfelter et al., 2010; Rivkin & Schiman, 2015). (2) Feedback check: test whether 2023 subject achievement gaps predict the *change* in the clarity gap; a significant result is reported as evidence of feedback. (3) Alternative predictor: leave-one-out classmate mean of clarity if class identifiers are available in both waves (one student's shock barely moves a class mean). (4) Subject-specific controls (liking, confidence, disorderly behavior). (5) Results are presented as estimates that are causal under a stated, partly tested assumption, not as proven effects. The standard lead-of-regressor test needs three waves and is not possible with two. |
| **Sufficient within-person variation** | FE uses only students whose clarity changes; ceiling effects shrink it | Report the within-student SD of Δclarity and share of students with non-zero change, by country and SES group, before estimation |
| **No time-varying omitted confounders** | A new teacher or class change can move clarity and achievement together | Not removable by FE; cross-subject contrast helps only if the shock is shared across subjects. Sensitivity analysis and explicit limitation |
| **Common subject trends** | Math and science may grow at different rates for reasons unrelated to clarity | Subject effects (τ_s, μ_st); SES-specific growth terms (θ·S_i); sensitivity adding baseline achievement |
| **Linearity and additivity** | Clarity effects may be non-linear | Re-estimate with categories of clarity change (decrease, none, increase) |
| **Classical measurement error** | First-differencing amplifies attenuation bias because signal falls faster than noise (Griliches & Hausman, 1986) | Use Rasch scale scores, report reliability, and interpret coefficients as conservative lower bounds |
| **Error structure and inference** | Students are clustered in schools; T = 2 allows no test of serial correlation | Design-based variance with JK2 replicate weights (TIMSS procedure); where model-based, cluster at school level (Abadie et al., 2023) |
| **Plausible values** | A single score is not a valid individual measure | Run each model with all five plausible values; combine with Rubin's rules; use official change PVs. The user guide states PVs are valid as outcomes in analysis but not as individual growth measures (Fishbein et al., 2026) |
| **Missing data** | Imputing one wave from the other pulls values together and shrinks the change FE relies on (van Buuren, 2018; Rubin, 1987) | Complete-case sensitivity analysis; report imputation diagnostics |
| **Scale comparability across waves and subjects** | Science questionnaires differ by country (see Limitations) | Country-specific models; subject-by-wave effects; sensitivity with the integrated-science countries only |

### 4.4 Reporting

- Effect sizes in standard-deviation units with confidence intervals; interaction plots by SES.
- Multiple-testing caution for the exploratory digital self-efficacy moderator; reported as exploratory.
- A reproducible specification table so a reader can see what each fixed-effects layer changes.

---

## 5. 📊 Key Findings & Model Performance



## 6. 🧩 Business Questions & Key Insights

- Does improving the clarity of guidance raise performance, and is the gain largest for the least-advantaged group? 
- Which signals can serve as a leading indicator of future performance, and how much of that signal is explained by reverse causality? 
- How would the same within-unit, cross-product design transfer to an employee, customer, or store panel? 

## 7. ⚙️ Requirements



## 8. ▶️ How to Reproduce



## 9. 🗂️ Project Structure



---

## 10. ⚠️ Limitations & Further Recommendations (preliminary)

1. **Strict exogeneity cannot be proven.** The cross-subject design reduces, but does not remove, feedback from achievement to perceived clarity. With two waves there is no lead-based test. Estimates should be read as within-student associations that approximate causal effects under a stated assumption.
2. **Self-reported predictor.** Clarity is the student's perception. The student-by-wave effect removes general response style, but not subject-specific mood or performance effects.
3. **Time-varying confounders.** A teacher or class change can affect both clarity and achievement; fixed effects do not remove it unless the change is shared across subjects.
4. **Science comparability.** Korea used integrated science in both years and Sweden used separate science subjects in both years, while Jordan changed from integrated (2023) to separate (2024) (Fishbein et al., 2026). Science clarity for Sweden requires an aggregation rule, and Jordan may be usable only in sensitivity analyses.
5. **Limited power in subgroups and interactions.** Two waves, three countries, and subgroup interactions limit precision, especially for any three-way interaction.
6. **Within-student variation may be small.** Ceiling effects and stable perceptions can leave few students whose clarity changes.
7. **Measurement error is amplified by differencing,** so coefficients may be attenuated.
8. **Imputation effects.** Wide-format MICE may shrink within-student change; mitigated by the complete-case check.
9. **Joint scaling of achievement.** Achievement was estimated jointly across both years using context information from both waves, which may build some correlation between changes in achievement and changes in context scales (Fishbein et al., 2025) `[UPDATE: confirm details in the technical report]`.
10. **External validity.** Three countries and one school year; results may not transfer to other systems or to private-sector settings.

**Further recommendations:** replicate on the Grade 4–5 cohort (nine countries) to test generalization; add a third wave if one becomes available, which would allow a lead-based exogeneity test; examine teacher change as an alternative, more external predictor using the student–teacher linkage files.

---

## 11. 📚 References


**Data and documentation**
- Fishbein, B., Taneva, M., & Kowolik, K. (2026). *TIMSS 2023 Longitudinal user guide for the international database*. Boston College, TIMSS & PIRLS International Study Center. https://timss2023.org/longitudinal/data
- Fishbein, B., et al. (2025). *TIMSS 2023 Longitudinal technical report*. Boston College, TIMSS & PIRLS International Study Center. https://timss2023.org/wp-content/uploads/methods/T23L_TechnicalReport.pdf
- von Davier, M., et al. (2025). *TIMSS 2023 Longitudinal international results in mathematics and science*. Boston College. https://timss2023.org/longitudinal/results/

**Fixed-effects design and cross-subject identification**
- Clotfelter, C. T., Ladd, H. F., & Vigdor, J. L. (2010). Teacher credentials and student achievement in high school: A cross-subject analysis with student fixed effects. *Journal of Human Resources, 45*(3), 655–681.
- Rivkin, S. G., & Schiman, J. C. (2015). Instruction time, classroom quality, and academic achievement. *Economic Journal, 125*(588), F425–F448.
- Wedel, K. (2021). Instruction time and student achievement: The moderating role of teacher qualifications. *Economics of Education Review*.
- Bietenbeck, J. (2014). Teaching practices and cognitive skills. *Labour Economics, 30*, 143–153.

**Panel-data assumptions and inference**
- Wooldridge, J. M. (2010). *Econometric analysis of cross section and panel data* (2nd ed.). MIT Press.
- Allison, P. D. (2009). *Fixed effects regression models*. SAGE.
- Griliches, Z., & Hausman, J. A. (1986). Errors in variables in panel data. *Journal of Econometrics, 31*(1), 93–118.
- Nickell, S. (1981). Biases in dynamic models with fixed effects. *Econometrica, 49*(6), 1417–1426.
- Abadie, A., Athey, S., Imbens, G. W., & Wooldridge, J. M. (2023). When should you adjust standard errors for clustering? *Quarterly Journal of Economics, 138*(1), 1–35.

**Missing data**
- Rubin, D. B. (1987). *Multiple imputation for nonresponse in surveys*. Wiley.
- van Buuren, S. (2018). *Flexible imputation of missing data* (2nd ed.). CRC Press.

**Substantive literature (instructional clarity, SES, digital self-efficacy)**
- Gustafsson, J.-E., Nilsen, T., & Hansen, K. Y. (2018). School characteristics moderating the relation between student socio-economic status and mathematics achievement in grade 8. Evidence from 50 countries in TIMSS 2011. Studies in Educational Evaluation, 57, 16–30. https://doi.org/10.1016/j.stueduc.2016.09.004
- Patsis, P., Yang Hansen, K., & Klapp, A. (2026). The role of instructional quality in explaining socioeconomic disparities in mathematics achievement: A multilevel structural equation modeling of Swedish TIMSS 2019 data. *Large-scale Assessments in Education*. https://link.springer.com/article/10.1186/s40536-026-00307-9
- Sideridis, G., & Alghamdi, M. (2026). Diagnosing ceiling effects and unstable nonlinearity in short ordinal scales: A TIMSS 2023 application. *Behavioral Sciences*. https://doi.org/10.3390/bs16091485

---

## 12. License

![License](https://img.shields.io/github/license/nalpako2027/telco_customer_churn_prediction_simulation?style=for-the-badge)

## 13. 👤 Author / Attribution
Orhan KAPLAN, PhD

