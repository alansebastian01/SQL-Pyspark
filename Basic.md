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
