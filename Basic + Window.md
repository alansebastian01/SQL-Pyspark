In an interview, you want to explain JOINs in a **business-oriented way**, not just give the SQL definition.

You could say:

> “In a relational database, business data is usually stored across multiple tables to avoid duplication. I use SQL JOINs when I need to combine related information from those tables using a common key. For example, in an e-commerce database, customer information may be stored in a `customers` table while purchase information is stored in an `orders` table. I can join them using `customer_id` to analyze which customers placed which orders. The type of JOIN I choose depends on the business question—for example, an INNER JOIN when I only need matching records, or a LEFT JOIN when I also need to identify customers who haven't placed any orders.”

If they ask **“Why do we need different types of joins?”**, a strong answer is:

> “Different JOINs help answer different business questions. INNER JOIN gives me records that exist in both tables. LEFT JOIN keeps all records from my primary table, which is useful for finding missing relationships. For example, if I want customers who haven't purchased anything, I would LEFT JOIN customers to orders and look for customers where the order is NULL.”

### Interview example

Suppose the interviewer says:

**“We have `customers` and `orders`. How would you find customers who never ordered?”**

You could explain your thinking first:

> “Since I need all customers, including those without matching orders, I'd use the `customers` table as my left table and LEFT JOIN it with `orders`. Customers without a matching order will have NULL on the order side, so I'll filter for those NULL values.”

Then write:

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

That's often more impressive than simply writing the query because you're demonstrating that you understand **why** you're choosing the JOIN.

A simple interview formula to remember is:

**Business question → Which tables contain the data? → How are they related? → Which records do I need to keep? → Choose the JOIN.**

For example: *“Find customers who purchased electronics”* → `customers + orders + order_items + products` → connect them through their IDs → only matching purchases matter → `INNER JOIN`.

If you'd like, I can also give you **10 common SQL JOIN interview questions using this e-commerce scenario**, from beginner to advanced.

For **window functions**, interviewers usually want to hear that you understand **why you'd use them instead of `GROUP BY`**, plus a business example.

A strong interview answer would be:

> “I use window functions when I need to perform calculations across a set of related rows while still preserving the individual rows in my result. Unlike `GROUP BY`, which collapses multiple rows into one summary row, window functions allow me to calculate things like rankings, running totals, previous or next values, and averages while keeping the original row-level detail.”

### E-commerce example

Suppose your `orders` table contains:

| order_id | customer_id | order_date | amount |
| -------- | ----------- | ---------- | -----: |
| 101      | 1           | Jan 5      |   $100 |
| 102      | 1           | Jan 10     |    $50 |
| 103      | 1           | Jan 15     |   $200 |
| 104      | 2           | Jan 7      |    $80 |

The business asks:

**“For each customer, show their orders and their running total of spending.”**

You could say in an interview:

> “Because I need to keep every individual order but also calculate cumulative spending for each customer, I would use a window function. I would partition by customer so each customer's calculation is separate, and order by the order date so the running total follows the sequence of purchases.”

```sql
SELECT
    customer_id,
    order_id,
    order_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) AS running_total
FROM orders;
```

The result would conceptually be:

| customer | order | amount | running_total |
| -------- | ----- | -----: | ------------: |
| 1        | 101   |   $100 |          $100 |
| 1        | 102   |    $50 |          $150 |
| 1        | 103   |   $200 |          $350 |
| 2        | 104   |    $80 |           $80 |

### The key difference: `GROUP BY` vs Window Function

This is **very useful to explain in an interview**.

With `GROUP BY`:

```sql
SELECT
    customer_id,
    SUM(amount)
FROM orders
GROUP BY customer_id;
```

You get:

```text
Customer 1 → $350
Customer 2 → $80
```

You've lost the individual orders.

With:

```sql
SUM(amount) OVER(PARTITION BY customer_id)
```

you can get:

```text
Order 101 → $100 → Customer total $350
Order 102 → $50  → Customer total $350
Order 103 → $200 → Customer total $350
```

So a great sentence to memorize is:

> **“`GROUP BY` reduces the number of rows, whereas window functions perform calculations across related rows without losing row-level detail.”**

### Common window functions to know for interviews

| Function       | Business use                          |
| -------------- | ------------------------------------- |
| `ROW_NUMBER()` | Give each row a unique sequence       |
| `RANK()`       | Rank customers/products               |
| `DENSE_RANK()` | Rank without gaps                     |
| `SUM() OVER()` | Running/cumulative totals             |
| `AVG() OVER()` | Compare against an average            |
| `LAG()`        | Compare current row with previous row |
| `LEAD()`       | Compare current row with next row     |

For example, if an interviewer asks **“Find the top 3 products in each category by sales,”** that's a strong clue that you may need:

```sql
RANK() OVER(
    PARTITION BY category
    ORDER BY sales DESC
)
```

The mental shortcut is:

**JOIN → I need data from multiple tables.**

**GROUP BY → I need to summarize rows.**

**Window function → I need calculations/rankings/comparisons while keeping the individual rows.**

That's a very solid distinction to communicate in a SQL interview.

