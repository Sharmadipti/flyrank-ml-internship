Content Opportunity Scoring Using Search Performance Signals
Author: Dipti Sharma
Lane: Refresh / Content Opportunity Scoring
Repo: https://github.com/Sharmadipti/flyrank-ml-internship
Date: November 2026

# 0. Abstract

This project investigates whether a machine learning model can help prioritize which content pages should be reviewed first compared with a simple rule-based approach. The analysis uses the FlyRank ML Internship warehouse dataset, combining February 2026 search-performance features with a March 2026 outcome label indicating whether a page went dark. A Logistic Regression model and a rule-based baseline were evaluated using search-performance signals including impressions, clicks, CTR, average position, and content age. The balanced Logistic Regression model improved detection of pages that went dark, while the baseline provided transparent prioritization through action scores and reason codes. The final output is a ranked decision-support workflow that helps content teams prioritize pages for review, monitoring, protection, or improvement.

---

# 1. Problem framing

Large websites may contain thousands of content pages, making manual review difficult and expensive. This project supports the decision of which pages should be reviewed first by content and SEO teams.

The unit of analysis is one content page for one client during a monthly observation period. The output is a ranked prioritization workflow consisting of risk signals, action labels, and review recommendations.

The intended human action is to review, improve, monitor, or protect content based on the observed signals and model output.

The cost of a wrong decision is either reviewing pages that did not require attention (false positives) or missing pages that may lose visibility and deserved earlier review (false negatives).

Machine learning and data analysis help by identifying patterns across large content portfolios that are difficult to assess manually at scale.

---

# 2. Data safety

The project uses data from the FlyRank ML Internship warehouse dataset accessed through DuckDB and Hugging Face.

## Tables Used

- fact_content_daily_performance
- dim_content

## Feature Set

The final feature set included:

- impressions_feb
- clicks_feb
- ctr_feb
- avg_position_feb
- content_age_days

## Target Variable

The target variable was:

- went_dark

where:

- 1 = page received zero clicks in March 2026
- 0 = page continued receiving clicks in March 2026

## Excluded Fields

The following fields were deliberately excluded:

- Client names
- URLs
- Private search queries
- Future outcome data
- Label-derived variables

Leakage risks were reviewed carefully. Variables such as `trend_direction` and `trend_pct` were not included in the final model because they could be derived from outcome information. Client identifiers were used only for grouped validation and were never used as model features.

No client-identifying information appears in the project outputs, notebooks, recommendations, or report.

The final dataset was filtered to:

- Published content only
- February impressions ≥ 100
- February clicks ≥ 3

These filters help ensure sufficient search-performance history while maintaining a public-safe analysis.


# 3. Baseline

A transparent rule-based baseline scoring system was developed before building a machine learning model.

The baseline was designed to prioritize content pages for review using observable search-performance signals. The scoring framework incorporated factors such as:

- Search impressions
- Search clicks
- Content age
- Freshness indicators
- Content review signals

Pages were assigned:

- A baseline score
- A reason code
- An action label

The resulting queue was exported as:

work/outputs/baseline_action_score.csv

This baseline provides a fair comparison because it uses the same content-review objective as the machine learning approach while remaining highly interpretable.

The baseline supports human decision-making through transparent rules that can be reviewed and adjusted by content teams.

---

# 4. Model / Analysis

## Feature Definition

The feature frame was constructed using February 2026 search-performance data.

The final feature set included:

- impressions_feb
- clicks_feb
- ctr_feb
- avg_position_feb
- content_age_days

These features were selected because they were available before the prediction window and could therefore be used in a realistic decision-support workflow.

## Target Definition

The target variable, `went_dark`, was defined using March 2026 clicks:

- 1 = page received zero clicks during March
- 0 = page continued receiving clicks during March

This definition provides a simple way to identify pages that may have lost visibility after the February observation period.

## Machine Learning Model

A Logistic Regression model was trained using the five-feature dataset.

Because the dataset was highly imbalanced, a balanced Logistic Regression model using:

```python
class_weight="balanced"
```

was evaluated.

The objective was not to automate content decisions but to estimate which pages may deserve earlier human review.

---

# 5. Evaluation

## Validation Design

A stratified train/test split was used during model development.

A grouped split based on client_hash_id was also used during validation to reduce the risk of client-specific patterns appearing in both training and testing sets.

Because only a single February-to-March snapshot was available, a true time-based validation was not possible.

## Results

The initial Logistic Regression model achieved approximately 94.9% accuracy.

However, further analysis showed that the model failed to identify pages in the minority `went_dark` class.

A balanced Logistic Regression model substantially improved recall for the minority class.

Key observations:

| Metric | Result |
|----------|----------|
| Initial Accuracy | 0.949 |
| Balanced Recall (went_dark=1) | 0.84 |
| Dataset Rows | 29,700 |

## Error Analysis

The dataset was highly imbalanced, with approximately 95% of pages remaining active and only 5% belonging to the `went_dark` class.

The standard Logistic Regression model largely learned to predict the majority class.

Using class balancing improved detection of pages that went dark but increased the number of false positives.

This tradeoff is acceptable in a review-prioritization workflow because missing an at-risk page may be more costly than reviewing additional pages.

---

# 6. Interpretation

The analysis suggests that historical search-performance signals contain useful information for identifying pages that may lose visibility.

The machine learning model found patterns across impressions, clicks, CTR, average position, and content age that were associated with future outcomes.

A key finding was that class imbalance had a larger impact on performance than model complexity.

The project also demonstrated the importance of leakage checks. A deliberate leakage experiment achieved near-perfect performance when outcome information was included, highlighting the need for strict feature controls.

Negative results were also informative. High overall accuracy did not necessarily indicate a useful model because minority-class detection remained poor until class balancing was introduced.

---

# 7. Recommendation

The resulting workflow is intended for SEO analysts, content strategists, and content review teams.

Recommended actions:

## Review

Pages showing elevated risk of going dark or declining performance signals should be reviewed first.

## Improve

Pages with search demand but weak CTR or other performance signals should be considered for optimization.

## Monitor

Pages with stable performance should continue to be tracked.

## Protect

High-performing pages should avoid unnecessary edits or refreshes.

Confidence in these recommendations is moderate. The workflow provides decision-support signals based on observed search-performance data but should not be interpreted as automated decision making.

---

# 8. Reproducibility

Environment:

- Python
- Pandas
- NumPy
- Scikit-learn
- DuckDB
- Hugging Face datasets

Random Seed:

- train_test_split random_state = 42

Core workflow:

1. Connect to the FlyRank warehouse using DuckDB.
2. Build the February 2026 feature frame.
3. Create the March 2026 `went_dark` outcome label.
4. Train a Logistic Regression model.
5. Evaluate using stratified and grouped validation.
6. Generate action scores and recommendations.

Primary notebooks:

- w03_data_contract.ipynb
- w04_baseline_score.ipynb
- w05_model.ipynb
- w06_validation_audit.ipynb
- w07_action_playbook.ipynb
- capstone.ipynb

---

# 9. Acknowledgments & Data Credit

Built on the FlyRank ML Internship dataset.

Data source:

https://flyrank.ai

The project was completed as part of the FlyRank Machine Learning Internship and uses publicly shareable outputs generated from the internship warehouse environment.

All findings should be interpreted as observational, directional, and decision-support oriented rather than causal claims.
