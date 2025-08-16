<div align="center">
  <h1>Onboarding → Retention → Revenue — SQL Case Study</h1>
  <p><i>Turning user behavior into business decisions</i></p>

  <a href="https://www.linkedin.com/in/muhammad-ashraful/">
    <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn Badge"/>
  </a>

  <img src="https://img.shields.io/badge/Tool-SQL%20%7C%20SQLite-217346?style=for-the-badge" alt="SQL Badge"/>
  <img src="https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge" alt="Status Badge"/>
</div>

---

<table align="center">
  <tr>
    <td width="33%" valign="top">
      <h3>🎯 Key Business Question</h3>
      <p>Can we increase short-term retention and revenue by improving the onboarding experience (specifically, driving completion of onboarding step 2)?</p>
    </td>
    <td width="33%" valign="top">
      <h3>💡 Analytical Approach</h3>
      <p>Cohort-based retention, LTV vs CAC by acquisition channel, behavioral drivers of churn (feature use, NPS, tickets), and a counterfactual uplift simulation to estimate revenue impact.</p>
    </td>
    <td width="33%" valign="top">
      <h3>📊 Expected Impact</h3>
      <p>Targeted nudges for high-potential non-onboarded users could produce measurable revenue uplift and improve LTV:CAC across acquisition channels. Example estimate is included below.</p>
    </td>
  </tr>
</table>

---

## SQL-driven Analysis (Queries & Sample Results)

Below are **12** high-impact SQL queries designed to demonstrate analytical thinking and business impact. Each query is followed by a short interpretation and a sample result using the provided dataset.

### 1) Onboarding: Retention & Revenue by Onboarding Completion

```sql
-- Q1: Retention & revenue by onboarding_step2_completed
SELECT
  onboarding_step2_completed,
  COUNT(*) AS users,
  ROUND(100.0 * SUM(CASE WHEN churned_90d=0 THEN 1 ELSE 0 END) / COUNT(*), 2) AS retention90_pct,
  ROUND(AVG(revenue_30d),2) AS avg_revenue_30d,
  ROUND(AVG(ltv_90d),2) AS avg_ltv_90d
FROM users
GROUP BY onboarding_step2_completed
ORDER BY onboarding_step2_completed DESC;
```

**Sample result (from `portfolio_dataset_rich.csv`)**

| onboarding\_step2\_completed | users | retention90\_pct | avg\_revenue\_30d | avg\_ltv\_90d |
| ---------------------------: | ----: | ---------------: | ----------------: | ------------: |
|                            1 |    93 |            86.02 |             26.22 |         30.57 |
|                            0 |   107 |            55.14 |              7.13 |          5.58 |

**Interpretation / recommendation:** Users who complete onboarding step 2 show materially higher retention and LTV — suggest prioritizing nudges for non-completers.

---

### 2) Channel economics: CAC vs LTV (which channels to scale?)

```sql
-- Q2: CAC vs LTV by channel
SELECT channel,
       COUNT(*) AS users,
       ROUND(AVG(acquisition_cost),2) AS avg_cac,
       ROUND(AVG(ltv_90d),2) AS avg_ltv_90d,
       ROUND(AVG(ltv_90d)/NULLIF(AVG(acquisition_cost),0),2) AS ltv_to_cac
FROM users
GROUP BY channel
ORDER BY ltv_to_cac DESC;
```

**Sample result (from `portfolio_dataset_rich.csv`)**

| channel         | users | avg\_cac | avg\_ltv90 | avg\_revenue30 | ltv\_to\_cac |
| :-------------- | ----: | -------: | ---------: | -------------: | -----------: |
| D\_partnership  |    19 |     5.59 |      25.66 |          14.85 |         4.59 |
| B\_email        |    50 |     2.84 |       8.89 |           4.66 |         3.13 |
| A\_organic      |    90 |     1.56 |       9.36 |           6.53 |         6.00 |
| C\_paid\_social |    41 |     8.46 |       5.35 |           3.22 |         0.63 |

**Interpretation / recommendation:** Compare LTV\:CAC to decide which channels to invest in. Channels with LTV\:CAC < 1 need optimization or pause.

