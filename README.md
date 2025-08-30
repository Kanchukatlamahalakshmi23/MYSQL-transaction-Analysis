# MYSQL-transaction-Analysis
"Built a complete SQL project to process over 280,000 transactions and create a ranked 'fraud score' for high-risk activity."
# Fraud Detection in SQL

## Summary
This project uses SQL to analyze a large transaction dataset to identify and score potentially fraudulent activities. I built a rule-based system from scratch to flag suspicious patterns like high-frequency spending and unusually large purchases.

## Problem Statement
Credit card fraud is a major issue. This project aims to build a simple but effective system using only SQL to analyze transaction data and rank suspicious activities by a "fraud score," providing a clear list for analysts to review.

## Tools & Technologies
* **Database:** MySQL
* **IDE:** MySQL Workbench

## Process
1.  **Data Loading & Preparation:** Sourced a dataset of over 284,000 transactions from Kaggle. Troubleshot a frozen import process by bypassing the GUI wizard and using the powerful `LOAD DATA LOCAL INFILE` command.
2.  **Rule-Based Analysis:** Wrote a series of SQL queries to identify specific fraud patterns:
    * **High-Frequency Transactions:** Used `GROUP BY` and `HAVING` to find moments with an abnormally high number of transactions (card testing).
    * **Unusually Large Transactions:** Used a Common Table Expression (CTE) to calculate the average transaction amount and find significant outliers.
3.  **Fraud Scoring:** Combined all rules into a single, comprehensive query using multiple CTEs and `LEFT JOIN`s. A `CASE` statement was used to assign points and calculate a final `total_fraud_score` for each transaction.

## Key Query
This is the final query that calculates the fraud score for all suspicious transactions.

```sql
-- Paste your final, impressive fraud-scoring query here.
```-- Final Project Query: Combining Rules to Create a Fraud Score
USE fraud_project;

WITH HighFrequency AS (
    -- Rule 1: Flag moments in time with more than 10 transactions
    SELECT `Time`
    FROM transactions
    GROUP BY `Time`
    HAVING COUNT(*) > 10
),
LargeTransactions AS (
    -- Rule 2: Flag transactions that are > 50x the overall average amount
    SELECT `Time`, `Amount`
    FROM transactions, (SELECT AVG(Amount) AS overall_avg FROM transactions) AS avg_calc
    WHERE Amount > (avg_calc.overall_avg * 50)
)
-- Calculate a fraud score for transactions that trigger at least one rule
SELECT
    t.*,
    (
        CASE WHEN hf.Time IS NOT NULL THEN 25 ELSE 0 END +
        CASE WHEN lt.Time IS NOT NULL THEN 75 ELSE 0 END
    ) AS total_fraud_score
FROM
    transactions AS t
LEFT JOIN HighFrequency AS hf ON t.Time = hf.Time
LEFT JOIN LargeTransactions AS lt ON t.Time = lt.Time AND t.Amount = lt.Amount
WHERE
    hf.Time IS NOT NULL OR lt.Time IS NOT NULL
ORDER BY
    total_fraud_score DESC,
    t.Amount DESC;
