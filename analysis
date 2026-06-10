-- ============================================================
-- Bank Transaction Risk Analysis
-- Analyst: Atashi Sharma
-- Dataset: Synthetic Bank Transaction Dataset (Kaggle)
-- Tool: SQLite via DB Browser
-- Period: January 2023 - January 2024
-- ============================================================
 
 
-- ============================================================
-- STAGE 1: DATA EXPLORATION
-- Objective: Validate data integrity and understand dataset scope
-- ============================================================
 
-- 1.1 Total transaction count
SELECT COUNT(*) AS total_transactions
FROM bank_transactions;
 
-- 1.2 Inspect column names and data types
PRAGMA table_info(bank_transactions);
 
-- 1.3 Check for duplicate TransactionIDs (data quality check)
SELECT TransactionID, COUNT(*) AS occurrences
FROM bank_transactions
GROUP BY TransactionID
HAVING COUNT(*) > 1;
 
-- 1.4 Date range covered by the dataset
SELECT 
    MIN(TransactionDate) AS earliest_transaction,
    MAX(TransactionDate) AS latest_transaction
FROM bank_transactions;
 
-- 1.5 Transaction type breakdown (Debit vs Credit)
SELECT 
    TransactionType,
    COUNT(*) AS count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM bank_transactions), 2) AS percentage
FROM bank_transactions
GROUP BY TransactionType;
 
-- 1.6 Channel breakdown (ATM, Branch, Online)
SELECT 
    Channel,
    COUNT(*) AS count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM bank_transactions), 2) AS percentage
FROM bank_transactions
GROUP BY Channel
ORDER BY count DESC;
 
-- 1.7 Customer occupation distribution
-- Note: Occupation is unreliable in this dataset - individual accounts 
-- appear with multiple different occupations across transactions,
-- indicating it was randomly assigned per transaction, not per customer.
-- Excluded from customer-level analysis.
SELECT 
    CustomerOccupation,
    COUNT(*) AS transaction_count,
    COUNT(DISTINCT AccountID) AS unique_customers
FROM bank_transactions
GROUP BY CustomerOccupation
ORDER BY unique_customers DESC;
 
-- 1.8 Transaction amount summary statistics
SELECT 
    ROUND(MIN(TransactionAmount), 2) AS min_amount,
    ROUND(MAX(TransactionAmount), 2) AS max_amount,
    ROUND(AVG(TransactionAmount), 2) AS avg_amount,
    ROUND(SUM(TransactionAmount), 2) AS total_volume
FROM bank_transactions;
 
-- 1.9 Top 10 accounts by total spend (initial profile)
SELECT 
    AccountID,
    COUNT(*) AS transaction_count,
    ROUND(SUM(TransactionAmount), 2) AS total_spend,
    ROUND(AVG(TransactionAmount), 2) AS avg_transaction,
    ROUND(MAX(AccountBalance), 2) AS max_balance
FROM bank_transactions
GROUP BY AccountID
ORDER BY total_spend DESC
LIMIT 10;
 
 
-- ============================================================
-- STAGE 2: CUSTOMER SEGMENTATION
-- Objective: Classify customers into value tiers based on
-- annual spend behaviour
-- ============================================================
 
-- 2.1 Assign segment to each customer based on total annual spend
SELECT 
    AccountID,
    ROUND(SUM(TransactionAmount), 2) AS total_spend,
    CASE 
        WHEN SUM(TransactionAmount) >= 3000 THEN 'High Value'
        WHEN SUM(TransactionAmount) >= 1500 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS customer_segment
FROM bank_transactions
GROUP BY AccountID
ORDER BY total_spend DESC;
 
-- 2.2 Segment distribution - how many customers in each tier
-- Uses window function OVER() to calculate percentages without a subquery
SELECT 
    customer_segment,
    COUNT(*) AS customer_count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2) AS percentage
