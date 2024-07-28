Exploratory Data Analysis (EDA) involves analyzing datasets to summarize their main characteristics using various techniques and tools. SQL window functions are particularly powerful for EDA, as they allow you to perform complex calculations across rows related to the current row without collapsing the result set. This capability makes them extremely useful for various types of data analysis tasks, including ranking, aggregating, and filtering data.

Let's dive into the top 10 SQL window function analysis queries that you can use for EDA on an e-commerce database.

### 1. **Calculating Running Total of Orders**

The running total of orders is useful for understanding cumulative sales over time. You can use the `SUM()` window function to calculate it.

```sql
SELECT 
    order_date,
    total_amount,
    SUM(total_amount) OVER (ORDER BY order_date) AS running_total
FROM 
    Orders
ORDER BY 
    order_date;
```

- **Explanation**:
  - This query computes a cumulative sum (`running_total`) of `total_amount` for each order date.
  - The `ORDER BY order_date` clause in the `OVER()` function ensures that the total is calculated in order of the dates.

### 2. **Calculating Moving Average of Order Amounts**

Moving averages help smooth out fluctuations in data, making trends easier to see. Here's how to compute a 7-day moving average of order amounts.

```sql
SELECT 
    order_date,
    total_amount,
    AVG(total_amount) OVER (ORDER BY order_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_average
FROM 
    Orders
ORDER BY 
    order_date;
```

- **Explanation**:
  - The query calculates a 7-day moving average of `total_amount`, considering the current row and the previous six rows (`6 PRECEDING`).
  - This window can be adjusted for different time frames by changing the number of preceding rows.

### 3. **Ranking Customers by Total Spending**

Ranking customers based on their total spending gives insight into who your most valuable customers are. The `RANK()` function is ideal for this task.

```sql
SELECT 
    customer_id,
    SUM(total_amount) AS total_spent,
    RANK() OVER (ORDER BY SUM(total_amount) DESC) AS spending_rank
FROM 
    Orders
GROUP BY 
    customer_id
ORDER BY 
    spending_rank;
```

- **Explanation**:
  - This query ranks customers by the total amount they have spent across all orders.
  - The `RANK()` function assigns ranks, with ties getting the same rank. If you prefer unique ranks, use `DENSE_RANK()`.

### 4. **Calculating Customer Lifetime Value (CLV)**

Customer Lifetime Value is a critical metric for understanding long-term revenue potential from a customer. It can be calculated using window functions to aggregate spending over time.

```sql
SELECT 
    customer_id,
    SUM(total_amount) AS lifetime_value,
    AVG(total_amount) OVER (PARTITION BY customer_id) AS avg_order_value,
    COUNT(order_id) OVER (PARTITION BY customer_id) AS order_count
FROM 
    Orders
GROUP BY 
    customer_id
ORDER BY 
    lifetime_value DESC;
```

- **Explanation**:
  - This query calculates the total spending (`lifetime_value`) for each customer.
  - It also provides the average order value and the total number of orders per customer.

### 5. **Calculating Time to Deliver Orders**

Analyzing delivery times helps in assessing logistics performance. You can use the `DATEDIFF()` function to calculate delivery durations.

```sql
SELECT 
    order_id,
    customer_id,
    order_date,
    delivery_date,
    DATEDIFF(delivery_date, order_date) AS delivery_days
FROM 
    Orders o
JOIN 
    Shipments s ON o.id = s.order_id
WHERE 
    delivery_date IS NOT NULL
ORDER BY 
    delivery_days;
```

- **Explanation**:
  - The query calculates the number of days taken to deliver each order using `DATEDIFF()`.
  - Orders are then sorted by delivery time for further analysis.

### 6. **Identifying Sales Trends by Week**

Weekly sales trends can be visualized using `SUM()` and date functions to aggregate sales by week.

