### These questions cover a range of scenarios and SQL functionalities, helping to solidify your SQL skills at an intermediate level.

1. **Retrieve the names and email addresses of customers who have placed orders.**

2. **List the total amount spent by each customer in descending order.**
   
3. **Find the average quantity of products ordered in each category.**

4. **Retrieve the details of orders made by a specific customer.**
  
5. **List the top 5 best-selling products along with their total quantity sold.**

6. **Calculate the total revenue generated from sales in the last month.**

7. **Find the number of orders placed by each customer in the year 2023.**

8. **Identify customers who have not placed any orders.**
  
9. **List the products that are out of stock.**
 
10. **Calculate the total amount of discounts given in each order.**

11. **Find the average salary of employees in each department.**  

12. **Retrieve the details of the highest and lowest priced products.**
   
13. **List the customers who have made purchases more than once in a single day.**
    
14. **Identify the products that have been discounted more than 20%.**
   
15. **Retrieve the details of employees who earn a salary higher than their department average.**
Certainly! Here are 15 advanced SQL questions along with their answers:

16. **Find the customers who have made at least two orders, where the time difference between their first and last order is more than 30 days.**

17. **Calculate the moving average of total sales amount over a window of 3 months.**
18. 
3. **Identify customers who have spent more than the average amount spent by all customers.**

    ```sql
    SELECT c.Name, SUM(o.TotalAmount) AS TotalSpent
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
    GROUP BY c.Name
    HAVING TotalSpent > (SELECT AVG(TotalAmount) FROM Orders);
    ```

4. **Retrieve the top 3 categories that contribute the most to total revenue.**

    ```sql
    SELECT p.Category, SUM(s.Amount) AS TotalRevenue
    FROM Products p
    INNER JOIN Sales s ON p.ProductID = s.ProductID
    GROUP BY p.Category
    ORDER BY TotalRevenue DESC
    LIMIT 3;
    ```

5. **List the customers who have not made any purchases in the last 6 months.**

    ```sql
    SELECT c.Name, MAX(o.OrderDate) AS LastPurchaseDate
    FROM Customers c
    LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
    GROUP BY c.CustomerID
    HAVING LastPurchaseDate IS NULL OR LastPurchaseDate < DATE_SUB(CURRENT_DATE(), INTERVAL 6 MONTH);
    ```

6. **Calculate the percentage contribution of each product to total sales.**

    ```sql
    SELECT p.Name, 
           SUM(s.Amount) / (SELECT SUM(Amount) FROM Sales) * 100 AS ContributionPercentage
    FROM Products p
    INNER JOIN Sales s ON p.ProductID = s.ProductID
    GROUP BY p.Name;
    ```

7. **Find the customers who have made purchases in all categories.**

    ```sql
    SELECT c.Name
    FROM Customers c
    INNER JOIN (
        SELECT DISTINCT CustomerID, Category
        FROM Orders o
        INNER JOIN OrderDetails od ON o.OrderID = od.OrderID
        INNER JOIN Products p ON od.ProductID = p.ProductID
    ) cust_categories ON c.CustomerID = cust_categories.CustomerID
    GROUP BY c.Name
    HAVING COUNT(DISTINCT cust_categories.Category) = (SELECT COUNT(DISTINCT Category) FROM Products);
    ```

8. **Retrieve the top 5 customers who have made the highest number of orders in the last year.**

    ```sql
    SELECT c.Name, COUNT(o.OrderID) AS NumOrders
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
    WHERE o.OrderDate >= DATE_SUB(CURRENT_DATE(), INTERVAL 1 YEAR)
    GROUP BY c.Name
    ORDER BY NumOrders DESC
    LIMIT 5;
    ```

9. **List the employees who have managed departments with an average salary higher than $50000.**

    ```sql
    SELECT e.Name
    FROM Employees e
    INNER JOIN (
        SELECT ManagerName, AVG(Salary) AS AvgSalary
        FROM Employees
        GROUP BY ManagerName
    ) dept_avg ON e.Name = dept_avg.ManagerName
    WHERE dept_avg.AvgSalary > 50000;
    ```

10. **Calculate the 90th percentile of order amounts.**

    ```sql
    SELECT PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY TotalAmount) AS 90thPercentile
    FROM Orders;
    ```

11. **Identify the products that have been ordered by at least 3 different customers.**

    ```sql
    SELECT p.Name
    FROM Products p
    INNER JOIN OrderDetails od ON p.ProductID = od.ProductID
    INNER JOIN Orders o ON od.OrderID = o.OrderID
    GROUP BY p.Name
    HAVING COUNT(DISTINCT o.CustomerID) >= 3;
    ```

12. **Find the total revenue generated for each month in the last year.**

    ```sql
    SELECT DATE_FORMAT(SaleDate, '%Y-%m') AS Month, SUM(Amount) AS TotalRevenue
    FROM Sales
    WHERE SaleDate >= DATE_SUB(CURRENT_DATE(), INTERVAL 1 YEAR)
    GROUP BY Month;
    ```

13. **Retrieve the top 3 customers who have made the highest total purchases in terms of quantity.**

    ```sql
    SELECT c.Name, SUM(od.Quantity) AS TotalQuantity
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
    INNER JOIN OrderDetails od ON o.OrderID = od.OrderID
    GROUP BY c.Name
    ORDER BY TotalQuantity DESC
    LIMIT 3;
    ```

14. **List the customers who have made purchases of at least $1000 in each of the last 3 months.**

    ```sql
    SELECT c.Name, 
           YEAR(o.OrderDate) AS Year, 
           MONTH(o.OrderDate) AS Month, 
           SUM(o.TotalAmount) AS MonthlyTotal
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
    WHERE o.OrderDate >= DATE_SUB(CURRENT_DATE(), INTERVAL 3 MONTH)
    GROUP BY c.Name, Year, Month
    HAVING MonthlyTotal >= 1000;
    ```

15. **Identify the customers who have placed orders for all products in a specific category.**

    ```sql
    SELECT c.Name
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
    INNER JOIN OrderDetails od ON o.OrderID = od.OrderID
    INNER JOIN Products p ON od.ProductID = p.ProductID
    WHERE p.Category = 'specific_category'
    GROUP BY c.Name
    HAVING COUNT(DISTINCT p.ProductID) = (SELECT COUNT(*) FROM Products WHERE Category = 'specific_category');
    ```

These advanced SQL questions cover complex scenarios and utilize advanced SQL techniques such as window functions, subqueries, and advanced aggregations.