FROM (
    SELECT 
        AccountID,
        CASE 
            WHEN SUM(TransactionAmount) >= 3000 THEN 'High Value'
            WHEN SUM(TransactionAmount) >= 1500 THEN 'Medium Value'
            ELSE 'Low Value'
        END AS customer_segment
    FROM bank_transactions
    GROUP BY AccountID
) 
GROUP BY customer_segment
ORDER BY customer_count DESC;
 
-- 2.3 Average financial profile by occupation
-- Note: Occupation unreliable - see Stage 1 data quality note
SELECT 
    CustomerOccupation,
    ROUND(AVG(TransactionAmount), 2) AS avg_transaction,
    ROUND(AVG(AccountBalance), 2) AS avg_balance,
    COUNT(DISTINCT AccountID) AS customers
FROM bank_transactions
GROUP BY CustomerOccupation
ORDER BY avg_balance DESC;
 
-- 2.4 Segment breakdown by occupation (informational only - occupation unreliable)
SELECT 
    CustomerOccupation,
    customer_segment,
    COUNT(DISTINCT AccountID) AS customer_count
FROM (
    SELECT 
        AccountID,
        CustomerOccupation,
        CASE 
            WHEN SUM(TransactionAmount) >= 3000 THEN 'High Value'
            WHEN SUM(TransactionAmount) >= 1500 THEN 'Medium Value'
            ELSE 'Low Value'
        END AS customer_segment
    FROM bank_transactions
    GROUP BY AccountID, CustomerOccupation
)
GROUP BY CustomerOccupation, customer_segment
ORDER BY CustomerOccupation, customer_segment;
 
-- 2.5 Data quality check: accounts with multiple occupation labels
-- Confirms occupation field is unreliable for customer-level analysis
SELECT 
    AccountID,
    COUNT(DISTINCT CustomerOccupation) AS occupation_count
FROM bank_transactions
GROUP BY AccountID
HAVING COUNT(DISTINCT CustomerOccupation) > 1
LIMIT 10;
 
-- 2.6 Full segment profile - spend, balance, and transaction frequency per tier
-- Key insight: balance is near-identical across segments; 
-- High Value customers transact more, not hold more money
SELECT
    customer_segment,
    COUNT(*) AS customer_count,
    ROUND(AVG(total_spend), 2) AS avg_spend,
    ROUND(AVG(avg_balance), 2) AS avg_balance,
    ROUND(AVG(transaction_count), 2) AS avg_transactions
FROM (
    SELECT
        AccountID,
        COUNT(*) AS transaction_count,
        ROUND(SUM(TransactionAmount), 2) AS total_spend,
        ROUND(AVG(AccountBalance), 2) AS avg_balance,
        CASE
            WHEN SUM(TransactionAmount) >= 3000 THEN 'High Value'
            WHEN SUM(TransactionAmount) >= 1500 THEN 'Medium Value'
            ELSE 'Low Value'
        END AS customer_segment
    FROM bank_transactions
    GROUP BY AccountID
)
GROUP BY customer_segment
ORDER BY avg_spend DESC;
 
 
-- ============================================================
-- STAGE 3: TRANSACTION PATTERN ANALYSIS
-- Objective: Understand spending trends over time and across channels
-- Note: January 2024 excluded (single day, incomplete month)
-- Note: Day-of-week excluded (Monday = 43% of transactions, 
--       anomalous distribution, likely a data generation artifact)
-- ============================================================
 
-- 3.1 Monthly transaction volume and value trends (Jan 2023 - Dec 2023)
SELECT 
    STRFTIME('%Y-%m', TransactionDate) AS month,
    COUNT(*) AS transaction_count,
    ROUND(SUM(TransactionAmount), 2) AS total_volume,
    ROUND(AVG(TransactionAmount), 2) AS avg_transaction
FROM bank_transactions
GROUP BY STRFTIME('%Y-%m', TransactionDate)
ORDER BY month;
 
