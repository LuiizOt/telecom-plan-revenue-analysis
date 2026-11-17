# Telecom Plan Revenue Analysis: Surf vs. Ultimate

## Project Overview

Megaline, a telecom operator, offers two prepaid plans, **Surf** and **Ultimate**, and its commercial department wants to know which one generates more revenue in order to adjust the advertising budget. Using 2018 usage data for 500 customers, this project cleans and joins five tables, rebuilds each customer's monthly bill from the plan rules, compares usage and revenue between plans, and tests two statistical hypotheses about average revenue.

**Key outcome:** usage is very similar across plans, Surf generates considerably more total revenue, and the mean monthly revenue differs significantly both between plans (p ≈ 2.9 × 10⁻¹⁴) and between the New York–New Jersey area and other regions (p ≈ 0.0064).

## Business Problem

Advertising budgets should be directed to the products that bring the most revenue. The two plans have very different price points and allowances, so it is not obvious which one is more valuable to the company: Ultimate charges a higher fixed fee, while Surf relies more on overage charges. Answering the question requires reconstructing actual monthly revenue per customer from raw call, message and data records.

## Objectives

1. Prepare the data: correct data types and join the usage records with customer and plan information.
2. Calculate the monthly revenue of each customer according to the rules of their plan.
3. Compare calls, messages, data usage and revenue between the two plans.
4. Test whether the mean revenue differs (a) between Surf and Ultimate users and (b) between users in the NY-NJ area and users in other regions, at α = 0.05.

## Dataset Description

Data for 500 Megaline customers in 2018, provided for educational purposes as part of the TripleTen Data Science program.

| File | Rows | Columns | Content |
|---|---|---|---|
| `megaline_users.csv` | 500 | 8 | `user_id`, `first_name`, `last_name`, `age`, `city`, `reg_date`, `plan`, `churn_date` |
| `megaline_calls.csv` | 137,735 | 4 | `id`, `user_id`, `call_date`, `duration` (minutes) |
| `megaline_messages.csv` | 76,051 | 3 | `id`, `user_id`, `message_date` |
| `megaline_internet.csv` | 104,825 | 4 | `id`, `user_id`, `session_date`, `mb_used` |
| `megaline_plans.csv` | 2 | 8 | Plan conditions (see below) |

**Plan conditions**

| Plan | Monthly fee | Minutes included | Messages included | Data included | Extra minute | Extra message | Extra GB |
|---|---|---|---|---|---|---|---|
| Surf | $20 | 500 | 50 | 15,360 MB (15 GB) | $0.03 | $0.03 | $10 |
| Ultimate | $70 | 3,000 | 1,000 | 30,720 MB (30 GB) | $0.01 | $0.01 | $7 |

**Data quality**

- `churn_date` has 466 missing values, which is expected: a missing date means the customer was still active.
- All date columns are stored as strings and are converted to `datetime`.
- Some calls have a duration of 0.00 minutes and some data sessions record 0.00 MB.

## Technologies and Libraries

- Python
- pandas, NumPy
- Matplotlib, seaborn
- SciPy (`scipy.stats`)
- Jupyter Notebook

## Methodology

1. **Data preparation:** each table is inspected with `info()` and sampled. Dates are converted to `datetime`, names are normalized to lowercase, and `plan_name` is renamed to `plan` so that it can be used as a join key.
2. **Monthly aggregation:** for each user and month, the notebook computes the number of calls, total call minutes, number of messages and total MB used. These tables are merged on `user_id` and month, resulting in 2,258 user-month records, which are then enriched with the plan conditions.
3. **Revenue calculation:** monthly revenue = plan fee + (minutes above the allowance × price per minute) + (messages above the allowance × price per message) + (MB above the allowance ÷ 1,024 × price per GB). Overage values are clipped at zero, and user-months without messages are filled with 0.
4. **Behavior analysis:** for each plan, the notebook compares average call duration by month (bar chart), monthly minutes (histograms, mean, variance and box plot), monthly messages (histograms), and monthly data usage (median, histograms and box plot).
5. **Revenue analysis:** total revenue per plan (bar chart), revenue distribution (histograms) and box plots.
6. **Hypothesis testing:** two-sample t-tests (`scipy.stats.ttest_ind`) with α = 0.05. A two-sided test was used, since the question is whether the means differ, not which one is higher.

