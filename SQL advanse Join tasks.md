# SQL advanse Join tasks

---

## Task 1: Online Bookstore Sales Contest:

![alt text](Screenshot%202024-07-18%20094702.png)

## Step 1: Create Table and Insert Data:

### **Sample Data**:

#### Book Table:

```sql
CREATE TABLE Books (
    book_id INT PRIMARY KEY,
    title VARCHAR(100)
);

INSERT INTO Books (book_id, title) VALUES
(101, 'Book A'),
(202, 'Book B'),
(303, 'Book C');
```

### Sales Table:

```sql

CREATE TABLE Sales (
    sale_date DATE,
    sale_id INT PRIMARY KEY,
    book_id INT,
    quantity INT,
    FOREIGN KEY (book_id) REFERENCES Books(book_id)
);

INSERT INTO Sales (sale_date, sale_id, book_id, quantity) VALUES
('2024-04-01', 1, 101, 10),
('2024-04-01', 2, 202, 5),
('2024-04-02', 3, 101, 8),
('2024-04-03', 4, 101, 7),
('2024-04-04', 5, 101, 6),
('2024-04-05', 6, 303, 9),
('2024-04-06', 7, 101, 5);
```

---

### 1. List all sales by date, including `sale_date`, `sale_id`, `book_id`, and `quantity`.

```sql
SELECT sale_date, sale_id, book_id, quantity
FROM Sales
ORDER BY sale_date;
```

---

### 2. Count the number of unique books sold each day.

```sql
SELECT sale_date, COUNT(DISTINCT book_id) AS unique_books_sold
FROM Sales
Group By sale_date;
```

---

### 3. Find the `book_id` and `title` of the book with the maximum number of sales each day.

```sql
WITH DailySales AS (
    SELECT sale_date, book_id, SUM(quantity) AS total_sales
    FROM Sales
    GROUP BY sale_date, book_id
), MaxSales AS (
    SELECT sale_date, MAX(total_sales) AS max_sales
    FROM DailySales
    GROUP BY sale_date
)
SELECT ds.sale_date, ds.book_id, b.title
FROM DailySales ds
JOIN MaxSales ms ON ds.sale_date = ms.sale_date AND ds.total_sales = ms.max_sales
JOIN Books b ON ds.book_id = b.book_id
ORDER BY ds.sale_date;
```

---

### 4. If multiple books have the same number of maximum sales, select the one with the lowest `book_id`.

```sql
WITH DailySales AS (
    SELECT sale_date, book_id, SUM(quantity) AS total_sales
    FROM Sales
    GROUP BY sale_date, book_id
), MaxSales AS (
    SELECT sale_date, MAX(total_sales) AS max_sales
    FROM DailySales
    GROUP BY sale_date
), RankedSales AS (
    SELECT ds.sale_date, ds.book_id, ds.total_sales,
           ROW_NUMBER() OVER (PARTITION BY ds.sale_date ORDER BY ds.total_sales DESC, ds.book_id) AS rank
    FROM DailySales ds
    JOIN MaxSales ms ON ds.sale_date = ms.sale_date AND ds.total_sales = ms.max_sales
)
SELECT rs.sale_date, rs.book_id, b.title
FROM RankedSales rs
JOIN Books b ON rs.book_id = b.book_id
WHERE rs.rank = 1
ORDER BY rs.sale_date;
```

---

### 5. Create a summary of the total sales across all days, including the count of total ales and the number of unique books sold.

```sql
SELECT COUNT(*) AS total_sales, COUNT(DISTINCT book_id) AS unique_books_sold
FROM Sales;
```

---

### 6. Calculate the daily sales rate as the ratio of books sold to the total number of books available.

```sql
SELECT sale_date, SUM(quantity) / (SELECT COUNT(*) FROM Books) AS daily_sales_rate
FROM Sales
GROUP BY sale_date;
```

---

### 7. Analyze the trend of sales over the contest period to identify peaks and dips.

