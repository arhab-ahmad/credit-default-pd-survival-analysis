# Credit Default (Probability of Default) Modelling with Macroeconomic Variables

Predicting **when** a borrower is likely to default, using discrete-time survival analysis, Logistic Regression, Random Forest and Gradient Boosting in **R**.

![R](https://img.shields.io/badge/R-276DC3?logo=r&logoColor=white) ![Domain](https://img.shields.io/badge/Domain-Credit%20Risk-1F3864) ![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## 1. Business problem

Lenders such as banks and NBFCs must estimate how likely a borrower is to default, not only in the next 12 months but **over the whole remaining life of the loan**. Under IFRS 9 (the Expected Credit Loss framework), this estimate is called the **Probability of Default (PD)** and it must also include forward-looking economic conditions such as GDP growth and unemployment.

This project asks two questions:

1. Which borrower, loan and economic factors raise or lower the risk of default, and how does risk change as a loan ages?
2. Which model predicts default best on loans it has never seen: a simple, explainable Logistic Regression, or the more complex Random Forest and Gradient Boosting models?

## 2. Data

| Item | Value |
|---|---|
| Loans | 49,982 |
| Loan-month records (person-period rows) | 622,215 (after removing 270 rows with missing values) |
| Defaults | 15,149 |
| Default rate (share of records) | 2.435% (rare event) |
| Source | Rösch and Scheule (2020) |

**Variables used:** loan-to-value ratio (LTV), interest rate, house price index (HPI), GDP growth, unemployment rate (UER), credit score at origination (FICO) and loan age.

> The raw dataset is **not included** in this repository. Please obtain it from the original source cited above and place it in the `data/` folder.

## 3. Approach

1. **Person-period format:** each loan gets one row for every month it is observed, with `event = 1` in the month it defaults and `0` otherwise. This lets default be modelled as a risk that develops over time (the discrete-time hazard framework of Singer and Willett, 2003).
2. **Left truncation (delayed entry):** many loans were already old when observation began. Loan age is used as the time axis, and each loan is counted only at the ages at which it was actually observed. This avoids survivorship bias.
3. **Models**
   - Discrete-time logistic hazard model (`glm`): the explainable benchmark, with five loan-age bands (0-12, 12-24, 24-36, 36-60, 60-120 months).
   - Random Forest (`ranger`): 300 trees, 3 variables per split, minimum node size 50.
   - Gradient Boosting (`gbm`): 300 trees, interaction depth 3, learning rate 0.05, bag fraction 0.7.
4. **Evaluation:** 10-fold cross-validation with folds formed **by loan ID**, so no loan appears in both the training and test sets. Metrics: ROC-AUC, PR-AUC and log-loss, chosen because defaults are rare.

## 4. Results

Ten-fold, loan-level cross-validation (mean over folds):

| Model | ROC-AUC | PR-AUC | Log-loss |
|---|---|---|---|
| Logistic (discrete-time hazard) | 0.7314 | 0.0547 | 0.1076 |
| Random Forest | 0.7376 | 0.0623 | 0.1075 |
| **Gradient Boosting** | **0.7544** | **0.0642** | **0.1045** |

- Gradient Boosting was best on all three metrics and beat the logistic model in every one of the 10 folds.
- PR-AUC is about 2.2 to 2.6 times the 2.4% base rate, so the models are meaningfully better than random at finding defaulters.

### Key drivers of default (Logistic model, odds ratios)

| Factor | Effect on default risk |
|---|---|
| Loan-to-value (LTV) | Higher LTV raises risk (a 10-point rise multiplies the odds by about 1.19) |
| Interest rate | Each 1-point rise raises the odds by about 13% |
| Credit score (FICO) | A 100-point higher score lowers the odds by about one third |
| GDP growth | Stronger growth lowers the odds by about 7% per unit |
| House price index | Small but statistically clear risk reduction |
| Unemployment rate | No independent effect once other variables are included |

- **Loan age:** default risk is highest in the first two years and then falls and flattens.
- **Most important variables (tree models):** LTV, interest rate and credit score, with the macroeconomic variables contributing least.

## 5. Conclusions

- The non-linear models give a **modest** gain in accuracy (about +0.023 ROC-AUC), so the simple logistic hazard model stays highly competitive.
- Because its coefficients can be explained directly, the logistic model suits **explaining decisions and regulatory reporting**, while Gradient Boosting is a useful **accuracy benchmark**.

## 6. Limitations

- Only moderate discrimination (ROC-AUC about 0.73 to 0.75), which is typical for credit default data, since much of the risk depends on events not recorded in the data.
- Variable importance shows how much a model relied on a variable, not the direction or size of its effect.
- Standard errors for the shared macroeconomic variables may be somewhat optimistic because all loans share the same economic conditions in a given month.

## 7. Repository structure

```
credit-default-pd-survival-analysis/
├── README.md
├── report/
│   └── Credit_Default_PD_Report.pdf
├── code/
│   ├── 01_data_preparation_and_truncation.R
│   └── 02_models_and_cross_validation.R
├── images/
│   └── hazard_by_loan_age.png
├── data/
│   └── README.md          (instructions to download the dataset)
└── .gitignore
```

## 8. How to run

1. Download the dataset from the source above and save it in `data/`.
2. Open R or RStudio and install the packages: `install.packages(c("ranger", "gbm", "pROC", "PRROC", "dplyr"))`.
3. Run `code/01_data_preparation_and_truncation.R`, then `code/02_models_and_cross_validation.R`.

## 9. Tools

R (`glm`, `ranger`, `gbm`), survival analysis (discrete-time hazard models), cross-validation, ROC-AUC / PR-AUC / log-loss.

## 10. Author

**Mohd Arhab Ahmad**: M.Sc. Statistics, Aligarh Muslim University
[LinkedIn](https://www.linkedin.com/in/arhab-ahmad/) | [GitHub](https://github.com/arhab-ahmad)
