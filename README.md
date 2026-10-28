# Statistical Tests

A small, self-contained learning repository of the statistical tests most commonly used in data science, A/B testing, and model comparison.

The goal is not just to show the library call, but to explain *why* the test is appropriate, what the hypotheses mean, and how to interpret the result in practice.

---

## Repository structure

```text
statistical_tests/
├── chi_square_test.ipynb      # Chi-square test of independence (SciPy)
├── independent_t_test.ipynb   # Independent two-sample t-test (SciPy)
├── paired_t_test.ipynb        # Paired t-test (SciPy)
├── z_test.ipynb               # Two-proportion z-test (statsmodels)
```
---

## Tests covered

| Test | Question answered | Data type | Library |
| --- | --- | --- | --- |
| **Two-proportion z-test** | Did the conversion/churn rate differ between two groups? | Two proportions | `statsmodels.stats.proportion.proportions_ztest` |
| **Chi-square test of independence** | Are two categorical variables associated? | Contingency table of counts | `scipy.stats.chi2_contingency` |
| **Independent two-sample t-test** | Do two independent groups have different means? | Continuous outcome, different subjects | `scipy.stats.ttest_ind` |
| **Paired t-test** | Is the average within-subject difference different from zero? | Continuous outcome, same subjects/models | `scipy.stats.ttest_rel` |

---

## Notebooks

### `z_test.ipynb` — Two-proportion z-test

Compares churn proportions between a treatment group (fast delivery) and a control group (standard delivery), using 10,000 users per group.

```python
successes = np.array([4200, 4800])  # churned users
nobs      = np.array([10000, 10000])  # users per group

z_stat, p_value = proportions_ztest(
    successes,
    nobs,
    alternative="smaller"
)
```

`alternative="smaller"` encodes the one-sided hypothesis that the treatment churn proportion is lower than control. The notebook prints the z-statistic and p-value, then applies the decision rule at `alpha = 0.05`.

### `chi_square_test.ipynb` — Chi-square test of independence

Tests whether product preference (A, B, C) is associated with gender (Male, Female) using a 2×3 contingency table.

```python
observed = np.array([
    [50, 30, 20],
    [30, 40, 30],
])

chi2_stat, p_value, dof, expected = stats.chi2_contingency(observed)
```

The code prints the chi-square statistic, degrees of freedom, p-value, and an automatic hypothesis-test conclusion.

### `independent_t_test.ipynb` — Independent two-sample t-test

Compares task-completion time between a control and a treatment system using separate groups of users.

```python
t_stat, p_value = stats.ttest_ind(
    treatment,
    control,
    equal_var=True,       # Student's t-test (pooled variance)
    alternative="less"    # H1: treatment mean < control mean
)
```

The notebook also demonstrates the production-friendly alternative:

```python
t_stat, p_value = stats.ttest_ind(
    treatment,
    control,
    equal_var=False,      # Welch's t-test, no equal-variance assumption
    alternative="less"
)
```

Welch's test is generally recommended for real A/B tests because it does not assume equal variances.

### `paired_t_test.ipynb` — Paired t-test

Compares two regression models (Random Forest vs. Linear Regression) on the same validation observations by testing the per-observation absolute errors.

```python
absolute_errors[model] = np.abs(y_val - predictions[model])

t_stat, p_value = stats.ttest_rel(
    competitor_errors,
    best_errors
)
```

Because both models predict the same observations, the errors are naturally paired. The test asks whether the mean difference in absolute error is statistically distinguishable from zero.

---

## Requirements

The notebooks use Python 3.11 and the following packages:

```text
numpy
pandas
scipy
statsmodels
scikit-learn
jupyter
```

Install them with:

```bash
pip install numpy pandas scipy statsmodels scikit-learn
```
---

## Interpreting results

All notebooks follow the same decision framework:

- Choose a significance level, typically `α = 0.05`.
- If `p < α`, **reject the null hypothesis**.
- If `p ≥ α`, **fail to reject the null hypothesis** (never conclude that the null is true).
- Report both **effect size** (e.g., seconds saved, error reduction, percentage-point difference) and **statistical significance**. A statistically significant result is not automatically business-significant.
- Match the test to the data structure:
  - Different subjects, continuous outcome → independent t-test.
  - Same subjects/observations, continuous outcome → paired t-test.
  - Two proportions → two-proportion z-test.
  - Two categorical variables → chi-square test of independence.
