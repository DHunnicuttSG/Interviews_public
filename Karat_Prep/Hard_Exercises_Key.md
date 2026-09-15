Absolutely. Below is a **Karat-style answer key** for the five hard exercises, with the SQL and an explanation of the reasoning behind each solution.

# Karat-Style SQL — Hard Exercises Answer Key

This answer key assumes the following tables:

* `Client`
* `Invoice`
* `InvoiceLineItem`
* `ClientGoal`
* `WorkoutGoal`
* `ClientWorkout`

The solutions use MySQL syntax.

---

# Exercise 1 — Revenue Rank Within State and Overall

## Question

For every client, calculate:

* Revenue
* Rank within state
* Rank overall

Use `RANK()`.

---

## Answer

```sql
WITH RevenuePerClient AS (
    SELECT
        c.ClientId,
        c.FirstName,
        c.LastName,
        c.StateAbbr,
        SUM(ili.Price * ili.Quantity) AS Revenue
    FROM Client c
    JOIN Invoice i
        ON c.ClientId = i.ClientId
    JOIN InvoiceLineItem ili
        ON i.InvoiceId = ili.InvoiceId
    GROUP BY
        c.ClientId,
        c.FirstName,
        c.LastName,
        c.StateAbbr
)

SELECT
    ClientId,
    FirstName,
    LastName,
    StateAbbr,
    Revenue,

    RANK() OVER (
        PARTITION BY StateAbbr
        ORDER BY Revenue DESC
    ) AS StateRank,

    RANK() OVER (
        ORDER BY Revenue DESC
    ) AS OverallRank

FROM RevenuePerClient
ORDER BY OverallRank;
```

## Explanation

The first challenge is that revenue exists at the `InvoiceLineItem` level, while we want revenue at the **client level**.

The CTE calculates:

```sql
SUM(ili.Price * ili.Quantity)
```

for each client.

For example:

| Client | State | Revenue |
| ------ | ----- | ------: |
| Alice  | IN    |    5000 |
| Bob    | IN    |    4000 |
| Carol  | IN    |    4000 |
| Dave   | KY    |    6000 |
| Erin   | KY    |    3000 |

We then use two different window functions.

### Rank within state

```sql
RANK() OVER (
    PARTITION BY StateAbbr
    ORDER BY Revenue DESC
)
```

`PARTITION BY StateAbbr` essentially says:

> Start the ranking over again for each state.

### Overall rank

```sql
RANK() OVER (
    ORDER BY Revenue DESC
)
```

There is no `PARTITION BY`, so everybody competes against everybody else.

## Important Interview Point

`RANK()` handles ties differently from `ROW_NUMBER()`.

If two clients both have $4,000:

```text
1
2
2
4
```

The next rank is 4.

That is often exactly what an interviewer wants you to recognize.

---

# Exercise 2 — Revenue Growth Between Consecutive Invoices

## Question

Find revenue growth between consecutive invoices.

Use `LAG()`.

---

## Answer

```sql
WITH InvoiceRevenue AS (
    SELECT
        i.InvoiceId,
        i.ClientId,
        i.InvoiceDate,
        SUM(ili.Price * ili.Quantity) AS Revenue
    FROM Invoice i
    JOIN InvoiceLineItem ili
        ON i.InvoiceId = ili.InvoiceId
    GROUP BY
        i.InvoiceId,
        i.ClientId,
        i.InvoiceDate
)

SELECT
    InvoiceId,
    ClientId,
    InvoiceDate,
    Revenue,

    LAG(Revenue) OVER (
        PARTITION BY ClientId
        ORDER BY InvoiceDate, InvoiceId
    ) AS PreviousRevenue,

    Revenue -
        LAG(Revenue) OVER (
            PARTITION BY ClientId
            ORDER BY InvoiceDate, InvoiceId
        ) AS RevenueChange,

    ROUND(
        (
            Revenue -
            LAG(Revenue) OVER (
                PARTITION BY ClientId
                ORDER BY InvoiceDate, InvoiceId
            )
        )
        /
        NULLIF(
            LAG(Revenue) OVER (
                PARTITION BY ClientId
                ORDER BY InvoiceDate, InvoiceId
            ),
            0
        ) * 100,
        2
    ) AS RevenueGrowthPercent

FROM InvoiceRevenue
ORDER BY ClientId, InvoiceDate;
```

