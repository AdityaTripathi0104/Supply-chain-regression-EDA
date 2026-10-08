# LogiPredict: Supply Chain Diagnostic & Sales Prediction Engine

## Project Overview
**LogiPredict** is an automated Exploratory Data Analysis (EDA) pipeline and predictive modeling engine designed to analyze Fast-Moving Consumer Goods (FMCG) supply chain data. Built on a dataset of 1.1 million rows spanning three years, this project was initially conceived to identify historical freight bottlenecks and predict transit delays. 

However, through rigorous statistical testing and correlation analysis, the project successfully identified critical artificial anomalies within the dataset itself, ultimately pivoting to a predictive regression model for unit sales based on marketing drivers.

---

## Phase 1: Data Ingestion and Cleaning
**The "What":** Loading, profiling, and sanitizing a massive enterprise-scale dataset (1.1 million rows, 33 columns).
**The "Why":** Real-world data is rarely clean. "Hidden nulls" (strings that look like valid data but represent missing values) can severely skew statistical models and lead to false conclusions. 
**The "How":** 
*   Implemented a "Cardinality Shield" using dictionary comprehensions to isolate categorical columns with fewer than 50 unique values.
*   Scanned for and neutralized hidden nulls by converting edge-case strings (e.g., `' '`, `'-'`, `'Unknown'`, `'N/A'`, `'null'`) into mathematically recognizable `np.nan` values.
*   **Result:** Verified the dataset possessed unusually high data integrity, establishing a safe foundation for analysis.

---

## Phase 2: Univariate Analysis
**The "What":** Analyzing the individual distributions of core business metrics (`gross_sales`, `units_sold`, `lead_time_days`).
**The "Why":** To understand the baseline behavior of the business before looking for complex relationships.
**The "How":** Visualized the data using Seaborn histograms and Pandas statistical summaries (`.describe()`).
*   **Sales Insight:** Both `gross_sales` and `units_sold` are heavily right-skewed. The vast majority of transactions are small (0 to 500 units), with a few massive outliers representing bulk B2B purchases.
*   **Supply Chain Insight:** `lead_time_days` exhibited a near-normal distribution centered around 6-8 days. However, inventory levels (`stock_on_hand`) pooled around exact, round numbers. This indicated that the company uses strict, fixed reorder quantities (e.g., standard pallets of 50 or 100) rather than organic, unit-by-unit restocking.

---

## Phase 3: Bivariate Analysis & The "Stockout" Anomaly
**The "What":** Cross-referencing operational metrics against business failures (Stockouts) to find root causes.
**The "Why":** The logical business assumption is that slow supplier lead times or massive promotional sales cause shelves to go empty. We needed to prove this mathematically.
**The "How":** Executed multiple GroupBy aggregations and generated a comprehensive Correlation Matrix Heatmap.
*   **The Investigation:** We tested `lead_time_days`, `units_sold`, `promo_flag`, `stock_on_hand`, and `channel` against the `stock_out_flag`. 
*   **The Revelation:** Across every single metric, the stockout rate remained a perfectly flat ~3%. Lead times of 2 days vs. 15 days had the exact same stockout rate. 
*   **The Conclusion:** We mathematically proved that stockouts in this dataset are entirely random. Because this is a Kaggle dataset, we identified a synthetic data artifact: the creator likely wrote a script to randomly assign a "1" to 3% of the rows to simulate stockouts, rather than providing real causal logic.

---

## Phase 4: Predictive Modeling (Sales Forecasting)
**The "What":** Building a Machine Learning pipeline to forecast `units_sold`.
**The "Why":** Since the supply chain delay data proved to be randomized, the engine pivoted to predicting inventory outflow (sales) based on controllable marketing levers. 
**The "How":** 
*   Isolated `promo_flag` and `discount_pct` as the input features (X) and `units_sold` as the target variable (y).
*   Performed an 80/20 Train/Test split to ensure the model was evaluated on unseen data.
*   Trained both a **Linear Regression** model and a **Ridge Regression** (L2 Regularization) model to predict sales volume.

### Model Evaluation & Performance
*   **Mean Absolute Error (MAE): 33.36 units.** When using this model to predict tomorrow's sales, the prediction is typically off by about 33 units.
*   **R-Squared ($R^2$): 0.0896.**

---

## Limitations & Key Takeaways

1. **The Synthetic Data Limitation:** The most valuable outcome of this diagnostic engine was proving that the target variable for supply chain delays (`stock_out_flag`) was artificially generated noise. A lesser analysis would have blindly trained a classification model on this data, resulting in a deployed model that guesses randomly.
2. **The Feature Deficit (Understanding the 9%):** The predictive regression model yielded an R-Squared of ~0.09. This means that marketing levers (promotions and discounts) only account for roughly 9% of the reason sales fluctuate. 
3. **Strategic Next Steps:** Because 91% of sales variance is caused by factors *outside* of promotions and discounts, future iterations of this model must incorporate exogenous variables (e.g., seasonality, day-of-week trends, localized events, and macroeconomic indicators) to achieve production-grade accuracy.