## Results and Model Performance

This is a statistical analysis project, so no predictive model is trained.

**Monthly call minutes per plan**

| Plan | Mean | Variance |
|---|---|---|
| Surf | 412.10 | 47,001.25 |
| Ultimate | 410.18 | 50,510.63 |

**Hypothesis tests (α = 0.05)**

| Hypothesis | H₀ | p-value | Decision |
|---|---|---|---|
| Revenue differs between plans | Mean monthly revenue of Surf users = mean monthly revenue of Ultimate users | 2.91 × 10⁻¹⁴ | Reject H₀ |
| Revenue differs between NY-NJ and other regions | Mean monthly revenue of NY-NJ users = mean monthly revenue of other users | 0.0064 | Reject H₀ |

## Key Findings

- **Similar usage, very different allowances:** Surf and Ultimate users talk for practically the same number of minutes per month (about 410 minutes), far below Ultimate's 3,000-minute allowance and close to Surf's 500-minute limit.
- **Data usage is similar too:** median data usage differs by only about 1 GB between plans, and most users consume roughly 15–20 GB per month, which is around or above Surf's 15 GB allowance.
- **Surf brings more total revenue:** the sum of revenue across all user-months is considerably higher for Surf, even though its monthly fee is $50 lower.
- **Revenue differs by plan and by region:** both null hypotheses were rejected.

## Conclusions

**Achievement of objectives:** all four objectives were completed. The analysis indicates that Surf generates more total revenue, and that the Ultimate plan's large allowances are not used by a typical customer, which suggests that Ultimate's pricing could be revisited to increase adoption.

**Limitations**

- The total revenue comparison is affected by the number of customers on each plan; the mean revenue per user is tested statistically but is not reported as a value.
- The aggregation starts from the calls table, so user-months with messages or data sessions but no calls are excluded. The 15 user-months without data sessions get a missing revenue value and are dropped before the comparisons.
- Overage is calculated on exact minutes and fractional gigabytes. If the operator rounds call durations or monthly data up when billing, revenue would be underestimated.
- `ttest_ind` is used with its default `equal_var=True` (Student's t-test). Welch's t-test would be more robust, since equal variances between the groups were not verified.
- Some chart titles and axis labels are in Portuguese, and the messages histogram reuses the title "Minutos por mes" (minutes per month).

**Potential improvements:** report the mean and median revenue per user for each plan, include user-months without calls, apply the operator's rounding rules if confirmed, use Welch's t-test, and quantify the effect sizes behind the significant differences.

## Repository Structure

```text
telecom-plan-revenue-analysis/
├── README.md
├── notebook.ipynb
└── datasets/
    ├── megaline_calls.csv
    ├── megaline_internet.csv
    ├── megaline_messages.csv
    ├── megaline_plans.csv
    └── megaline_users.csv
```

## How to Run

1. Create and activate a Python 3 environment (the notebook was last run with Python 3.13):

   ```bash
   python -m venv .venv
   # Windows
   .venv\Scripts\activate
   # macOS / Linux
   source .venv/bin/activate
   ```

2. Install the dependencies:

   ```bash
   pip install pandas numpy scipy matplotlib seaborn jupyter
   ```

3. Start Jupyter **from the project root**, so that the relative paths `datasets/<file>.csv` resolve correctly, and open `notebook.ipynb`:

   ```bash
   jupyter notebook notebook.ipynb
   ```

Printed messages produced by the code (for example, the hypothesis-test decisions) are in Portuguese, as in the original run.

## Author

**Luiz Otávio Carneiro Filho**
Geospatial Engineer | GIS & Data Science
[LinkedIn](https://www.linkedin.com/in/luizot/)
