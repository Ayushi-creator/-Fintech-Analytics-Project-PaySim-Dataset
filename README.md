📊 Fintech Analytics Project – PaySim Dataset

This project demonstrates end-to-end analytics on mobile money transactions using SQL, SQLite, and Python. The goal is to derive actionable business insights such as user activity, churn, revenue growth, and behavioral patterns from transactional data.


🔍 Objective

To analyze non-fraudulent mobile money transactions and build core KPIs like Monthly Transacting Users (MTU), Churn, Revenue Trends, and Power Users. Advanced SQL features such as **CTEs**, **JOINs**, and **Window Functions** are used to simulate real-world fintech analytics.


📦 Dataset

- Name: PaySim – A Financial Transaction Simulation Dataset
- Source: [Kaggle – PaySim Dataset](https://www.kaggle.com/datasets/ntnu-testimon/paysim1)
- Format: CSV
- Size: ~6.3 million rows
- Description: A synthetic dataset simulating real-world mobile money transactions, including fraud and non-fraud cases.

🛠️ Tools & Technologies

- Python (Jupyter Notebook)
- SQLite (via Python’s `sqlite3` module)
- SQL (CTEs, Window Functions, JOINs)
- Pandas (for data manipulation)
- Matplotlib (for visualizations)
- GitHub (for version control & project showcasing)


📊 Key Performance Indicators (KPIs) – Implemented in SQL
---
1. 📆 Monthly Transacting Users (MTU)
Definition: Unique users who made at least one transaction in each month.
SQL Logic:
SELECT (step / 720) + 1 AS month,
       COUNT(DISTINCT nameOrig) AS mtu
FROM transactions
WHERE isFraud = 0
GROUP BY month;
---
2. 📉 Churned Users
Definition: Users who were active in month M but didn’t return in month M+1.
SQL Logic:
WITH user_months AS (
    SELECT DISTINCT nameOrig AS user_id,
                    (step / 720) + 1 AS month
    FROM transactions
    WHERE isFraud = 0
),
churned_users AS (
    SELECT this_month.user_id,
           this_month.month AS churn_month
    FROM user_months this_month
    LEFT JOIN user_months next_month
      ON this_month.user_id = next_month.user_id
     AND this_month.month + 1 = next_month.month
    WHERE next_month.user_id IS NULL
)
SELECT churn_month, COUNT(DISTINCT user_id) AS churned_users
FROM churned_users
GROUP BY churn_month;
---
3. 🔥 Power Users
Definition: Top 10 users by number of transactions.
SQL Logic:
SELECT nameOrig AS user_id,
       COUNT(*) AS txn_count
FROM transactions
WHERE isFraud = 0
GROUP BY user_id
ORDER BY txn_count DESC
LIMIT 10;
---
4. 💰 Revenue Growth (Monthly)
Definition: Total transaction amount per month (non-fraud).
SQL Logic:
SELECT (step / 720) + 1 AS month,
       SUM(amount) AS total_revenue
FROM transactions
WHERE isFraud = 0
GROUP BY month
ORDER BY month;
---
5. 🔁 Funnel Analysis (Optional)
Definition: Users who performed CASH_IN followed by TRANSFER.
SQL Logic:
WITH cash_in_users AS (
    SELECT DISTINCT nameOrig
    FROM transactions
    WHERE type = 'CASH_IN' AND isFraud = 0
),
transfer_users AS (
    SELECT DISTINCT nameOrig
    FROM transactions
    WHERE type = 'TRANSFER' AND isFraud = 0
)
SELECT
    (SELECT COUNT(*) FROM cash_in_users) AS total_cash_in_users,
    (SELECT COUNT(*) FROM transfer_users) AS total_transfer_users,
    (SELECT COUNT(*)
     FROM cash_in_users
     WHERE nameOrig IN (SELECT nameOrig FROM transfer_users)) AS converted_users;
---
6. 🪜 KPI – Window Function (Top Transaction Per User)
Definition: Find the highest-value transaction per user.
SQL Logic:
WITH ranked_txns AS (
    SELECT
        nameOrig AS user_id,
        amount,
        type,
        step,
        ROW_NUMBER() OVER (PARTITION BY nameOrig ORDER BY amount DESC) AS rn
    FROM transactions
    WHERE isFraud = 0
)
SELECT *
FROM ranked_txns
WHERE rn = 1
LIMIT 10;

📈 Visualizations (Python)
Built using matplotlib for presenting insights in a visual form:

📈 MTU Over Time (Line chart)

📊 Monthly Revenue (Bar chart)