---

### 3) Cohort retention: signup month × channel

```sql
-- Q3: Cohort retention by signup_month and channel
SELECT date_trunc('month', signup_date) AS signup_month,
       channel,
       COUNT(*) AS users,
       ROUND(100.0 * SUM(CASE WHEN churned_90d=0 THEN 1 ELSE 0 END) / COUNT(*), 2) AS retention90_pct
FROM users
GROUP BY signup_month, channel
ORDER BY signup_month, channel;
```

**Sample result (from `portfolio_dataset_rich.csv`)**

| signup\_month | channel         | users | retention90\_pct |
| :------------ | :-------------- | ----: | ---------------: |
| 2025-04       | A\_organic      |     4 |               25 |
| 2025-04       | B\_email        |     5 |               80 |
| 2025-04       | C\_paid\_social |     1 |              100 |
| 2025-04       | D\_partnership  |     1 |              100 |
| 2025-05       | A\_organic      |    22 |            81.82 |
| 2025-05       | B\_email        |     8 |               75 |
| 2025-05       | C\_paid\_social |     7 |            57.14 |
| 2025-05       | D\_partnership  |     1 |                0 |
| 2025-06       | A\_organic      |    30 |            63.33 |
| 2025-06       | B\_email        |    16 |            68.75 |
| 2025-06       | C\_paid\_social |     9 |            77.78 |
| 2025-06       | D\_partnership  |     4 |               75 |

**Interpretation / recommendation:** Cohort trends reveal whether recent product changes (or campaigns) improved retention; look for monotonic improvements across months.

---

### 4) Onboarding funnel: who completes step2 (by channel & plan)

```sql
-- Q4: Onboarding completion rate by channel & plan
SELECT channel, plan,
       COUNT(*) AS users,
       ROUND(100.0 * SUM(onboarding_step2_completed)/COUNT(*),2) AS onboard_rate_pct
FROM users
GROUP BY channel, plan
ORDER BY channel, plan;
```

**Sample result (from `portfolio_dataset_rich.csv`)**

| channel         | plan    | users | onboard\_rate\_pct |
| :-------------- | :------ | ----: | -----------------: |
| A\_organic      | annual  |     9 |              77.78 |
| A\_organic      | free    |    47 |              29.79 |
| A\_organic      | monthly |    26 |              53.85 |
| A\_organic      | trial   |     8 |                100 |
| B\_email        | annual  |     6 |                100 |
| B\_email        | free    |    18 |              55.56 |
| B\_email        | monthly |    19 |              68.42 |
| B\_email        | trial   |     7 |              85.71 |
| C\_paid\_social | annual  |     4 |                 75 |
| C\_paid\_social | free    |    17 |              17.65 |
| C\_paid\_social | monthly |     9 |              77.78 |
| C\_paid\_social | trial   |     3 |                100 |
| D\_partnership  | annual  |     2 |                100 |
| D\_partnership  | free    |     6 |                100 |
| D\_partnership  | monthly |     6 |              66.67 |
| D\_partnership  | trial   |     5 |                100 |

**Interpretation / recommendation:** Channels/plans with low onboard\_rate are prime candidates for tailored onboarding experiences or incentives.

---

### 5) Feature usage: do power users retain more?

```sql
-- Q5: Feature usage vs retention (aggregated)
SELECT churned_90d,
       COUNT(*) AS users,
       ROUND(AVG(feature_usage_score),2) AS avg_feature_usage,
       ROUND(AVG(events_30d),2) AS avg_events_30d
FROM users
GROUP BY churned_90d
ORDER BY churned_90d;
```

**Sample result (from `portfolio_dataset_rich.csv`)**

| churned\_90d  | users | avg\_feature\_usage | avg\_events30 |
| :------------ | ----: | ------------------: | ------------: |
| retained\_90d |   143 |               18.23 |         19.03 |
| churned\_90d  |    57 |                4.77 |          4.28 |

**Interpretation / recommendation:** Higher feature usage correlates with retention — invest in driving early feature activation.

---