```sql
SELECT 
    YEAR(order_date) AS year,
    WEEK(order_date) AS week,
    SUM(total_amount) AS weekly_sales
FROM 
    Orders
GROUP BY 
    YEAR(order_date), WEEK(order_date)
ORDER BY 
    year, week;
```

- **Explanation**:
  - This query aggregates sales by week using `YEAR()` and `WEEK()` functions.
  - It provides a weekly sales trend overview, which is useful for identifying patterns or seasonal fluctuations.

### 7. **Analyzing Order Frequency by Customer**

Understanding how frequently customers place orders can inform marketing and engagement strategies. The `COUNT()` function helps analyze order frequency.

```sql
SELECT 
    customer_id,
    COUNT(order_id) AS order_count,
    DENSE_RANK() OVER (ORDER BY COUNT(order_id) DESC) AS frequency_rank
FROM 
    Orders
GROUP BY 
    customer_id
ORDER BY 
    frequency_rank;
```

- **Explanation**:
  - This query counts the number of orders placed by each customer.
  - The `DENSE_RANK()` function ranks customers based on order frequency.

### 8. **Detecting Changes in Average Order Value Over Time**

Detecting shifts in average order value over time can indicate changes in customer behavior or pricing strategies.

```sql
SELECT 
    order_date,
    AVG(total_amount) OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS avg_order_value
FROM 
    Orders
ORDER BY 
    order_date;
```

- **Explanation**:
  - This query calculates the average order value (`avg_order_value`) for each order date, considering all previous orders.
  - The `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` clause includes all rows up to the current row, allowing for a complete running average.

### 9. **Comparing Sales Performance Across Regions**

Analyzing sales performance by region can highlight geographic trends and opportunities. Use window functions to compare regional sales.

```sql
SELECT 
    region,
    SUM(total_amount) AS total_sales,
    RANK() OVER (ORDER BY SUM(total_amount) DESC) AS sales_rank
FROM 
    Orders o
JOIN 
    Customers c ON o.customer_id = c.id
GROUP BY 
    region
ORDER BY 
    sales_rank;
```

- **Explanation**:
  - This query sums up sales by region and ranks them using `RANK()`.
  - It joins `Orders` and `Customers` tables to fetch the region associated with each order.

### 10. **Detecting Top Products by Sales Volume**

Identifying top-selling products helps in managing inventory and sales strategies. The `ROW_NUMBER()` function can rank products by sales volume.

```sql
SELECT 
    product_id,
    SUM(quantity) AS total_sold,
    ROW_NUMBER() OVER (ORDER BY SUM(quantity) DESC) AS sales_volume_rank
FROM 
    Order_Items
GROUP BY 
    product_id
ORDER BY 
    sales_volume_rank;
```

- **Explanation**:
  - The query calculates the total quantity sold for each product.
  - The `ROW_NUMBER()` function ranks products by total sales volume.

---

### 1. Running Total of Orders Over Time

```sql
SELECT 
    o.order_date,
    o.total_amount,
    SUM(o.total_amount) OVER (ORDER BY o.order_date) AS running_total
FROM 
    Orders o
ORDER BY 
    o.order_date;
```

- **Purpose**: Analyze cumulative sales over time.
- **Use Case**: Financial analysis to understand revenue growth.

### 2. 7-Day Moving Average of Order Amounts

```sql
SELECT 
    o.order_date,
    o.total_amount,
    AVG(o.total_amount) OVER (ORDER BY o.order_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_average
FROM 
    Orders o
ORDER BY 
    o.order_date;
```

- **Purpose**: Smooth out daily sales fluctuations for trend analysis.
- **Use Case**: Evaluate short-term sales performance and seasonal trends.

### 3. Ranking Customers by Total Spending

```sql
SELECT 
    c.id AS customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    SUM(o.total_amount

) AS total_spent,
    RANK() OVER (ORDER BY SUM(o.total_amount) DESC) AS spending_rank
FROM 
    Customers c
JOIN 
    Orders o ON c.id = o.customer_id
GROUP BY 
    c.id, c.first_name, c.last_name
ORDER BY 
    spending_rank;
```