## Explanation

The first step is to determine the revenue for each invoice.

For example:

| Invoice | Client | Revenue |
| ------- | ------ | ------: |
| 101     | 1      |     500 |
| 102     | 1      |     700 |
| 103     | 1      |     600 |

Then:

```sql
LAG(Revenue)
```

looks backward one row.

The result becomes:

| Invoice | Revenue | PreviousRevenue |
| ------- | ------: | --------------: |
| 101     |     500 |            NULL |
| 102     |     700 |             500 |
| 103     |     600 |             700 |

We can then calculate:

```text
Revenue - PreviousRevenue
```

So:

```text
700 - 500 = +200
600 - 700 = -100
```

## Why `PARTITION BY ClientId`?

This is an important interview concept.

We don't want to compare Alice's invoice with Bob's invoice.

```sql
PARTITION BY ClientId
```

tells SQL:

> Look at the previous invoice for this same client.

## Why `NULLIF()`?

Suppose the previous revenue was zero.

This would cause a division-by-zero error:

```sql
Revenue / PreviousRevenue
```

Using:

```sql
NULLIF(PreviousRevenue, 0)
```

turns zero into `NULL`, preventing the division-by-zero problem.

## Interview Discussion

A strong candidate should mention that the first invoice for each client will have:

```text
PreviousRevenue = NULL
```

because there is no previous invoice.

---

# Exercise 3 — Most Popular Goal in Every State

## Question

Find the most popular goal in every state.

Use `ROW_NUMBER()`.

---

## Answer

```sql
WITH GoalCounts AS (
    SELECT
        c.StateAbbr,
        cg.GoalId,
        COUNT(*) AS ClientCount
    FROM Client c
    JOIN ClientGoal cg
        ON c.ClientId = cg.ClientId
    GROUP BY
        c.StateAbbr,
        cg.GoalId
),

RankedGoals AS (
    SELECT
        StateAbbr,
        GoalId,
        ClientCount,

        ROW_NUMBER() OVER (
            PARTITION BY StateAbbr
            ORDER BY ClientCount DESC
        ) AS rn

    FROM GoalCounts
)

SELECT
    StateAbbr,
    GoalId,
    ClientCount
FROM RankedGoals
WHERE rn = 1
ORDER BY StateAbbr;
```

## Explanation

This problem requires two levels of aggregation.

First, we need to count how many clients have each goal in each state.

For example:

| State | Goal        | Clients |
| ----- | ----------- | ------: |
| IN    | Weight Loss |      50 |
| IN    | Strength    |      35 |
| IN    | Endurance   |      20 |
| KY    | Strength    |      40 |
| KY    | Weight Loss |      25 |

The CTE calculates those counts.

Then we rank the goals within each state:

```sql
ROW_NUMBER() OVER (
    PARTITION BY StateAbbr
    ORDER BY ClientCount DESC
)
```

For Indiana:

```text
Weight Loss    50    1
Strength       35    2
Endurance      20    3
```

For Kentucky:

```text
Strength       40    1
Weight Loss    25    2
```

Finally:

```sql
WHERE rn = 1
```

keeps only the most popular goal.

## Why `ROW_NUMBER()`?

The question specifically asks for the most popular goal.

`ROW_NUMBER()` guarantees that we get one row per state.

However, there is an important interview discussion here.

Suppose two goals are tied:

| Goal        | Clients |
| ----------- | ------: |
| Weight Loss |      50 |
| Strength    |      50 |

`ROW_NUMBER()` will choose only one.

If the business requirement is:

> Return **all goals tied for first place**

then `RANK()` would be better:

```sql
RANK() OVER (
    PARTITION BY StateAbbr
    ORDER BY ClientCount DESC
)
```

That's a very good interview observation.

---

# Exercise 4 — Client Segmentation Report

## Question

Build a client segmentation report.

Create `CASE` expressions for:

* Age Band
* Revenue Tier
* Goal Count

---

## Answer