-- 3.2 Day-of-week distribution
-- Included for completeness but excluded from conclusions
-- Monday accounts for 43% of all transactions - not representative of real behaviour
SELECT 
    CASE CAST(STRFTIME('%w', TransactionDate) AS INTEGER)
        WHEN 0 THEN 'Sunday'
        WHEN 1 THEN 'Monday'
        WHEN 2 THEN 'Tuesday'
        WHEN 3 THEN 'Wednesday'
        WHEN 4 THEN 'Thursday'
        WHEN 5 THEN 'Friday'
        WHEN 6 THEN 'Saturday'
    END AS day_of_week,
    COUNT(*) AS transaction_count,
    ROUND(AVG(TransactionAmount), 2) AS avg_transaction
FROM bank_transactions
GROUP BY STRFTIME('%w', TransactionDate)
ORDER BY CAST(STRFTIME('%w', TransactionDate) AS INTEGER);
 
-- 3.3 Monthly volume by channel (excludes incomplete January 2024)
SELECT 
    STRFTIME('%Y-%m', TransactionDate) AS month,
    Channel,
    COUNT(*) AS transaction_count,
    ROUND(SUM(TransactionAmount), 2) AS total_volume
FROM bank_transactions
WHERE TransactionDate < '2024-01-01'
GROUP BY STRFTIME('%Y-%m', TransactionDate), Channel
ORDER BY month, Channel;
 
-- 3.4 Channel and transaction type cross-analysis
-- Checks whether transaction size differs meaningfully by channel
SELECT 
    Channel,
    TransactionType,
    COUNT(*) AS count,
    ROUND(AVG(TransactionAmount), 2) AS avg_amount,
    ROUND(MIN(TransactionAmount), 2) AS min_amount,
    ROUND(MAX(TransactionAmount), 2) AS max_amount
FROM bank_transactions
GROUP BY Channel, TransactionType
ORDER BY Channel, TransactionType;
 
-- 3.5 Most active customers - accounts with 8 or more transactions
SELECT
    AccountID,
    COUNT(*) AS transaction_count,
    MIN(TransactionDate) AS first_transaction,
    MAX(TransactionDate) AS last_transaction,
    ROUND(AVG(TransactionAmount), 2) AS avg_transaction
FROM bank_transactions
GROUP BY AccountID
HAVING COUNT(*) >= 8
ORDER BY transaction_count DESC
LIMIT 15;
 
 
-- ============================================================
-- STAGE 4: RISK FLAGGING
-- Objective: Identify transactions with multiple anomaly signals
-- Two signals used:
--   Signal 1 (flag_login): LoginAttempts >= 3
--   Signal 2 (flag_high_spend): TransactionAmount > 3x customer average
--     Note: Threshold = avg + 2*avg = 3x avg (simplified anomaly rule)
--     Standard statistical approach would use mean + 2 standard deviations,
--     but SQLite lacks a native STDEV function.
-- Composite risk score = flag_login + flag_high_spend (max 2)
-- ============================================================
 
-- 4.1 Window function: rank each transaction within its account by spend
-- Uses AVG OVER PARTITION BY to calculate per-customer average without GROUP BY
SELECT
    TransactionID,
    AccountID,
    TransactionAmount,
    TransactionDate,
    ROUND(AVG(TransactionAmount) OVER(PARTITION BY AccountID), 2) AS customer_avg,
    ROUND(TransactionAmount - AVG(TransactionAmount) OVER(PARTITION BY AccountID), 2) AS deviation,
    RANK() OVER(PARTITION BY AccountID ORDER BY TransactionAmount DESC) AS spend_rank
FROM bank_transactions
ORDER BY deviation DESC
LIMIT 20;
 
