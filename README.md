# Bank Transaction SQL Analysis
### A Data & Business Analysis Portfolio Project

**Analyst:** Atashi Sharma  
**Dataset:** Synthetic Bank Transaction Dataset (Kaggle — Fraud Detection)  
**Tool:** SQLite via DB Browser  
**Period Covered:** January 2023 – January 2024  
**Rows Analysed:** 2,512 transactions | 495 unique customers

---

## Project Overview

This project analyses 12 months of retail banking transaction data using SQL, with a consulting and financial services lens. Every query is designed to answer a real business question a bank or management consultant would ask — not just demonstrate technical capability.

The analysis covers five stages:
1. Data Exploration — validating structure, quality, and scope
2. Customer Segmentation — identifying high, medium, and low value customers
3. Transaction Pattern Analysis — understanding spending trends and channel behaviour
4. Risk Flagging — identifying anomalous transactions using multi-signal scoring
5. Business Summary — consolidated findings and recommendations

---

## Dataset Overview

| Column | Type | Description |
|---|---|---|
| TransactionID | Text | Unique transaction identifier |
| AccountID | Text | Customer account identifier |
| TransactionAmount | REAL | Value of the transaction |
| TransactionDate | Text (YYYY-MM-DD) | Timestamp of transaction |
| TransactionType | Text | Credit or Debit |
| Channel | Text | ATM, Branch, or Online |
| LoginAttempts | Integer | Authentication attempts before transaction |
| AccountBalance | REAL | Balance at time of transaction |
| CustomerOccupation | Text | Self-reported occupation (unreliable — see data quality) |
| MerchantID, DeviceID, IP Address | Text | Transaction metadata |

---

## Data Quality Findings

Three data quality issues were identified before analysis. Flagging these proactively is standard practice in a client engagement.

**1. CustomerOccupation is unreliable**  
Individual AccountIDs appear with multiple different occupation labels across transactions. One customer showed four different occupations (Student, Doctor, Engineer, Retired) across their transaction history. This indicates the attribute was randomly assigned per transaction in the synthetic dataset, not per customer. Occupation was excluded from all customer-level analysis.

**2. Day-of-week distribution is anomalous**  
Monday accounts for 43% of all transactions (1,070 of 2,512). All remaining transactions fall on Tuesday–Friday with no weekend activity. This does not reflect real retail banking behaviour and was flagged as a data generation artifact. Day-of-week analysis was excluded from conclusions.

**3. January 2024 is an incomplete month**  
Only 13 transactions are recorded on a single day (January 1st, 2024). This month was excluded from all trend and time-series analysis to avoid distorting monthly averages.

---

## Stage 1: Data Exploration

**Objective:** Confirm data integrity and understand dataset scope before analysis.

**Key findings:**
- 2,512 transactions across 12 months with no null values and no duplicate TransactionIDs
- 77% of transactions are Debits — consistent with a current account (spending-heavy) book
- Channel mix is near-equal: ATM 34.6%, Branch 34.6%, Online 32.3%
- Transaction amounts range from £0.26 to £1,919 with an average of £297
- Total transaction volume across the year: £747,556
- Four customer segments by occupation (unreliable — see data quality): Doctor, Engineer, Student, Retired — each approximately 350–365 customers

**SQL techniques used:** COUNT, MIN, MAX, AVG, SUM, GROUP BY, ROUND, subqueries for percentages, PRAGMA table_info

---

## Stage 2: Customer Segmentation

**Objective:** Classify customers into value tiers based on financial behaviour, and understand what distinguishes each segment.

**Segmentation logic:**

| Segment | Threshold (Total Annual Spend) | Customers | % of Base |
|---|---|---|---|
| High Value | ≥ £3,000 | 41 | 8.3% |
| Medium Value | £1,500 – £2,999 | 182 | 36.8% |
| Low Value | < £1,500 | 272 | 54.9% |

**Key findings:**

| Segment | Avg Total Spend | Avg Balance | Avg Transactions |
|---|---|---|---|
| High Value | £3,511 | £5,093 | 8.3 |
| Medium Value | £2,093 | £5,018 | 6.2 |
| Low Value | £819 | £5,225 | 3.9 |

**Critical insight:** Average account balance is nearly identical across all three segments (£5,018–£5,225). High Value customers are not wealthier — they are more behaviourally engaged. Low Value customers hold similar balances but transact far less frequently.

**Business implication:** Low Value customers represent a product engagement opportunity. They have the financial capacity to transact more but are not doing so through this bank. The strategic question is: where are they spending instead?

**SQL techniques used:** CASE WHEN for segmentation, GROUP BY with aggregation, subqueries, COUNT(DISTINCT) for unique customer counts, nested subqueries to resolve GROUP BY alias errors

---

## Stage 3: Transaction Pattern Analysis

**Objective:** Understand how transaction behaviour changes over time and across channels.

**Monthly trends (Jan 2023 – Dec 2023):**
- Transaction volumes are broadly stable: 161–226 transactions per month
- April 2023 is a notable outlier — only 161 transactions and £41,004 total volume, the lowest of any complete month. The dip is consistent across all three channels, ruling out a channel-specific issue
- August–September 2023 shows a spending uptick — September has the highest average transaction value (£340) and second highest monthly volume (£72,832)

