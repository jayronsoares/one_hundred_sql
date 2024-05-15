**These questions cover a range of scenarios and SQL functionalities, helping to solidify SQL skills at an intermediate level.**

1. **Retrieve the names and email addresses of customers who have placed orders.**
   
   ```sql
   SELECT DISTINCT c.Name, c.Email
   FROM Customers c
   INNER JOIN Orders o ON c.CustomerID = o.CustomerID;
   ```

2. **List the total amount spent by each customer in descending order.**
   
   ```sql
   SELECT c.Name, SUM(o.TotalAmount) AS TotalSpent
   FROM Customers c
   INNER JOIN Orders o ON c.CustomerID = o.CustomerID
   GROUP BY c.Name
   ORDER BY TotalSpent DESC;
   ```

3. **Find the average quantity of products ordered in each category.**
   
   ```sql
   SELECT p.Category, AVG(od.Quantity) AS AvgQuantity
   FROM Products p
   INNER JOIN OrderDetails od ON p.ProductID = od.ProductID
   GROUP BY p.Category;
   ```

4. **Retrieve the details of orders made by a specific customer.**
   
   ```sql
   SELECT *
   FROM Orders
   WHERE CustomerID = <CustomerID>;
   ```

5. **List the top 5 best-selling products along with their total quantity sold.**
   
   ```sql
   SELECT p.Name, SUM(od.Quantity) AS TotalQuantitySold
   FROM Products p
   INNER JOIN OrderDetails od ON p.ProductID = od.ProductID
   GROUP BY p.Name
   ORDER BY TotalQuantitySold DESC
   LIMIT 5;
   ```

6. **Calculate the total revenue generated from sales in the last month.**
   
   ```sql
   SELECT SUM(Amount) AS TotalRevenue
   FROM Sales
   WHERE SaleDate >= DATE_SUB(CURRENT_DATE(), INTERVAL 1 MONTH);
   ```

7. **Find the number of orders placed by each customer in the year 2023.**
   
   ```sql
   SELECT c.Name, COUNT(o.OrderID) AS NumOrders
   FROM Customers c
   INNER JOIN Orders o ON c.CustomerID = o.CustomerID
   WHERE YEAR(o.OrderDate) = 2023
   GROUP BY c.Name;
   ```

8. **Identify customers who have not placed any orders.**
   
   ```sql
   SELECT c.Name, c.Email
   FROM Customers c
   LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
   WHERE o.OrderID IS NULL;
   ```

9. **List the products that are out of stock.**
   
   ```sql
   SELECT *
   FROM Products
   WHERE StockLevel = 0;
   ```

10. **Calculate the total amount of discounts given in each order.**
    
    ```sql
    SELECT OrderID, SUM(UnitPrice * Quantity * DiscountRate) AS TotalDiscount
    FROM OrderDetails
    GROUP BY OrderID;
    ```

11. **Find the average salary of employees in each department.**
    
    ```sql
    SELECT Department, AVG(Salary) AS AvgSalary
    FROM Employees
    GROUP BY Department;
    ```

12. **Retrieve the details of the highest and lowest priced products.**
    
    ```sql
    SELECT * FROM Products
    WHERE Price = (SELECT MAX(Price) FROM Products)
    UNION
    SELECT * FROM Products
    WHERE Price = (SELECT MIN(Price) FROM Products);
    ```

13. **List the customers who have made purchases more than once in a single day.**
    
    ```sql
    SELECT c.Name, o.OrderDate, COUNT(o.OrderID) AS NumOrders
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
    GROUP BY c.Name, o.OrderDate
    HAVING COUNT(o.OrderID) > 1;
    ```

14. **Identify the products that have been discounted more than 20%.**
    
    ```sql
    SELECT *
    FROM Products
    WHERE DiscountRate > 0.2;
    ```

15. **Retrieve the details of employees who earn a salary higher than their department average.**
    
    ```sql
    SELECT e.*
    FROM Employees e
    INNER JOIN (
        SELECT Department, AVG(Salary) AS AvgSalary
        FROM Employees
        GROUP BY Department
    ) dept_avg ON e.Department = dept_avg.Department
    WHERE e.Salary > dept_avg.AvgSalary;
    ```


**These advanced SQL questions cover complex scenarios and utilize advanced SQL techniques such as window functions, subqueries, and advanced aggregations.**


1. **Find the customers who have made at least two orders, where the time difference between their first and last order is more than 30 days.**

    ```sql
    SELECT c.Name, COUNT(o.OrderID) AS NumOrders
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
    GROUP BY c.CustomerID
    HAVING NumOrders >= 2 AND DATEDIFF(MAX(o.OrderDate), MIN(o.OrderDate)) > 30;
    ```

2. **Calculate the moving average of total sales amount over a window of 3 months.**

    ```sql
    SELECT SaleDate, 
           SUM(Amount) OVER (ORDER BY SaleDate ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS MovingAvgSales
    FROM Sales;
    ```

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

**There are 10 SQL questions with complex analysis scenarios to help master SQL:**

1. **Identify customers who have made purchases in all months of the last year, and calculate the average number of days between their purchases.**

    ```sql
    SELECT 
        c.CustomerID, 
        AVG(DATEDIFF(o1.OrderDate, o2.OrderDate)) AS AvgDaysBetweenPurchases
    FROM Customers c
    INNER JOIN Orders o1 ON c.CustomerID = o1.CustomerID
    INNER JOIN Orders o2 ON c.CustomerID = o2.CustomerID
    WHERE o1.OrderDate BETWEEN DATE_SUB(CURRENT_DATE(), INTERVAL 1 YEAR) AND CURRENT_DATE()
    AND o2.OrderDate BETWEEN DATE_SUB(CURRENT_DATE(), INTERVAL 1 YEAR) AND CURRENT_DATE()
    AND YEAR(o1.OrderDate) = YEAR(o2.OrderDate)
    AND MONTH(o1.OrderDate) != MONTH(o2.OrderDate)
    GROUP BY c.CustomerID
    HAVING COUNT(DISTINCT MONTH(o1.OrderDate)) = 12;
    ```