```sql
SELECT sale_date, SUM(quantity) AS total_sales
FROM Sales
ORDER BY sale_date;
```

---

### 8. Identify books that made sales on all days of the contest.

```sql
WITH DailySales AS (
    SELECT sale_date, book_id
    FROM Sales
    GROUP BY sale_date, book_id
)
SELECT book_id
FROM DailySales
GROUP BY book_id
HAVING COUNT(sale_date) = (SELECT COUNT(DISTINCT sale_date) FROM Sales);
```

---

### 9. Determine the number of days each book was sold (made at least one sale).

```sql
SELECT book_id, COUNT(DISTINCT sale_date) AS days_sold
FROM Sales
GROUP BY book_id;
```

---

### 10. Count the number of sales made for each book each day.

```sql
SELECT sale_date, book_id, COUNT(*) AS sales_count
FROM Sales
GROUP BY sale_date, book_id;
```

---

### 11. Identify the highest quantity sold for any book each day.

```sql
SELECT sale_date, MAX(quantity) AS max_quantity_sold
FROM Sales
GROUP BY sale_date;
```

---

### 12. List books that had multiple sales on any given day.

```sql
SELECT sale_date, book_id, COUNT(*) AS sales_count
FROM Sales
GROUP BY sale_date, book_id
HAVING COUNT(*) > 1;
```

---

### 13. Find books with sales quantities below a certain threshold (e.g., 5 copies).

```sql
SELECT book_id, title
FROM Books
WHERE book_id IN (
    SELECT book_id
    FROM Sales
    GROUP BY book_id
    HAVING SUM(quantity) < 5
);
```

---

### 14. Provide a final summary of the contest, including total sales, unique books sold, and highest selling book.

```sql
WITH TotalSales AS (
    SELECT COUNT(*) AS total_sales, COUNT(DISTINCT book_id) AS unique_books_sold
    FROM Sales
),
TopBook AS (
    SELECT book_id, SUM(quantity) AS total_quantity
    FROM Sales
    GROUP BY book_id
    ORDER BY total_quantity DESC
    LIMIT 1
)
SELECT t.total_sales, t.unique_books_sold, b.book_id, b.title, tb.total_quantity
FROM TotalSales t
JOIN TopBook tb ON 1=1
JOIN Books b ON tb.book_id = b.book_id;
```

---

---

## Task 2. SmartBuy Database

### Step 1. Create Tables and Insert Sample Data

```sql
-- Table 1: customers
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    status VARCHAR(10)
);

INSERT INTO customers (customer_id, customer_name, status) VALUES
(1, 'Alice', 'active'),
(2, 'Bob', 'inactive'),
(3, 'Charlie', 'active'),
(4, 'Diana', 'active'),
(5, 'Eve', 'inactive');

-- Table 2: orders
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    shipping_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

INSERT INTO orders (order_id, customer_id, order_date, shipping_date) VALUES
(1, 1, '2023-01-01', '2023-01-05'),
(2, 2, '2023-01-02', NULL),
(3, 3, '2023-01-03', '2023-01-06'),
(4, 4, '2023-01-04', NULL),
(5, 1, '2023-01-05', '2023-01-07');

-- Table 3: products
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100)
);

INSERT INTO products (product_id, product_name) VALUES
(1, 'Laptop'),
(2, 'Smartphone'),
(3, 'Tablet'),
(4, 'Monitor'),
(5, 'Keyboard');

-- Table 4: order_details
CREATE TABLE order_details (
    order_detail_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10, 2),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

INSERT INTO order_details (order_detail_id, order_id, product_id, quantity, price) VALUES
(1, 1, 1, 1, 1000.00),
(2, 1, 2, 2, 500.00),
(3, 2, 3, 1, 300.00),
(4, 3, 1, 1, 1000.00),
(5, 3, 4, 2, 150.00),
(6, 4, 5, 3, 50.00),
(7, 5, 2, 1, 500.00);

-- Table 5: employees
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    manager_id INT,
    department_id INT,
    FOREIGN KEY (manager_id) REFERENCES employees(employee_id),
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

INSERT INTO employees (employee_id, employee_name, manager_id, department_id) VALUES
(1, 'John', NULL, 1),
(2, 'Sara', 1, 1),
(3, 'Mike', 1, 2),
(4, 'Kate', 2, 1),
(5, 'Tom', 3, 2);

-- Table 6: departments
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100)
);

INSERT INTO departments (department_id, department_name) VALUES
(1, 'Sales'),
(2, 'Engineering'),
(3, 'HR');

-- Table 7: projects
CREATE TABLE projects (
    project_id INT PRIMARY KEY,
    project_name VARCHAR(100)
);

INSERT INTO projects (project_id, project_name) VALUES
(1, 'Project A'),
(2, 'Project B'),
(3, 'Project C');

-- Table 8: employee_projects
CREATE TABLE employee_projects (
    employee_id INT,
    project_id INT,
    PRIMARY KEY (employee_id, project_id),
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id),
    FOREIGN KEY (project_id) REFERENCES projects(project_id)
);

INSERT INTO employee_projects (employee_id, project_id) VALUES
(1, 1),
(2, 1),
(3, 2),
(4, 3),
(5, 2);
```