```sql
WITH ClientRevenue AS (
    SELECT
        c.ClientId,
        c.FirstName,
        c.LastName,
        c.BirthDate,

        COALESCE(
            SUM(ili.Price * ili.Quantity),
            0
        ) AS Revenue

    FROM Client c

    LEFT JOIN Invoice i
        ON c.ClientId = i.ClientId

    LEFT JOIN InvoiceLineItem ili
        ON i.InvoiceId = ili.InvoiceId

    GROUP BY
        c.ClientId,
        c.FirstName,
        c.LastName,
        c.BirthDate
),

GoalCounts AS (
    SELECT
        ClientId,
        COUNT(*) AS GoalCount
    FROM ClientGoal
    GROUP BY ClientId
)

SELECT
    cr.ClientId,
    cr.FirstName,
    cr.LastName,
    cr.Revenue,

    COALESCE(gc.GoalCount, 0) AS GoalCount,

    CASE
        WHEN cr.BirthDate IS NULL
            THEN 'Unknown'

        WHEN TIMESTAMPDIFF(
            YEAR,
            cr.BirthDate,
            CURDATE()
        ) < 30
            THEN 'Under 30'

        WHEN TIMESTAMPDIFF(
            YEAR,
            cr.BirthDate,
            CURDATE()
        ) < 50
            THEN '30-49'

        ELSE '50+'
    END AS AgeBand,

    CASE
        WHEN cr.Revenue = 0
            THEN 'No Revenue'

        WHEN cr.Revenue < 1000
            THEN 'Low'

        WHEN cr.Revenue < 5000
            THEN 'Medium'

        ELSE 'High'
    END AS RevenueTier

FROM ClientRevenue cr

LEFT JOIN GoalCounts gc
    ON cr.ClientId = gc.ClientId

ORDER BY cr.Revenue DESC;
```

## Explanation

This is a good example of a query that combines several concepts.

We first calculate revenue per client.

```sql
SUM(ili.Price * ili.Quantity)
```

Then we separately calculate the number of goals:

```sql
COUNT(*)
```

We keep those calculations separate using CTEs.

That makes the query easier to understand and avoids accidentally multiplying rows because of multiple joins.

---

## Age Band

The `CASE` expression converts an exact age into a business-friendly category:

```sql
CASE
    WHEN BirthDate IS NULL THEN 'Unknown'
    WHEN age < 30 THEN 'Under 30'
    WHEN age < 50 THEN '30-49'
    ELSE '50+'
END
```

This is an example of **feature engineering**.

Raw data:

```text
BirthDate = 1982-04-12
```

becomes a useful analytical feature:

```text
AgeBand = 30-49
```

---

## Revenue Tier

The same concept applies to revenue.

Instead of displaying:

```text
Revenue = $7,425
```

we create:

```text
RevenueTier = High
```

This can be useful for:

* Marketing
* Customer segmentation
* Retention analysis
* Revenue forecasting

---

## Why use `LEFT JOIN`?

This is another important interview concept.

Suppose a client has:

* No invoices
* No goals

We still want that client to appear in the report.

Therefore:

```sql
LEFT JOIN
```

is appropriate.

An `INNER JOIN` would remove clients who don't have matching records.

---

# Exercise 5 — Most Karat-Like Dashboard Query

## Question

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

---

# Answer