2. **Calculate the total sales amount for each day of the week, considering only weekdays.**

    ```sql
    SELECT 
        DAYOFWEEK(SaleDate) AS DayOfWeek,
        SUM(Amount) AS TotalSales
    FROM Sales
    WHERE DAYOFWEEK(SaleDate) BETWEEN 2 AND 6
    GROUP BY DayOfWeek;
    ```

3. **Find the customers who have made purchases on consecutive days, and calculate the average amount of their purchases.**

    ```sql
    SELECT 
        c.CustomerID, 
        AVG(o.TotalAmount) AS AvgPurchaseAmount
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
    WHERE EXISTS (
        SELECT 
            1
        FROM Orders o2
        WHERE o2.CustomerID = c.CustomerID
        AND DATEDIFF(o.OrderDate, o2.OrderDate) = 1
    )
    GROUP BY c.CustomerID;
    ```

4. **Identify the products that have been ordered in each quarter of the last year, and calculate the total quantity sold for each product in each quarter.**

    ```sql
    SELECT 
        p.ProductID, 
        p.Name,
        YEAR(o.OrderDate) AS Year,
        QUARTER(o.OrderDate) AS Quarter,
        SUM(od.Quantity) AS TotalQuantitySold
    FROM Products p
    INNER JOIN OrderDetails od ON p.ProductID = od.ProductID
    INNER JOIN Orders o ON od.OrderID = o.OrderID
    WHERE o.OrderDate BETWEEN DATE_SUB(CURRENT_DATE(), INTERVAL 1 YEAR) AND CURRENT_DATE()
    GROUP BY p.ProductID, Year, Quarter;
    ```

5. **Retrieve the top 3 most loyal customers, based on the number of repeat purchases they have made within a month of their previous purchase.**

    ```sql
    SELECT 
        c.CustomerID,
        c.Name,
        COUNT(DISTINCT o.OrderID) AS RepeatPurchases
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
    WHERE EXISTS (
        SELECT 
            1
        FROM Orders o2
        WHERE o2.CustomerID = c.CustomerID
        AND o2.OrderDate BETWEEN DATE_SUB(o.OrderDate, INTERVAL 1 MONTH) AND o.OrderDate
        AND o2.OrderID != o.OrderID
    )
    GROUP BY c.CustomerID, c.Name
    ORDER BY RepeatPurchases DESC
    LIMIT 3;
    ```

6. **Calculate the percentage of total sales contributed by each product category, excluding categories with less than 5% contribution.**

    ```sql
    SELECT 
        Category,
        SUM(Amount) / (SELECT SUM(Amount) FROM Sales) * 100 AS ContributionPercentage
    FROM Products p
    INNER JOIN Sales s ON p.ProductID = s.ProductID
    GROUP BY Category
    HAVING ContributionPercentage >= 5;
    ```

7. **Identify customers who have made purchases every month in the last year, and calculate the average amount they spent in each month.**

    ```sql
    SELECT 
        c.CustomerID,
        MONTH(o.OrderDate) AS Month,
        AVG(o.TotalAmount) AS AvgAmountSpent
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
    WHERE YEAR(o.OrderDate) = YEAR(CURRENT_DATE())
    GROUP BY c.CustomerID, Month
    HAVING COUNT(DISTINCT MONTH(o.OrderDate)) = 12;
    ```

8. **Find the products that have been ordered together at least 3 times within a month, and calculate the average quantity ordered together.**

    ```sql
    SELECT 
        od1.ProductID AS Product1,
        od2.ProductID AS Product2,
        AVG(od1.Quantity + od2.Quantity) AS AvgQuantityOrderedTogether
    FROM OrderDetails od1
    INNER JOIN OrderDetails od2 ON od1.OrderID = od2.OrderID AND od1.ProductID < od2.ProductID
    INNER JOIN Orders o ON od1.OrderID = o.OrderID
    WHERE o.OrderDate BETWEEN DATE_SUB(CURRENT_DATE(), INTERVAL 1 MONTH) AND CURRENT_DATE()
    GROUP BY Product1, Product2
    HAVING COUNT(*) >= 3;
    ```

9. **Retrieve the top 3 most profitable customers in terms of total purchases, considering only weekdays.**

    ```sql
    SELECT 
        c.CustomerID,
        c.Name,
        SUM(s.Amount) AS TotalPurchaseAmount
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
    INNER JOIN Sales s ON o.OrderID = s.OrderID
    WHERE DAYOFWEEK(o.OrderDate) BETWEEN 2 AND 6
    GROUP BY c.CustomerID, c.Name
    ORDER BY TotalPurchaseAmount DESC
    LIMIT 3;
    ```

10. **Calculate the average time taken between the first purchase and the second purchase for each customer.**

    ```sql
    SELECT 
        o1.CustomerID,
        AVG(DATEDIFF(o2.OrderDate, o1.OrderDate)) AS AvgDaysBetweenFirstAndSecondPurchase
    FROM Orders o1
    INNER JOIN Orders o2 ON o1.CustomerID = o2.CustomerID AND o2.OrderDate > o1.OrderDate
    GROUP BY o1.CustomerID;
    ```