- **Purpose**: Identify high-value customers.
- **Use Case**: Target marketing campaigns towards top spenders.

### 4. Calculating Customer Lifetime Value (CLV)

```sql
SELECT 
    c.id AS customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    SUM(o.total_amount) AS lifetime_value,
    AVG(o.total_amount) OVER (PARTITION BY c.id) AS avg_order_value,
    COUNT(o.id) OVER (PARTITION BY c.id) AS order_count
FROM 
    Customers c
JOIN 
    Orders o ON c.id = o.customer_id
GROUP BY 
    c.id, c.first_name, c.last_name
ORDER BY 
    lifetime_value DESC;
```

- **Purpose**: Assess the long-term value of customers.
- **Use Case**: Inform retention strategies and personalized offerings.

### 5. Calculating Time to Deliver Orders

```sql
SELECT 
    o.id AS order_id,
    o.customer_id,
    o.order_date,
    s.delivery_date,
    DATEDIFF(s.delivery_date, o.order_date) AS delivery_days
FROM 
    Orders o
JOIN 
    Shipments s ON o.id = s.order_id
WHERE 
    s.delivery_date IS NOT NULL
ORDER BY 
    delivery_days;
```

- **Purpose**: Evaluate delivery efficiency.
- **Use Case**: Optimize logistics and customer satisfaction.

### 6. Identifying Sales Trends by Week

```sql
SELECT 
    YEAR(o.order_date) AS year,
    WEEK(o.order_date) AS week,
    SUM(o.total_amount) AS weekly_sales
FROM 
    Orders o
GROUP BY 
    YEAR(o.order_date), WEEK(o.order_date)
ORDER BY 
    year, week;
```

- **Purpose**: Track weekly sales trends.
- **Use Case**: Plan inventory and marketing strategies around sales cycles.

### 7. Analyzing Order Frequency by Customer

```sql
SELECT 
    c.id AS customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    COUNT(o.id) AS order_count,
    DENSE_RANK() OVER (ORDER BY COUNT(o.id) DESC) AS frequency_rank
FROM 
    Customers c
JOIN 
    Orders o ON c.id = o.customer_id
GROUP BY 
    c.id, c.first_name, c.last_name
ORDER BY 
    frequency_rank;
```

- **Purpose**: Understand customer purchase patterns.
- **Use Case**: Develop customer engagement and loyalty programs.

### 8. Detecting Changes in Average Order Value Over Time

```sql
SELECT 
    o.order_date,
    AVG(o.total_amount) OVER (ORDER BY o.order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS avg_order_value
FROM 
    Orders o
ORDER BY 
    o.order_date;
```

- **Purpose**: Monitor shifts in average purchase size.
- **Use Case**: Adjust pricing strategies and identify emerging buying behaviors.

### 9. Comparing Sales Performance Across Regions

```sql
SELECT 
    c.region,
    SUM(o.total_amount) AS total_sales,
    RANK() OVER (ORDER BY SUM(o.total_amount) DESC) AS sales_rank
FROM 
    Orders o
JOIN 
    Customers c ON o.customer_id = c.id
GROUP BY 
    c.region
ORDER BY 
    sales_rank;
```

- **Purpose**: Identify top-performing regions.
- **Use Case**: Focus marketing efforts on high-potential geographic areas.

### 10. Detecting Top Products by Sales Volume

```sql
SELECT 
    p.id AS product_id,
    p.name AS product_name,
    SUM(oi.quantity) AS total_sold,
    ROW_NUMBER() OVER (ORDER BY SUM(oi.quantity) DESC) AS sales_volume_rank
FROM 
    Order_Items oi
JOIN 
    Products p ON oi.product_id = p.id
GROUP BY 
    p.id, p.name
ORDER BY 
    sales_volume_rank;
```

- **Purpose**: Identify top-selling products.
- **Use Case**: Optimize inventory and product marketing strategies.

