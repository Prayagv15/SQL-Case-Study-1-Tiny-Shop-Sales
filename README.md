# SQL-Case-Study-1-Tiny-Shop-Sales

**Schema (MySQL v8.0)**

    CREATE TABLE customers (
        customer_id integer PRIMARY KEY,
        first_name varchar(100),
        last_name varchar(100),
        email varchar(100)
    );
    
    CREATE TABLE products (
        product_id integer PRIMARY KEY,
        product_name varchar(100),
        price decimal
    );
    
    CREATE TABLE orders (
        order_id integer PRIMARY KEY,
        customer_id integer,
        order_date date
    );
    
    CREATE TABLE order_items (
        order_id integer,
        product_id integer,
        quantity integer
    );
    
    INSERT INTO customers (customer_id, first_name, last_name, email) VALUES
    (1, 'John', 'Doe', 'johndoe@email.com'),
    (2, 'Jane', 'Smith', 'janesmith@email.com'),
    (3, 'Bob', 'Johnson', 'bobjohnson@email.com'),
    (4, 'Alice', 'Brown', 'alicebrown@email.com'),
    (5, 'Charlie', 'Davis', 'charliedavis@email.com'),
    (6, 'Eva', 'Fisher', 'evafisher@email.com'),
    (7, 'George', 'Harris', 'georgeharris@email.com'),
    (8, 'Ivy', 'Jones', 'ivyjones@email.com'),
    (9, 'Kevin', 'Miller', 'kevinmiller@email.com'),
    (10, 'Lily', 'Nelson', 'lilynelson@email.com'),
    (11, 'Oliver', 'Patterson', 'oliverpatterson@email.com'),
    (12, 'Quinn', 'Roberts', 'quinnroberts@email.com'),
    (13, 'Sophia', 'Thomas', 'sophiathomas@email.com');
    
    INSERT INTO products (product_id, product_name, price) VALUES
    (1, 'Product A', 10.00),
    (2, 'Product B', 15.00),
    (3, 'Product C', 20.00),
    (4, 'Product D', 25.00),
    (5, 'Product E', 30.00),
    (6, 'Product F', 35.00),
    (7, 'Product G', 40.00),
    (8, 'Product H', 45.00),
    (9, 'Product I', 50.00),
    (10, 'Product J', 55.00),
    (11, 'Product K', 60.00),
    (12, 'Product L', 65.00),
    (13, 'Product M', 70.00);
    
    INSERT INTO orders (order_id, customer_id, order_date) VALUES
    (1, 1, '2023-05-01'),
    (2, 2, '2023-05-02'),
    (3, 3, '2023-05-03'),
    (4, 1, '2023-05-04'),
    (5, 2, '2023-05-05'),
    (6, 3, '2023-05-06'),
    (7, 4, '2023-05-07'),
    (8, 5, '2023-05-08'),
    (9, 6, '2023-05-09'),
    (10, 7, '2023-05-10'),
    (11, 8, '2023-05-11'),
    (12, 9, '2023-05-12'),
    (13, 10, '2023-05-13'),
    (14, 11, '2023-05-14'),
    (15, 12, '2023-05-15'),
    (16, 13, '2023-05-16');
    
    INSERT INTO order_items (order_id, product_id, quantity) VALUES
    (1, 1, 2),
    (1, 2, 1),
    (2, 2, 1),
    (2, 3, 3),
    (3, 1, 1),
    (3, 3, 2),
    (4, 2, 4),
    (4, 3, 1),
    (5, 1, 1),
    (5, 3, 2),
    (6, 2, 3),
    (6, 1, 1),
    (7, 4, 1),
    (7, 5, 2),
    (8, 6, 3),
    (8, 7, 1),
    (9, 8, 2),
    (9, 9, 1),
    (10, 10, 3),
    (10, 11, 2),
    (11, 12, 1),
    (11, 13, 3),
    (12, 4, 2),
    (12, 5, 1),
    (13, 6, 3),
    (13, 7, 2),
    (14, 8, 1),
    (14, 9, 2),
    (15, 10, 3),
    (15, 11, 1),
    (16, 12, 2),
    (16, 13, 3);
    

---

