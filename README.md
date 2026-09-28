# Titanic Survival Analysis

**Who survived the Titanic disaster, and which factors were associated with survival?**

An end-to-end exploratory analysis of the Titanic passenger dataset (891 passengers with a recorded outcome). Instead of only counting survivors, the notebook works through a sequence of questions: data quality, missing-value handling, group comparisons with confidence intervals, confounding checks, and finally a multivariate logistic regression.

> **Note:** this is observational data from a single event. All findings are *associations*: they show who survived, not why.

---

## Table of Contents

- [Questions Explored](#questions-explored)
- [Dataset](#dataset)
- [Methods and Skills Demonstrated](#methods-and-skills-demonstrated)
- [Data Cleaning Decisions](#data-cleaning-decisions)
- [Key Findings](#key-findings)
- [Getting Started](#getting-started)
- [Repository Structure](#repository-structure)
- [Limitations and Next Steps](#limitations-and-next-steps)

---

## Questions Explored

1. **Data quality:** what is missing, and is the missingness random?
2. **Overall survival:** who survived, and how did **sex** and **passenger class** matter, alone and together?
3. **Age:** were children treated differently, and does missing age data bias the picture?
4. **Fare and class:** does fare add information beyond class?
5. **Family:** is travelling alone, or in a large family, related to survival?
6. **Port of embarkation:** is the apparent port effect real, or a class effect in disguise?
7. **Multivariate view:** which factors still matter when considered together?

## Dataset

The notebook uses the Kaggle Titanic **train** file (891 rows, 12 columns).

| Column | Meaning |
|---|---|
| `PassengerId` | Row identifier |
| `Survived` | Target: 1 = survived, 0 = did not |
| `Pclass` | Ticket class (1 = upper, 2 = middle, 3 = lower) |
| `Name`, `Sex`, `Age` | Passenger details |
| `SibSp` | Number of siblings / spouses aboard |
| `Parch` | Number of parents / children aboard |
| `Ticket`, `Fare`, `Cabin` | Ticket number, fare paid, cabin |
| `Embarked` | Port of embarkation (C = Cherbourg, Q = Queenstown, S = Southampton) |

**Loading:** the notebook first looks for a local file named `train - train.csv`, `train.csv` or `titanic.csv` in the working directory. If none is found, it falls back to a public mirror of the same dataset (internet connection required).

## Methods and Skills Demonstrated

- Missing-value analysis, including testing whether missingness is random
- Group-wise imputation (`groupby().transform`)
- Feature engineering: `Title`, `FamilySize`, `IsAlone`, `HasCabin`, `Deck`, `TicketGroupSize`, `FarePerPerson`
- `groupby` / pivot analysis with **95% Wilson confidence intervals**
- Hypothesis tests: chi-square and Mann-Whitney U
- Confounding checks (port vs class, fare vs class)
- Logistic regression reported as odds ratios (`statsmodels`)
- Visualisation with `matplotlib` and `seaborn`

## Data Cleaning Decisions

| Issue | Handling |
|---|---|
| `Embarked`: 2 missing | Filled with the most common port **within the passenger's class**. (A plain `fillna(df["Embarked"].mode())` only fills the first row because `mode()` returns a Series aligned on the index. The notebook corrects this.) |
| `Age`: 177 missing (19.9%) | Raw column kept. A separate `AgeFilled` column uses the **median age of the passenger's title group within class**. Age-specific charts use recorded ages only; `AgeFilled` is used only where a complete column is required (the regression). |
| `Cabin`: 687 missing (77.1%) | Not imputed. Replaced with `HasCabin` and `Deck` features. |
| 15 passengers with `Fare = 0` | Kept and flagged, since the data cannot confirm whether they are errors or genuine special tickets. |
| Very high fares (up to 512.3) | Kept, as they are real first-class fares. |
| Shared tickets | Fares repeat for each holder of a ticket, so a per-person fare (`FarePerPerson`) is derived. |

## Key Findings

1. **Sex and class dominate.** 342 of 891 passengers survived (38.4%). Women survived at 74% versus 19% for men; survival fell from 63% (1st class) to 47% (2nd) to 24% (3rd).
2. **Sex and class interact.** 97% of first-class women survived versus 14% of third-class men. Among women, class made a large difference (third-class women: 50%); among men, only first class stood out (37% vs 14-16%).
3. **Children were favoured.** Children aged 12 or under survived at 58% (boys and girls alike), but this varied by class (100% in 2nd class, 42% in 3rd).
4. **Family size is non-linear.** Groups of 2-4 survived best (58%), solo travellers 30%, and groups of 5 or more only 16%, driven mostly by third-class families.
5. **The port effect is mostly a class effect.** Cherbourg's higher survival reflects its first-class-heavy mix. A residual gap remains only in third class (Southampton 19% vs 38% at Cherbourg and Queenstown).
6. **Fare adds information only in the upper classes.** Within first class survival rises with fare; in third class it is flat.
7. **Missing data is not random.** Age is missing for 27.7% of third-class passengers versus 6.0% in second class and 13.9% in first class, so dropping those rows would bias results. Cabin availability is itself largely a class marker.
8. **Adjusted view (logistic regression, McFadden pseudo-R² = 0.37).** Sex, class, age and large family size remain associated with survival; port of embarkation and travelling alone do not.

| Factor (holding others constant) | Odds ratio | 95% CI |
|---|---|---|
| Male | 0.06 | 0.04 - 0.09 |
| Family of 5+ | 0.06 | 0.02 - 0.16 |
| 3rd class (vs 1st) | 0.16 | 0.07 - 0.35 |
| 2nd class (vs 1st) | 0.42 | 0.21 - 0.83 |
| Age (per 10 years) | 0.78 | 0.65 - 0.94 |
| Child (12 or under) | 6.81 | 2.71 - 17.09 |

## Getting Started

### Prerequisites

- Python 3.9+
- Jupyter Notebook or JupyterLab
- Recent versions of `pandas` (2.1+ recommended), `numpy`, `matplotlib`, `seaborn`, `scipy` and `statsmodels`

### Installation

```bash
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>
pip install pandas numpy matplotlib seaborn scipy statsmodels jupyter
```

### Run

```bash
jupyter notebook Titanic_Survival_Analysis.ipynb
```

Then run all cells (**Kernel → Restart & Run All**). To work offline, place `train.csv` (or `titanic.csv`) in the same folder as the notebook.

## Repository Structure

```
.
├── Titanic_Survival_Analysis.ipynb   # Full analysis
├── README.md
└── titanic.csv                        # Optional local copy of the dataset
```

## Limitations and Next Steps

**Limitations**

- **Association, not causation.** Social norms, cabin location and lifeboat access are not recorded in the data.
- **Sample size.** 891 passengers (the Kaggle training split), so some groups are small; confidence intervals are shown for that reason.
- **Imputation.** `AgeFilled` is an estimate, and the regression's child indicator relies partly on it.
- **Model.** The regression has no interaction terms, even though the sex × class interaction is clear. Class and fare are correlated (Spearman -0.69), so the fare estimate is unstable.
- **Fare ambiguity.** `FarePerPerson` is a reasonable adjustment but is not confirmed by the data.

**Next steps**

- Build a cross-validated predictive model and evaluate it on the held-out test file
- Analyse surname / family groups
- Add sex × class interaction terms to the regression

## Data Source

Titanic dataset from [Kaggle: Titanic - Machine Learning from Disaster](https://www.kaggle.com/c/titanic).