```sql
WITH GoalCounts AS (
    SELECT
        ClientId,
        COUNT(*) AS GoalCount
    FROM ClientGoal
    GROUP BY ClientId
),

WorkoutCounts AS (
    SELECT
        ClientId,
        COUNT(*) AS WorkoutCount
    FROM ClientWorkout
    GROUP BY ClientId
),

ClientRevenue AS (
    SELECT
        i.ClientId,
        SUM(ili.Price * ili.Quantity) AS Revenue
    FROM Invoice i
    JOIN InvoiceLineItem ili
        ON i.InvoiceId = ili.InvoiceId
    GROUP BY i.ClientId
),

ClientMetrics AS (
    SELECT
        c.ClientId,
        CONCAT(
            c.FirstName,
            ' ',
            c.LastName
        ) AS ClientName,

        COALESCE(gc.GoalCount, 0) AS GoalCount,

        COALESCE(wc.WorkoutCount, 0) AS WorkoutCount,

        COALESCE(cr.Revenue, 0) AS Revenue

    FROM Client c

    LEFT JOIN GoalCounts gc
        ON c.ClientId = gc.ClientId

    LEFT JOIN WorkoutCounts wc
        ON c.ClientId = wc.ClientId

    LEFT JOIN ClientRevenue cr
        ON c.ClientId = cr.ClientId
)

SELECT
    ClientId,
    ClientName,
    GoalCount,
    WorkoutCount,
    Revenue,

    RANK() OVER (
        ORDER BY Revenue DESC
    ) AS RevenueRank,

    SUM(Revenue) OVER (
        ORDER BY Revenue DESC
        ROWS BETWEEN UNBOUNDED PRECEDING
        AND CURRENT ROW
    ) AS RunningRevenue,

    SUM(Revenue) OVER (
        ORDER BY Revenue DESC
        ROWS BETWEEN UNBOUNDED PRECEDING
        AND CURRENT ROW
    )
    /
    NULLIF(
        SUM(Revenue) OVER (),
        0
    ) * 100 AS RunningRevenueContributionPercent

FROM ClientMetrics

ORDER BY RevenueRank;
```

---

# Understanding Exercise 5

This is the most important exercise in the set because it combines several SQL techniques.

The query is essentially built in layers.

```text
Client
   |
   +---- Goal Counts
   |
   +---- Workout Counts
   |
   +---- Revenue
   |
   ↓
Client Metrics
   |
   ↓
Window Functions
   |
   ↓
Dashboard Report
```

---

## Step 1 — Calculate Goal Count

```sql
WITH GoalCounts AS (
    SELECT
        ClientId,
        COUNT(*) AS GoalCount
    FROM ClientGoal
    GROUP BY ClientId
)
```

Example:

| ClientId | GoalCount |
| -------: | --------: |
|        1 |         3 |
|        2 |         1 |
|        3 |         5 |

---

# Step 2 — Calculate Workout Count

```sql
WorkoutCounts AS (
    SELECT
        ClientId,
        COUNT(*) AS WorkoutCount
    FROM ClientWorkout
    GROUP BY ClientId
)
```

Example:

| ClientId | WorkoutCount |
| -------: | -----------: |
|        1 |           15 |
|        2 |            8 |
|        3 |           22 |

---

# Step 3 — Calculate Revenue

```sql
ClientRevenue AS (
    SELECT
        i.ClientId,
        SUM(ili.Price * ili.Quantity) AS Revenue
    FROM Invoice i
    JOIN InvoiceLineItem ili
        ON i.InvoiceId = ili.InvoiceId
    GROUP BY i.ClientId
)
```

This converts invoice line-item data into a client-level metric.

---

# Step 4 — Combine the Metrics

The `ClientMetrics` CTE combines everything:

```sql
ClientMetrics AS (
    SELECT
        ...
    FROM Client c
    LEFT JOIN GoalCounts gc
        ON c.ClientId = gc.ClientId
    LEFT JOIN WorkoutCounts wc
        ON c.ClientId = wc.ClientId
    LEFT JOIN ClientRevenue cr
        ON c.ClientId = cr.ClientId
)
```

The result might look like:

| Client      | Goals | Workouts | Revenue |
| ----------- | ----: | -------: | ------: |
| Alice Smith |     3 |       15 |    7500 |
| Bob Jones   |     1 |        8 |    4000 |
| Carol Brown |     5 |       22 |    9000 |

---

# Step 5 — Revenue Rank

```sql
RANK() OVER (
    ORDER BY Revenue DESC
)
```

If the data is:

| Client | Revenue |
| ------ | ------: |
| Carol  |    9000 |
| Alice  |    7500 |
| Bob    |    4000 |

The rank is:

| Client | Revenue | Rank |
| ------ | ------: | ---: |
| Carol  |    9000 |    1 |
| Alice  |    7500 |    2 |
| Bob    |    4000 |    3 |

---

# Step 6 — Running Revenue

This is where the window-function knowledge becomes more advanced.