**Channel analysis:**
- ATM, Branch, and Online channels maintain near-equal share throughout the year — no structural shift in customer behaviour
- ATM credits account for only 73 transactions vs 760 ATM debits — ATM is almost exclusively a cash withdrawal channel, consistent with real-world behaviour
- Average transaction values are consistent across channels (£277–£317), indicating customers do not use specific channels for larger or smaller transactions

**SQL techniques used:** STRFTIME for date extraction, multi-column GROUP BY, WHERE vs HAVING, ORDER BY with aliases, date filtering to exclude incomplete months

---

## Stage 4: Risk Flagging

**Objective:** Identify transactions with multiple anomaly signals that warrant fraud investigation.

**Two risk signals were defined:**

| Signal | Definition | Transactions Flagged |
|---|---|---|
| Anomalous login attempts | LoginAttempts ≥ 3 | 95 transactions |
| High spend deviation | Amount > 3x customer's personal average | ~80 transactions |

95% of transactions have exactly 1 login attempt. The 5% with 3 or more attempts are a statistically distinct group.

**Risk scoring:** Each signal contributes 1 point to a composite risk score (max 2). Transactions scoring 2 have both signals firing simultaneously.

**Results:**

| Risk Score | Transactions |
|---|---|
| 2 (both signals) | 3 |
| 1 (one signal) | ~150 |
| 0 (clean) | ~2,360 |

**The three highest-risk transactions:**

| Transaction | Account | Amount | Avg Spend | Deviation | Login Attempts | Channel |
|---|---|---|---|---|---|---|
| TX000899 | AC00083 | £1,531 | £459 | +£1,072 | 4 | Online |
| TX001214 | AC00170 | £1,192 | £247 | +£945 | 5 | Branch |
| TX000232 | AC00430 | £706 | £210 | +£495 | 3 | Branch |

TX001214 is the highest priority case — it appeared in every risk query across this stage, with the maximum login attempts (5) and a spend nearly 5x the customer's personal average.

**SQL techniques used:** CTEs (WITH syntax), window functions (AVG OVER PARTITION BY), RANK() OVER for within-customer ranking, multi-CTE chaining, LEFT JOIN to preserve all customers in summary output, binary flag scoring

---

## Stage 5: Risk by Customer Segment

**Objective:** Understand whether fraud risk is distributed evenly or concentrated in specific customer segments.

| Segment | Customers | Avg Spend | Avg Balance | Customers with Flags | % Flagged |
|---|---|---|---|---|---|
| High Value | 41 | £3,511 | £5,093 | 21 | 51.2% |
| Medium Value | 182 | £2,093 | £5,018 | 63 | 34.6% |
| Low Value | 272 | £819 | £5,225 | 41 | 15.1% |

**Key finding:** Risk is directly correlated with value. Over half of High Value customers have at least one flagged transaction, compared to 15% of Low Value customers. This is partly explained by activity levels — High Value customers transact more frequently (8.3 times on average), creating more exposure.

---

## Business Recommendations

Based on the analysis, three recommendations are presented in order of priority:

**1. Prioritise fraud monitoring on High Value accounts**  
51% of High Value customers carry at least one risk flag. Given their higher transaction frequency and balance levels, the financial exposure from a successful fraud event on these accounts is disproportionately large. Enhanced transaction monitoring, step-up authentication on anomalous transactions, and proactive outreach should be prioritised for this segment.

**2. Investigate the April 2023 transaction dip**  
April shows a consistent 25–30% drop in transaction volume across all channels. In a real dataset, this would require root cause analysis — potential explanations include a system outage, a data extraction issue, or a genuine demand reduction. Until explained, April figures should be excluded from any annualised projections.

**3. Develop an engagement strategy for Low Value customers**  
Low Value customers hold similar average balances (£5,225) to High Value customers (£5,093) but transact at less than half the frequency. This suggests wallet share is being lost to competitor banks or alternative payment methods. A targeted engagement campaign — personalised offers, lower friction digital onboarding, or product recommendations — could increase transaction frequency without requiring customer acquisition spend.

---

## SQL Skills Demonstrated

| Skill | Stage Used |
|---|---|
| Aggregate functions (COUNT, SUM, AVG, MIN, MAX) | 1, 2, 3 |
| CASE WHEN for classification | 2, 4 |
| Subqueries and nested queries | 2, 3, 4 |
| GROUP BY, HAVING, ORDER BY | 1, 2, 3 |
| STRFTIME for date extraction | 3 |
| WHERE vs HAVING distinction | 3 |
| CTEs (WITH syntax) | 4, 5 |
| Window functions (OVER, PARTITION BY) | 4, 5 |
| RANK() within partitions | 4 |
| LEFT JOIN vs INNER JOIN | 5 |
| Multi-signal risk scoring | 4, 5 |
| Data quality investigation and flagging | 1, 2, 3 |

---

*This project was built as part of a data analyst portfolio targeting Business Analyst and Data Analyst roles in consulting and financial services.*