**Query #1: Which product has the highest price? Only return a single row.**

    SELECT product_name,
    		(SELECT MAX(price) FROM products) AS `Max price`
    FROM products
    ORDER BY product_name DESC LIMIT 1;

| product_name | Max price |
| ------------ | --------- |
| Product M    | 70        |

---
**Query #2: Which customer has made the most orders?**

    WITH cte
    AS (
    	SELECT c.customer_id,CONCAT(c.first_name,' ',c.last_name) AS Customer,COUNT(DISTINCT o.order_id) AS order_count
    	FROM customers AS c
    	INNER JOIN orders AS o ON c.customer_id = o.customer_id
    	GROUP BY c.customer_id,c.first_name,c.last_name)
    SELECT customer_id,Customer,order_count
    FROM cte
    WHERE order_count = (
    		SELECT MAX(order_count)
    		FROM cte);

| customer_id | Customer    | order_count |
| ----------- | ----------- | ----------- |
| 1           | John Doe    | 2           |
| 2           | Jane Smith  | 2           |
| 3           | Bob Johnson | 2           |

---
**Query #3: What’s the total revenue per product?**

    SELECT DISTINCT p.product_name,SUM(p.price * oi.quantity) OVER (PARTITION BY p.product_name) AS Total_Revenue
    FROM products AS p
    INNER JOIN order_items AS oi ON p.product_id = oi.product_id;

| product_name | Total_Revenue |
| ------------ | ------------- |
| Product A    | 50            |
| Product B    | 135           |
| Product C    | 160           |
| Product D    | 75            |
| Product E    | 90            |
| Product F    | 210           |
| Product G    | 120           |
| Product H    | 135           |
| Product I    | 150           |
| Product J    | 330           |
| Product K    | 180           |
| Product L    | 195           |
| Product M    | 420           |

---
**Query #4: Find the day with the highest revenue.**

    WITH cte 
    AS  (SELECT p.price,oi.quantity,o.order_date, SUM(p.price * oi.quantity)
        	OVER (PARTITION BY o.order_date) AS total_price
        FROM products AS p
        INNER JOIN order_items AS oi
        	ON p.product_id = oi.product_id
        INNER JOIN orders AS o
        	ON oi.order_id = o.order_id
        GROUP BY  p.price, o.order_date, oi.quantity )
    SELECT order_date
    FROM cte
    WHERE total_price = 
        (SELECT MAX(total_price)
        FROM cte) limit 1;

| order_date |
| ---------- |
| 2023-05-16 |

---
**Query #5: Find the first order (by date) for each customer.**

    SELECT c.customer_id, concat(c.first_name,' ', c.last_name) AS customer, MIN(o.order_date) AS first_order_date
    FROM customers AS c
    INNER JOIN orders AS o
    	ON c.customer_id = o.customer_id
    GROUP BY  c.customer_id, c.first_name, c.last_name ;

| customer_id | customer         | first_order_date |
| ----------- | ---------------- | ---------------- |
| 1           | John Doe         | 2023-05-01       |
| 2           | Jane Smith       | 2023-05-02       |
| 3           | Bob Johnson      | 2023-05-03       |
| 4           | Alice Brown      | 2023-05-07       |
| 5           | Charlie Davis    | 2023-05-08       |
| 6           | Eva Fisher       | 2023-05-09       |
| 7           | George Harris    | 2023-05-10       |
| 8           | Ivy Jones        | 2023-05-11       |
| 9           | Kevin Miller     | 2023-05-12       |
| 10          | Lily Nelson      | 2023-05-13       |
| 11          | Oliver Patterson | 2023-05-14       |
| 12          | Quinn Roberts    | 2023-05-15       |
| 13          | Sophia Thomas    | 2023-05-16       |

---
**Query #6: Find the top 3 customers who have ordered the most distinct products**

    SELECT c.customer_id,CONCAT (first_name,' ',last_name) AS Customer,count(DISTINCT (oi.product_id)) AS 
    	distinct_product_order
    FROM customers AS c
    INNER JOIN orders AS o ON o.customer_id = c.customer_id
    INNER JOIN order_items AS oi ON o.order_id = oi.order_id
    GROUP BY c.customer_id,Customer
    ORDER BY distinct_product_order DESC limit 3;

