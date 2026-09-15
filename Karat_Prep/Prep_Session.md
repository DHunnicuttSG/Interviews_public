Absolutely. I’ve cleaned up the formatting, corrected the Markdown tables and code blocks, and preserved the SQL so it can be copied directly into a `.md` file.

````markdown
# Karat-Style SQL & Data Science Interview Preparation

This is an excellent schema for a Karat-style interview because it contains:

- Transactional data (`Invoice`, `InvoiceLineItem`)
- User/customer data (`Client`, `Login`)
- Many-to-many relationships (`ClientGoal`, `WorkoutGoal`, `ClientWorkout`)
- Hierarchical data (`ExerciseCategory`)
- NULL values (`BirthDate`, addresses, service dates)
- Natural business metrics (revenue, retention, engagement, workout adoption)

For a **2-hour preparation session**, I'd structure it as:

| Topic | Time |
|---|---:|
| Advanced SQL Review | 20 min |
| Window Functions Deep Dive | 25 min |
| Karat-Style SQL Problems | 35 min |
| Analytics & Data Science Scenarios | 25 min |
| Review / Answer Key Discussion | 15 min |

---

# SQL Window Function Cheat Sheet

## `ROW_NUMBER()`

Assigns a unique sequence number within a partition.

```sql
ROW_NUMBER() OVER (
    PARTITION BY ClientId
    ORDER BY InvoiceDate DESC
)
````

Use when:

* Finding latest records
* Removing duplicates
* Top-N analysis

---

## `RANK()`

Leaves gaps when ties exist.

```sql
RANK() OVER (
    ORDER BY Revenue DESC
)
```

Example:

| Revenue | Rank |
| ------: | ---: |
|    1000 |    1 |
|    1000 |    1 |
|     900 |    3 |

---

## `DENSE_RANK()`

No gaps after ties.

```sql
DENSE_RANK() OVER (
    ORDER BY Revenue DESC
)
```

---

## `LAG()`

Returns the previous row's value.

```sql
LAG(MonthRevenue)
OVER (ORDER BY Month)
```

Useful for:

* Month-over-month growth
* Detecting changes

---

## `LEAD()`

Returns the next row's value.

```sql
LEAD(MonthRevenue)
OVER (ORDER BY Month)
```

Useful for:

* Forecasting
* Trend analysis

---

## Running Total

```sql
SUM(Revenue)
OVER (
    ORDER BY InvoiceDate
)
```

---

## Moving Average

```sql
AVG(Revenue)
OVER (
    ORDER BY InvoiceDate
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)
```

---

# Karat-Style SQL Exercises

---

# Question 1

## Find all clients who do not have a login account.

### Skills

* `LEFT JOIN`
* NULL handling

### Solution

```sql
SELECT
    c.ClientId,
    c.FirstName,
    c.LastName
FROM Client c
LEFT JOIN Login l
    ON c.ClientId = l.ClientId
WHERE l.ClientId IS NULL;
```

### Interview Discussion

Why not:

```sql
WHERE l.ClientId = NULL
```

Because NULL comparisons require:

```sql
IS NULL
```

---

# Question 2

## Calculate revenue by client.

### Skills

* Aggregation
* Multi-table joins

### Solution

```sql
SELECT
    c.ClientId,
    CONCAT(c.FirstName, ' ', c.LastName) AS ClientName,
    SUM(ili.Price * ili.Quantity) AS Revenue
FROM Client c
JOIN Invoice i
    ON c.ClientId = i.ClientId
JOIN InvoiceLineItem ili
    ON i.InvoiceId = ili.InvoiceId
GROUP BY
    c.ClientId,
    ClientName;
```

---

# Question 3

## Rank clients by revenue.

### Skills

* `RANK()`
* CTEs
* Aggregation

### Solution

```sql
WITH RevenueCTE AS (
    SELECT
        c.ClientId,
        SUM(ili.Price * ili.Quantity) AS Revenue
    FROM Client c
    JOIN Invoice i
        ON c.ClientId = i.ClientId
    JOIN InvoiceLineItem ili
        ON i.InvoiceId = ili.InvoiceId
    GROUP BY c.ClientId
)

SELECT *,
       RANK() OVER (
           ORDER BY Revenue DESC
       ) AS RevenueRank
FROM RevenueCTE;
```

---

# Question 4

## Find the highest-spending client in each state.

### Skills

* `ROW_NUMBER()`
* CTE
* Partitioning

### Solution

```sql
WITH RevenuePerClient AS (
    SELECT
        c.StateAbbr,
        c.ClientId,
        SUM(ili.Price * ili.Quantity) AS Revenue
    FROM Client c
    JOIN Invoice i
        ON c.ClientId = i.ClientId
    JOIN InvoiceLineItem ili
        ON i.InvoiceId = ili.InvoiceId
    GROUP BY
        c.StateAbbr,
        c.ClientId
)

SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY StateAbbr
               ORDER BY Revenue DESC
           ) AS rn
    FROM RevenuePerClient
) x
WHERE rn = 1;
```

---

# Question 5

## Calculate running revenue over time.

### Skills

* Running totals
* CTEs
* Window functions

### Solution

```sql
WITH DailyRevenue AS (
    SELECT
        InvoiceDate,
        SUM(ili.Price * ili.Quantity) AS Revenue
    FROM Invoice i
    JOIN InvoiceLineItem ili
        ON i.InvoiceId = ili.InvoiceId
    GROUP BY InvoiceDate
)

SELECT
    InvoiceDate,
    Revenue,
    SUM(Revenue)
        OVER (ORDER BY InvoiceDate)
        AS RunningRevenue