### 6) Actionable list: High potential users who haven't completed onboarding

```sql
-- Q6: Export top non-onboarded users by potential (feature_usage_score, events)
SELECT user_id, signup_date, channel, feature_usage_score, events_30d, ltv_90d, last_active_date
FROM users
WHERE onboarding_step2_completed = 0
ORDER BY feature_usage_score DESC, events_30d DESC
LIMIT 50;
```

**Sample result (from `portfolio_dataset_rich.csv`)**

| user\_id | signup\_date | channel         | feature\_usage\_score | events\_30d | ltv\_90d | last\_active\_date |
| :------- | :----------- | :-------------- | --------------------: | ----------: | -------: | :----------------- |
| U046     | 2025-06-17   | A\_organic      |                    47 |          43 |    74.76 | 2025-07-23         |
| U067     | 2025-06-29   | C\_paid\_social |                    44 |          31 |    38.45 | 2025-08-05         |
| U075     | 2025-06-24   | A\_organic      |                    40 |          26 |     34.6 | 2025-07-30         |
| U036     | 2025-05-27   | A\_organic      |                    39 |          24 |    28.63 | 2025-07-10         |
| U003     | 2025-04-14   | A\_organic      |                    37 |          23 |    40.42 | 2025-07-12         |
| U181     | 2025-07-10   | A\_organic      |                    34 |          21 |    46.12 | 2025-08-02         |
| U118     | 2025-06-23   | B\_email        |                    33 |          20 |    27.28 | 2025-08-05         |
| U139     | 2025-06-03   | A\_organic      |                    33 |          18 |    20.05 | 2025-07-25         |
| U053     | 2025-05-19   | A\_organic      |                    32 |          18 |    21.58 | 2025-07-18         |
| U009     | 2025-04-27   | B\_email        |                    31 |          16 |    11.21 | 2025-06-30         |

**Interpretation / recommendation:** Provide this export to product/marketing for targeted nudges (in-app message, email + promo).

---

### 7) NPS & monetization: does satisfaction map to revenue?

```sql
-- Q7: NPS score vs avg revenue & LTV
SELECT nps_score, COUNT(*) AS users, ROUND(AVG(revenue_30d),2) AS avg_revenue_30d, ROUND(AVG(ltv_90d),2) AS avg_ltv_90d
FROM users
GROUP BY nps_score
ORDER BY nps_score;
```

**Sample result (from `portfolio_dataset_rich.csv`)**

| nps\_score | users | avg\_revenue\_30d | avg\_ltv\_90d |
| ---------: | ----: | ----------------: | ------------: |
|          2 |     5 |                 0 |          0.89 |
|          3 |     6 |                 0 |          0.56 |
|          4 |     7 |                 0 |          0.63 |
|          5 |    10 |               2.1 |           2.1 |
|          6 |    18 |              3.89 |           4.6 |

**Interpretation / recommendation:** If low NPS segments show both low revenue and high churn, prioritize customer success interventions.

---

### 8) Support load: tickets → churn

```sql
-- Q8: Churn rate by support tickets (bucketed)
SELECT CASE WHEN support_tickets=0 THEN '0' WHEN support_tickets=1 THEN '1' ELSE '2+' END AS tickets_bucket,
       COUNT(*) AS users,
       ROUND(100.0 * SUM(churned_90d)/COUNT(*),2) AS churn_rate_pct
FROM users
GROUP BY tickets_bucket
ORDER BY tickets_bucket;
```

**Sample result (from `portfolio_dataset_rich.csv`)**

| tickets\_bucket | users | churn\_rate\_pct |
| :-------------- | ----: | ---------------: |
| 0               |   167 |            24.55 |
| 1               |    28 |               50 |
| 2+              |     5 |               60 |

**Interpretation / recommendation:** More tickets correlate with higher churn — early proactive outreach can reduce churn and improve NPS.

---

### 9) Churn timing: days to churn distribution

```sql
-- Q9: Days to churn distribution (for churned users)
SELECT
  COUNT(*) AS churned_users,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY days_to_churn) AS median_days_to_churn,
  PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY days_to_churn) AS p25,
  PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY days_to_churn) AS p75
FROM users
WHERE churned_90d = 1;
```