| customer_id | Customer    | distinct_product_order |
| ----------- | ----------- | ---------------------- |
| 1           | John Doe    | 3                      |
| 2           | Jane Smith  | 3                      |
| 3           | Bob Johnson | 3                      |

---
**Query #7: Which product has been bought the least in terms of quantity?**

    WITH cte
    AS (
    	SELECT p.product_name,SUM(oi.quantity) AS quantity
    	FROM products AS p
    	INNER JOIN order_items AS oi ON p.product_id = oi.product_id
    	INNER JOIN orders AS o ON oi.order_id = o.order_id
    	GROUP BY p.product_name
    	)
    SELECT product_name,quantity
    FROM cte
    WHERE quantity = (SELECT MIN(quantity) FROM cte);

| product_name | quantity |
| ------------ | -------- |
| Product D    | 3        |
| Product E    | 3        |
| Product G    | 3        |
| Product H    | 3        |
| Product I    | 3        |
| Product K    | 3        |
| Product L    | 3        |

---
**Query #8: What is the median order total?**

    WITH cte  
    AS  (SELECT DISTINCT p.product_name,
    		 SUM(p.price * oi.quantity)
        	OVER (PARTITION BY p.product_name) AS Total_Revenue
        FROM products AS p
        INNER JOIN order_items AS oi
        	ON p.product_id = oi.product_id ), sorted_cte AS 
        (SELECT Total_Revenue,
    		 ROW_NUMBER()
        	OVER (ORDER BY Total_Revenue) AS row_index, 
            COUNT(*) OVER () AS total_rows
        FROM cte )
    SELECT ROUND(AVG(Total_Revenue),
    		 2) AS `Median Order Total`
    FROM sorted_cte
    WHERE row_index IN (FLOOR((total_rows + 1) / 2), CEIL((total_rows + 1) / 2));

| Median Order Total |
| ------------------ |
| 150.00             |

---
**Query #9: For each order, determine if it was ‘Expensive’ (total over 300), ‘Affordable’ (total over 100), or ‘Cheap’.**

    WITH cte
    AS (
    	SELECT DISTINCT p.product_name
    		,SUM(p.price * oi.quantity) OVER (PARTITION BY p.product_name) AS Total_Revenue
    	FROM products AS p
    	INNER JOIN order_items AS oi ON p.product_id = oi.product_id
    	)
    SELECT product_name,Total_Revenue,
    	CASE 
    		WHEN Total_Revenue > 300
    			THEN 'Expensive'
    		WHEN Total_Revenue BETWEEN 100
    				AND 300
    			THEN 'Affordable'
    		ELSE 'Cheap'
    		END AS Category
    FROM cte;

| product_name | Total_Revenue | Category   |
| ------------ | ------------- | ---------- |
| Product A    | 50            | Cheap      |
| Product B    | 135           | Affordable |
| Product C    | 160           | Affordable |
| Product D    | 75            | Cheap      |
| Product E    | 90            | Cheap      |
| Product F    | 210           | Affordable |
| Product G    | 120           | Affordable |
| Product H    | 135           | Affordable |
| Product I    | 150           | Affordable |
| Product J    | 330           | Expensive  |
| Product K    | 180           | Affordable |
| Product L    | 195           | Affordable |
| Product M    | 420           | Expensive  |

---
**Query #10: Find customers who have ordered the product with the highest price.**

    WITH cte
    AS (
    	SELECT c.customer_id,concat(first_name,' ',c.last_name) AS customer,p.price,p.product_name
    	FROM customers AS c
    	INNER JOIN orders AS o ON o.customer_id = c.customer_id
    	INNER JOIN order_items AS oi ON oi.order_id = o.order_id
    	INNER JOIN products AS p ON p.product_id = oi.product_id
    	GROUP BY c.customer_id,p.price,p.product_name,c.first_name,c.last_name
    	)
    SELECT customer_id,customer,price,product_name
    FROM cte
    WHERE price = (
    		SELECT MAX(price)
    		FROM cte);

| customer_id | customer      | price | product_name |
| ----------- | ------------- | ----- | ------------ |
| 8           | Ivy Jones     | 70    | Product M    |
| 13          | Sophia Thomas | 70    | Product M    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/5NT4w4rBa1cvFayg2CxUjr/144)