-- 4.2 Multi-signal risk scoring using CTEs
-- CTE 1 (customer_stats): calculates per-customer average and high spend threshold
-- CTE 2 (risk_flags): joins transactions to stats and applies both signal flags
-- Final SELECT: filters to transactions with at least one flag
WITH customer_stats AS (
    SELECT
        AccountID,
        ROUND(AVG(TransactionAmount), 2) AS avg_spend,
        -- Threshold = 3x customer average (simplified; see stage header note)
        ROUND(AVG(TransactionAmount) + (2 * AVG(TransactionAmount)), 2) AS high_spend_threshold
    FROM bank_transactions
    GROUP BY AccountID
),
risk_flags AS (
    SELECT
        t.TransactionID,
        t.AccountID,
        t.TransactionAmount,
        t.TransactionDate,
        t.Channel,
        t.LoginAttempts,
        c.avg_spend,
        ROUND(t.TransactionAmount - c.avg_spend, 2) AS deviation,
        CASE WHEN t.LoginAttempts >= 3 THEN 1 ELSE 0 END AS flag_login,
        CASE WHEN t.TransactionAmount > c.high_spend_threshold THEN 1 ELSE 0 END AS flag_high_spend
    FROM bank_transactions t
    JOIN customer_stats c ON t.AccountID = c.AccountID
)
SELECT
    TransactionID,
    AccountID,
    TransactionAmount,
    avg_spend,
    deviation,
    Channel,
    TransactionDate,
    LoginAttempts,
    flag_login,
    flag_high_spend,
    (flag_login + flag_high_spend) AS total_risk_flags
FROM risk_flags
WHERE (flag_login + flag_high_spend) >= 1
ORDER BY total_risk_flags DESC, deviation DESC
LIMIT 20;
 
 
-- ============================================================
-- STAGE 5: RISK BY CUSTOMER SEGMENT
-- Objective: Understand whether fraud risk concentrates in
-- specific customer value tiers
-- Note: Rewritten as flat CTE chain for compatibility with
-- standard SQL databases (PostgreSQL, BigQuery, SQL Server)
-- ============================================================
 
-- 5.1 Risk distribution across customer segments
-- CTE 1 (customer_summary): full customer profile with segment label
-- CTE 2 (high_spend_thresholds): per-customer spend threshold
-- CTE 3 (flagged_accounts): accounts with at least one risk signal
-- Final SELECT: LEFT JOIN preserves all customers including unflagged ones
WITH customer_summary AS (
    SELECT
        AccountID,
        COUNT(*) AS transaction_count,
        ROUND(SUM(TransactionAmount), 2) AS total_spend,
        ROUND(AVG(TransactionAmount), 2) AS avg_transaction,
        ROUND(AVG(AccountBalance), 2) AS avg_balance,
        CASE
            WHEN SUM(TransactionAmount) >= 3000 THEN 'High Value'
            WHEN SUM(TransactionAmount) >= 1500 THEN 'Medium Value'
            ELSE 'Low Value'
        END AS segment
    FROM bank_transactions
    GROUP BY AccountID
),
high_spend_thresholds AS (
    SELECT 
        AccountID,
        ROUND(AVG(TransactionAmount) + (2 * AVG(TransactionAmount)), 2) AS threshold
    FROM bank_transactions
    GROUP BY AccountID
),
flagged_accounts AS (
    SELECT DISTINCT t.AccountID
    FROM bank_transactions t
    JOIN high_spend_thresholds h ON t.AccountID = h.AccountID
    WHERE t.LoginAttempts >= 3 
       OR t.TransactionAmount > h.threshold
)
SELECT
    s.segment,
    COUNT(DISTINCT s.AccountID) AS customers,
    ROUND(AVG(s.total_spend), 2) AS avg_total_spend,
    ROUND(AVG(s.avg_balance), 2) AS avg_balance,
    ROUND(AVG(s.transaction_count), 2) AS avg_transactions,
    COUNT(DISTINCT f.AccountID) AS customers_with_flags,
    ROUND(COUNT(DISTINCT f.AccountID) * 100.0 / COUNT(DISTINCT s.AccountID), 2) AS pct_flagged
FROM customer_summary s
LEFT JOIN flagged_accounts f ON s.AccountID = f.AccountID
GROUP BY s.segment
ORDER BY avg_total_spend DESC;