---

### Advanced Window Function Analysis Queries

Beyond the basics, you can create more complex queries by combining window functions with other SQL features to perform deeper analysis. Here are some advanced examples:

### 11. Detecting Churn Risk by Analyzing Time Between Purchases

```sql
SELECT 
    customer_id,
    order_date,
    DATEDIFF(
        order_date, 
        LAG(order_date, 1) OVER (PARTITION BY customer_id ORDER BY order_date)
    ) AS days_between_orders
FROM 
    Orders
ORDER BY 
    customer_id, order_date;
```

- **Purpose**: Identify customers with increasing gaps between purchases.
- **Use Case**: Proactively engage with customers showing signs of inactivity.

### 12. Analyzing Revenue Contribution by Product Category

```sql
SELECT 
    p.category,
    SUM(oi.price * oi.quantity) AS category_revenue,
    SUM(oi.price * oi.quantity) * 100.0 / SUM(SUM(oi.price * oi.quantity)) OVER () AS percentage_of_total_revenue
FROM 
    Order_Items oi
JOIN 
    Products p ON oi.product_id = p.id
GROUP BY 
    p.category
ORDER BY 
    category_revenue DESC;
```

- **Purpose**: Evaluate each product category's contribution to total revenue.
- **Use Case**: Focus on high-revenue categories for marketing and product development.

### 13. Calculating Customer Segmentation Based on Purchase Patterns

```sql
SELECT 
    customer_id,
    AVG(total_amount) OVER (PARTITION BY customer_id) AS avg_order_value,
    COUNT(order_id) OVER (PARTITION BY customer_id) AS total_orders,
    CASE 
        WHEN AVG(total_amount) OVER (PARTITION BY customer_id) > 500 THEN 'High Value'
        WHEN COUNT(order_id) OVER (PARTITION BY customer_id) > 5 THEN 'Frequent Buyer'
        ELSE 'Occasional Buyer'
    END AS customer_segment
FROM 
    Orders
ORDER BY 
    customer_id;
```

- **Purpose**: Segment customers based on spending and frequency.
- **Use Case**: Tailor marketing efforts and promotions to different customer segments.

### 14. Analyzing Seasonal Trends in Sales

```sql
SELECT 
    MONTH(order_date) AS month,
    SUM(total_amount) AS monthly_sales,
    SUM(total_amount) * 100.0 / SUM(SUM(total_amount)) OVER () AS percentage_of_annual_sales
FROM 
    Orders
WHERE 
    YEAR(order_date) = 2024
GROUP BY 
    MONTH(order_date)
ORDER BY 
    month;
```

- **Purpose**: Detect seasonal patterns in sales data.
- **Use Case**: Plan inventory and marketing campaigns around seasonal peaks.

### 15. Evaluating Employee Performance Based on Order Fulfillment

```sql
SELECT 
    employee_id,
    COUNT(order_id) AS total_orders_handled,
    SUM(total_amount) AS total_sales_generated,
    AVG(DATEDIFF(s.delivery_date, o.order_date)) AS avg_delivery_time,
    RANK() OVER (ORDER BY SUM(total_amount) DESC) AS performance_rank
FROM 
    Orders o
JOIN 
    Shipments s ON o.id = s.order_id
GROUP BY 
    employee_id
ORDER BY 
    performance_rank;
```

- **Purpose**: Measure employee contribution to sales and efficiency.
- **Use Case**: Recognize top performers and identify areas for improvement.

---

### Key Takeaways

- **Window Functions**: Window functions like `SUM()`, `AVG()`, `RANK()`, and `ROW_NUMBER()` are powerful tools for performing complex analyses over data partitions without aggregating the result set.
- **Dynamic Analysis**: These queries allow you to analyze trends, rankings, and performance across various dimensions like time, geography, and product categories.
- **Performance**: Using window functions efficiently requires understanding their behavior and impact on query performance, especially for large datasets.