FROM DailyRevenue;
```

---

# Question 6

## Calculate month-over-month revenue growth.

### Skills

* `LAG()`
* Aggregation
* Date functions

### Solution

```sql
WITH MonthlyRevenue AS (
    SELECT
        DATE_FORMAT(InvoiceDate, '%Y-%m') AS MonthKey,
        SUM(ili.Price * ili.Quantity) AS Revenue
    FROM Invoice i
    JOIN InvoiceLineItem ili
        ON i.InvoiceId = ili.InvoiceId
    GROUP BY MonthKey
)

SELECT
    MonthKey,
    Revenue,
    LAG(Revenue)
        OVER (ORDER BY MonthKey) AS PreviousRevenue,
    Revenue
        - LAG(Revenue)
            OVER (ORDER BY MonthKey) AS RevenueChange
FROM MonthlyRevenue;
```

---

# Question 7

## Find clients with multiple goals.

### Skills

* `GROUP BY`
* `HAVING`

### Solution

```sql
SELECT
    ClientId,
    COUNT(*) AS NumberOfGoals
FROM ClientGoal
GROUP BY ClientId
HAVING COUNT(*) > 1;
```

---

# Question 8

## Classify clients into age bands using `CASE`.

### Skills

* `CASE`
* NULL handling
* Date functions

### Solution

```sql
SELECT
    ClientId,
    CASE
        WHEN BirthDate IS NULL THEN 'Unknown'
        WHEN TIMESTAMPDIFF(YEAR, BirthDate, CURDATE()) < 30
            THEN 'Under 30'
        WHEN TIMESTAMPDIFF(YEAR, BirthDate, CURDATE()) < 50
            THEN '30-49'
        ELSE '50+'
    END AS AgeGroup
FROM Client;
```

---

# Question 9

## Use `COALESCE()` on BirthDate.

### Skills

* `COALESCE()`
* NULL handling

### Solution

```sql
SELECT
    ClientId,
    COALESCE(
        CAST(BirthDate AS CHAR),
        'Birth Date Missing'
    ) AS BirthDate
FROM Client;
```

### Discussion

Without `COALESCE()`:

```text
NULL
```

would appear.

---

# NULL Questions Karat Loves

## What happens here?

```sql
COUNT(BirthDate)
```

Only counts **non-NULL rows**.

---

## Difference

```sql
COUNT(*)
```

Counts **all rows**.

---

## What happens to `AVG()`?

```sql
AVG(BirthYear)
```

NULL values are ignored.

---

## What happens in joins?

This returns nothing:

```sql
WHERE StateAbbr = NULL
```

Must be:

```sql
WHERE StateAbbr IS NULL
```

---

# Data Science Deep Dive Questions

These are very Karat-like.

---

# A/B Testing Question

Suppose we launch:

**Workout Recommendation Engine**

* **A** = Existing Assignment Process
* **B** = New AI Recommendation Process

## Success Metric

### Workout Adoption Rate

```text
Assigned Workout
       ↓
Completed Workout
```

Metric:

```text
Completed Workouts
------------------
Assigned Workouts
```

### Follow-up Questions

* How long should the test run?
* What sample size is needed?
* What are the guardrail metrics?
* Are there seasonality concerns?

Strong candidates discuss:

* Statistical significance
* Random assignment
* Selection bias

---

# Product Metrics Question

## What KPIs would you monitor for Personal Trainer?

### Acquisition

* New clients

### Engagement

* Active clients
* Workouts assigned

### Retention

* Repeat invoices

### Revenue

* Monthly Recurring Revenue
* Revenue per client

---

# Root Cause Analysis Question

## Revenue dropped 20%.

How would you investigate?

### Step 1 — Confirm Data Quality

Were invoices loaded correctly?

---

### Step 2 — Segment By

* State
* Goal
* Workout Level
* Age Group

---

### Step 3 — Check Drivers

* Fewer clients?
* Lower invoice value?
* Fewer sessions?

---

# Model Evaluation Question

Suppose we predict:

> Will client renew?

## Precision

Of predicted renewals, how many actually renewed?

## Recall

Of all renewals, how many did we identify?

## F1 Score

Balances precision and recall.

## ROC-AUC

Measures overall classification performance.

---

# Homework Exercises

## Easy

1. Find clients with no goals.
2. Find goals with the largest number of clients.
3. Count clients by state.

---

## Medium

1. Find the top 5 clients by revenue.
2. Find the most common goal by age group.
3. Calculate revenue by month.

---

## Hard

### Exercise 1

For every client, calculate:

* Revenue
* Rank within state
* Rank overall

Use:

```sql
RANK()
```

---

### Exercise 2

Find revenue growth between consecutive invoices.

Use:

```sql
LAG()
```

---

### Exercise 3

Find the most popular goal in every state.

Use:

```sql
ROW_NUMBER()
```

---

### Exercise 4

Build a client segmentation report.

Create `CASE` expressions for:

* Age Band
* Revenue Tier
* Goal Count

---

### Exercise 5 — Most Karat-Like

Build a dashboard query showing:

* Client Name
* Number of Goals
* Number of Workouts
* Revenue
* Revenue Rank
* Running Revenue Contribution %

Use:

* CTEs
* `CASE`
* `COALESCE`
* `RANK()`
* Window functions
* Aggregations

That final exercise is very close to the complexity and style of the SQL portion I've seen candidates encounter in Karat data science interviews.

```

**Tip:** If you're saving this as a Markdown file, the content above can be pasted directly into a file such as `karat_sql_interview_prep.md`. A useful next step would be to turn the **5 hard exercises into a separate interview worksheet with the solutions removed**, so it can be used for practice.
```
