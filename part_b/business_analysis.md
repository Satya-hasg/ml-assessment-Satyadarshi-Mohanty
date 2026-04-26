# Part B: Business Case Analysis

## Scenario: Promotion Effectiveness at a Fashion Retail Chain

---

## B1. Problem Formulation

### (a) ML Problem Definition

From a business standpoint, the objective here is to understand how different promotions impact sales performance at a store level and to recommend the most effective promotion strategy.

This can be framed as a *supervised regression problem*, where the goal is to predict:

- *Target Variable:*  
  items_sold (sales volume)

- *Input Features:*  
  - Promotion type  
  - Store characteristics (store size, location type)  
  - Competition density  
  - Temporal signals (month, weekend, festival periods, month-end effects)  
  - Historical demand patterns  

- *Problem Type:*  
  Regression

*Rationale:*  
We are predicting a continuous outcome (sales volume), and more importantly, this formulation allows the business to simulate “what-if” scenarios — i.e., how sales would change under different promotion strategies.

---

### (b) Why items_sold is a better target than revenue

In real-world retail analytics, revenue often introduces noise due to multiple external factors such as pricing strategies, discount depth, and product mix variations.

Using *items_sold* is a more reliable proxy for customer demand because:

- It directly reflects customer response to promotions  
- It is not distorted by pricing or discounting mechanics  
- It enables cleaner comparison across different promotion types  

This reflects a broader principle in ML problem design:

> The target variable should closely align with the core business objective while minimizing external distortions.

---

### (c) Alternative to a Single Global Model

In my experience, using a single global model across all stores tends to oversimplify the problem, especially in retail where store behavior is highly context-dependent.

A more practical approach would be:

- *Segmented Modeling Strategy:*
  - Build models by store clusters (urban, semi-urban, rural), OR  
  - Introduce interaction effects between promotion type and store attributes  

*Why this works better:*
- Customer behavior differs significantly across geographies  
- Promotion effectiveness is not uniform  
- It improves both model accuracy and interpretability  

This approach also aligns better with how business teams actually make decisions — at a regional or store segment level rather than globally.

---

## B2. Data and EDA Strategy

### (a) Data Integration and Dataset Design

The data is distributed across multiple tables — transactions, store attributes, promotions, and calendar.

From a data modeling perspective:

- Transactions will act as the *base fact table*
- Store attributes can be joined using store_id
- Promotion details via promotion identifiers
- Calendar data via transaction_date

*Final Dataset Grain:*
- One row per *store per day* (aggregated level)

*Key Aggregations:*
- Total items_sold per store per day  
- Promotion applied on that day  
- Derived temporal indicators (weekend, festival, month-end)

This level of granularity strikes a good balance between capturing patterns and keeping the dataset manageable for modeling.

---

### (b) EDA Approach

Before jumping into modeling, I would focus on understanding the data through a few targeted analyses:

1. *Promotion Effectiveness Analysis*
   - Compare average sales across different promotion types  
   - Identify consistently high-performing promotions  

2. *Temporal Trend Analysis*
   - Evaluate seasonality (monthly trends, festivals, weekends)  
   - Identify demand spikes and cyclic behavior  

3. *Store-Level Performance Analysis*
   - Analyze differences across store sizes and locations  
   - Check for regional demand patterns  

4. *Feature Relationships*
   - Correlation analysis to understand key drivers  
   - Detect redundant or highly correlated features  

5. *Outlier Analysis*
   - Identify abnormal sales spikes (e.g., festival peaks)  
   - Decide whether to retain or cap extreme values  

*Why this matters:*  
Strong EDA ensures that feature engineering decisions are grounded in actual business patterns rather than assumptions.

---

### (c) Handling Promotion Imbalance

The observation that ~80% of transactions occur without promotions is important.

*Potential Issue:*
- The model may become biased toward predicting non-promotion scenarios  
- It may under-represent the actual impact of promotions  

*How I would address this:*
- Monitor performance separately for promotion vs non-promotion cases  
- Use sample weighting if needed  
- Ensure evaluation metrics capture performance across both segments  

Rather than blindly balancing the dataset, I would focus on ensuring the model learns meaningful promotion effects.

---

## B3. Model Evaluation and Deployment

### (a) Train-Test Strategy and Metrics

Given the time-dependent nature of retail data, I would strictly use a *time-based split*:

- Train on historical data  
- Test on the most recent period  

*Why not random split:*
- It introduces data leakage  
- Breaks temporal dependencies  
- Leads to overly optimistic results  

*Evaluation Metrics:*
- RMSE → captures large prediction errors  
- MAE → provides interpretable average error  
- R² → explains overall model fit  

From a business perspective, MAE is particularly useful as it directly tells stakeholders the average deviation in predicted sales.

---

### (b) Explaining Model Recommendations

If the model recommends different promotions for the same store across different months, this is expected behavior.

To explain this to stakeholders, I would:

- Use *feature importance or SHAP values*  
- Compare the drivers influencing predictions in each scenario  

Typical influencing factors could include:
- Seasonal demand (e.g., December vs March)  
- Festival periods  
- Month-end buying behavior  
- Competitive environment  

*Business Communication:*
I would position this as the model being *context-aware*, rather than inconsistent, and highlight the key drivers behind each recommendation.

---

### (c) Deployment and Monitoring Strategy

From an implementation standpoint, I would design a simple but robust pipeline:

*Deployment Steps:*
1. Save the trained model (joblib/pickle)  
2. Automate monthly data ingestion and preprocessing  
3. Generate promotion recommendations at the start of each month  
4. Schedule batch jobs for execution  

---

*Monitoring Strategy:*

To ensure long-term reliability:

- Track prediction vs actual performance  
- Monitor key metrics (RMSE, MAE trends)  
- Detect:
  - Data drift (changes in input features)  
  - Concept drift (changes in customer behavior)  

---

*Retraining Approach:*

- Retrain periodically (monthly/quarterly)  
- Incorporate latest data  
- Validate before deploying updated models  

---

## Conclusion

A well-structured ML approach, combined with strong data understanding and continuous monitoring, can significantly improve promotion planning. 

More importantly, the value here is not just in prediction accuracy, but in enabling the business to make *data-driven, context-aware decisions at scale* across stores.