**Sample result (from `portfolio_dataset_rich.csv`)**

Churned users: 57
Median days to churn: 24
25th pct: 12
75th pct: 47

**Interpretation / recommendation:** If median days to churn is early (e.g., <30), prioritize immediate onboarding touches within first 7–14 days.

---

### 10) Conversion economics: CAC payback

```sql
-- Q10: Avg CAC and LTV for converted vs non-converted users (converted defined as revenue_30d>0)
SELECT (revenue_30d>0) AS converted_30d,
       COUNT(*) AS users,
       ROUND(AVG(acquisition_cost),2) AS avg_cac,
       ROUND(AVG(revenue_30d),2) AS avg_revenue_30d,
       ROUND(AVG(ltv_90d),2) AS avg_ltv_90d
FROM users
GROUP BY converted_30d;
```

**Sample result (from `portfolio_dataset_rich.csv`)**

| converted\_30d      | users | avg\_cac | avg\_revenue30 | avg\_ltv90 |
| :------------------ | ----: | -------: | -------------: | ---------: |
| not\_converted\_30d |   156 |     2.77 |              0 |       2.09 |
| converted\_30d      |    44 |     4.86 |           29.3 |      38.59 |

**Interpretation / recommendation:** Compare avg LTV to CAC to evaluate payback and decide channel allocation.

---

### 11) Channel LTV trend (sample months)

```sql
-- Q11: Channel LTV trend by signup_month (sample)
SELECT signup_month, channel, COUNT(*) AS users, ROUND(AVG(ltv_90d),2) AS avg_ltv_90d
FROM users
GROUP BY signup_month, channel
ORDER BY signup_month, avg_ltv_90d DESC;
```

**Sample result (from `portfolio_dataset_rich.csv`)**

| signup\_month | channel         | users | avg\_ltv\_90d |
| :------------ | :-------------- | ----: | ------------: |
| 2025-04       | B\_email        |     5 |         22.84 |
| 2025-04       | A\_organic      |     4 |          12.1 |
| 2025-04       | C\_paid\_social |     1 |            10 |
| 2025-04       | D\_partnership  |     1 |           8.1 |
| 2025-05       | A\_organic      |    22 |         14.55 |
| 2025-05       | B\_email        |     8 |          13.2 |
| 2025-05       | C\_paid\_social |     7 |         11.15 |
| 2025-05       | D\_partnership  |     1 |             0 |
| 2025-06       | A\_organic      |    30 |         12.01 |
| 2025-06       | B\_email        |    16 |         11.65 |
| 2025-06       | C\_paid\_social |     9 |         10.45 |
| 2025-06       | D\_partnership  |     4 |          9.03 |

**Interpretation / recommendation:** A rising channel LTV trend indicates improved targeting or product-market fit for that channel.

---

### 12) Counterfactual uplift: estimate revenue if we nudge 20 users

```sql
-- Q12: Counterfactual: estimate revenue uplift by moving 20 high-potential non-onboarded users to onboarded
-- (This is a simple lift estimate using observed avg LTV for onboarded vs non-onboarded)
```

**Sample result (from `portfolio_dataset_rich.csv`)**

avg\_ltv\_if\_completed: 30.57
avg\_ltv\_if\_not: 5.58
est\_additional\_revenue\_move\_20: 499.8

**Interpretation / recommendation:** This quick estimate helps prioritize investments in onboarding nudges by giving a dollar-value estimate for expected incremental revenue.

---

## Methodology & Analyst Notes

* I prioritize **business questions** and map them to KPIs (e.g., retention\_90d, revenue\_30d, ltv\_90d). Each query is designed to answer a specific stakeholder question (Product, Marketing, CS, Finance).
* I use cohort analysis, segmentation, counterfactuals, and operational exports to move from insight → action.



<div align="center">
  <h3>Let's Connect</h3>
  <a href="https://www.linkedin.com/in/muhammad-ashraful/">
    <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn Badge"/>
  </a>
</div>