```sql
SUM(Revenue) OVER (
    ORDER BY Revenue DESC
    ROWS BETWEEN UNBOUNDED PRECEDING
    AND CURRENT ROW
)
```

Suppose:

| Client | Revenue |
| ------ | ------: |
| Carol  |    9000 |
| Alice  |    7500 |
| Bob    |    4000 |

The running total becomes:

| Client | Revenue | Running Revenue |
| ------ | ------: | --------------: |
| Carol  |    9000 |            9000 |
| Alice  |    7500 |           16500 |
| Bob    |    4000 |           20500 |

The phrase:

```sql
UNBOUNDED PRECEDING
```

means:

> Start at the first row.

And:

```sql
CURRENT ROW
```

means:

> Continue through the current row.

---

# Step 7 — Running Revenue Contribution %

Now we take:

```text
Running Revenue
----------------------- × 100
Total Revenue
```

Suppose total revenue is:

```text
$20,500
```

Carol contributes:

```text
$9,000 / $20,500 × 100
= 43.90%
```

After Alice:

```text
$16,500 / $20,500 × 100
= 80.49%
```

After Bob:

```text
$20,500 / $20,500 × 100
= 100%
```

This creates a very useful business metric.

---

# Final Expected Output

The final dashboard might look like:

| Client      | Goals | Workouts | Revenue | Rank | Running Revenue | Running Contribution |
| ----------- | ----: | -------: | ------: | ---: | --------------: | -------------------: |
| Carol Brown |     5 |       22 |  $9,000 |    1 |          $9,000 |               43.90% |
| Alice Smith |     3 |       15 |  $7,500 |    2 |         $16,500 |               80.49% |
| Bob Jones   |     1 |        8 |  $4,000 |    3 |         $20,500 |              100.00% |

---

# What the Interviewer Is Testing

The interviewer isn't simply testing whether you know SQL syntax.

They are testing whether you can break a complicated analytical problem into manageable pieces.

A strong thought process is:

```text
1. What is the grain of my data?
        ↓
2. What metric do I need?
        ↓
3. Do I need aggregation?
        ↓
4. Can I isolate calculations with CTEs?
        ↓
5. Do I need a window function?
        ↓
6. Do I need partitioning?
        ↓
7. How should NULL values behave?
        ↓
8. What business question does the result answer?
```

---

# Key Interview Takeaways

## CTEs

Use CTEs to break complicated queries into logical steps.

```sql
WITH Something AS (...)
```

Think:

> "Build an intermediate result that I can use later."

---

## `GROUP BY`

Use it when you want to collapse multiple rows into summary rows.

```sql
GROUP BY ClientId
```

Think:

> "Give me one result per client."

---

## `RANK()`

Use it when you need ranking and want ties to share the same rank.

```sql
RANK() OVER (
    ORDER BY Revenue DESC
)
```

---

## `ROW_NUMBER()`

Use it when you need exactly one sequential number per row.

```sql
ROW_NUMBER() OVER (
    PARTITION BY StateAbbr
    ORDER BY Revenue DESC
)
```

Think:

> "Give me the #1 record for each group."

---

## `LAG()`

Use it to compare the current row with a previous row.

```sql
LAG(Revenue) OVER (
    ORDER BY InvoiceDate
)
```

Think:

> "What happened last time?"

---

## `COALESCE()`

Use it to replace NULL with a useful value.

```sql
COALESCE(GoalCount, 0)
```

Think:

> "If there isn't a value, give me this instead."

---

## `CASE`

Use it to turn raw values into business categories.

```sql
CASE
    WHEN Revenue < 1000 THEN 'Low'
    WHEN Revenue < 5000 THEN 'Medium'
    ELSE 'High'
END
```

Think:

> "Turn raw data into a business-friendly classification."

---

# The Most Important Karat Interview Skill

When given a complicated SQL problem, **don't immediately start writing one enormous SELECT statement.**

Instead, explain your approach:

> "First, I'll determine the grain of the data. Then I'll calculate each metric independently using CTEs. Once I have one row per client, I'll combine those metrics and use window functions for ranking and running calculations."

That explanation demonstrates the type of structured analytical thinking interviewers are looking for.