---

### Task 1: Retrieve a list of all customers along with their corresponding orders.

```sql
SELECT c.customer_id, c.customer_name, o.order_id, o.order_date, o.shipping_date
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```

---

### Task 2: Get a list of all products and their associated order details, including products that haven't been ordered.

```sql
SELECT p.product_id, p.product_name, od.order_detail_id, od.order_id, od.quantity, od.price
FROM products p
LEFT JOIN order_details od ON p.product_id = od.product_id;
```

---

### Task 3: Find all employees and their associated department names. Include departments with no employees.

```sql
SELECT e.employee_id, e.employee_name, d.department_id, d.department_name
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id;
```

---

### Task 4: Calculate the total sales amount for each product.

```sql
SELECT p.product_id, p.product_name, SUM(od.quantity * od.price) AS total_sales
FROM products p
JOIN order_details od ON p.product_id = od.product_id
GROUP BY p.product_id, p.product_name;
```

---

### Task 5: Retrieve a list of all orders with customer names, product names, and order quantities.

```sql
SELECT o.order_id, c.customer_name, p.product_name, od.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_details od ON o.order_id = od.order_id
JOIN products p ON od.product_id = p.product_id;
```

---

### Task 6: Find customers who have placed more than five orders.

```sql
SELECT c.customer_id, c.customer_name, COUNT(o.order_id) AS order_count
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
HAVING COUNT(o.order_id) > 5;
```

---

### Task 7: Get a list of all employees and the projects they are working on, including employees not assigned to any project and projects with no assigned employees.

```sql
SELECT e.employee_id, e.employee_name, p.project_id, p.project_name
FROM employees e
FULL OUTER JOIN employee_projects ep ON e.employee_id = ep.employee_id
FULL OUTER JOIN projects p ON ep.project_id = p.project_id;
```

---

### Task 8: Create a list of all active and inactive customers along with their orders, separating them using a status column.

```sql
SELECT c.customer_id, c.customer_name, c.status, o.order_id, o.order_date, o.shipping_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.status = 'active'
UNION
SELECT c.customer_id, c.customer_name, c.status, o.order_id, o.order_date, o.shipping_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.status = 'inactive';
```

---

### Task 9: Retrieve a list of all orders with their status ('Shipped', 'Pending', 'Canceled') based on the order date and shipping date.

```sql
SELECT o.order_id, o.order_date, o.shipping_date,
       CASE
           WHEN o.shipping_date IS NOT NULL THEN 'Shipped'
           WHEN o.order_date IS NOT NULL AND o.shipping_date IS NULL THEN 'Pending'
           ELSE 'Canceled'
       END AS status
FROM orders o;
```

---

---
