Sure, here are 15 intermediate-level SQL questions along with their expected answers:

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

These questions cover a range of scenarios and SQL functionalities, helping to solidify SQL skills at an intermediate level.
