# PostgreSQL

[tutorials](https://www.postgresqltutorial.com)

[TW doc](https://docs.postgresql.tw)

## Environment(For MAC)

```sh
brew install postgresql
brew services start postgresql
brew services list
initdb --locale=C -E UTF-8 /usr/local/var/postgres

---
To migrate existing data from a previous major version of PostgreSQL run:
  brew postgresql-upgrade-database

This formula has created a default database cluster with:
  initdb --locale=C -E UTF-8 /usr/local/var/postgres
For more details, read:
  https://www.postgresql.org/docs/13/app-initdb.html

To have launchd start postgresql now and restart at login:
  brew services start postgresql
Or, if you don't want/need a background service you can just run:
  pg_ctl -D /usr/local/var/postgres start

```

```sh
## Download smaple db
curl -O https://sp.postgresqltutorial.com/wp-content/uploads/2019/05/dvdrental.zip
unzip dvdrental.zip

## createdb <DATABASE>
createdb dvdrental

## restore db
pg_restore --dbname=dvdrental --verbose dvdrental.tar
rm dvdrental.*

## Log into database
#### psql -d <DATABASE>
#### psql postgres://<USER>:<PASSWORD>@<HOST>:<PORT>/<DATABASE>(Heroku URI)
psql -d dvdrental
\c dvdrental
select count(*) from film;

## 常用指令
#  `\?` = help
#  `\l` = list DB
#  `\dt` = list table
#  `\c DB` = connect DB
#  `\q` = quit
```

## Section 1. Querying Data

### SELECT

```sql
-- basic
SELECT first_name FROM customer;

-- multiple coloumns
SELECT
   first_name,
   last_name,
   email
FROM
   customer;
   
-- all coloumns(會影響效能)
SELECT * FROM customer;

-- concatenate coloumns
SELECT 
   first_name || ' ' || last_name,
   email
FROM 
   customer;
   
 -- expression
 SELECT 5 * 3;
```

### Column Alias

```sql
-- Alias
SELECT
    first_name || ' ' || last_name AS "full_name"
FROM
    customer;
 
SELECT 5 * 3 AS "5 * 3";
```

### ORDER BY

```sql
-- ORDER BY sort_expresssion [ASC | DESC] [NULLS FIRST | NULLS LAST]

-- Order by coloumns ASC
SELECT
	first_name,
	last_name
FROM
	customer
ORDER BY
	first_name ASC;
	
-- Order by coloumns DESC
SELECT
       first_name,
       last_name
FROM
       customer
ORDER BY
       last_name DESC;
       
-- sort rows by multiple coloumns
SELECT
	first_name,
	last_name
FROM
	customer
ORDER BY
	first_name ASC,
	last_name DESC;
	
-- sort rows by LENGTH()
SELECT 
	first_name,
	LENGTH(first_name) len
FROM
	customer
ORDER BY 
	len DESC;
	
SELECT 1,2,3, NUll;
```

### SELECT DISTINCT

搜尋時移除重複的 rows

```sql
CREATE TABLE distinct_demo (
	id serial NOT NULL PRIMARY KEY,
	bcolor VARCHAR,
	fcolor VARCHAR
);

INSERT INTO distinct_demo (bcolor, fcolor)
VALUES
	('red', 'red'),
	('red', 'red'),
	('red', NULL),
	(NULL, 'red'),
	('red', 'green'),
	('red', 'blue'),
	('green', 'red'),
	('green', 'blue'),
	('green', 'green'),
	('blue', 'red'),
	('blue', 'green'),
	('blue', 'blue');
```

```sql
-- bcolor - 4 rows
SELECT
	DISTINCT bcolor
FROM
	distinct_demo
ORDER BY
	bcolor;

-- bcolor & fcolor - 11 rows
SELECT
	DISTINCT bcolor, fcolor
FROM
	distinct_demo
ORDER BY
	bcolor,
	fcolor;
	
-- DISTINCT ON (bcolor)
SELECT
	DISTINCT ON (bcolor) 
	bcolor,fcolor
FROM
	distinct_demo 
ORDER BY
	bcolor,
	fcolor;
```

## Section 2. Filtering Data

### WHERE

-   =, >, <, >=, <=, <> or !=, AND, OR, IN, BETWEEN, LIKE, ISNULL, NOT

```sql
-- equal
SELECT
	last_name,
	first_name
FROM
	customer
WHERE
	first_name = 'Jamie';
	
-- and
SELECT
	last_name,
	first_name
FROM
	customer
WHERE
	first_name = 'Jamie' AND 
        last_name = 'Rice';
      
-- or
SELECT
	first_name,
	last_name
FROM
	customer
WHERE
	last_name = 'Rodriguez' OR 
	first_name = 'Adam';
	
-- in
SELECT
	first_name,
	last_name
FROM
	customer
WHERE 
	first_name IN ('Ann','Anne','Annie');
	
-- like
SELECT
	first_name,
	last_name
FROM
	customer
WHERE 
	first_name LIKE 'Ann%'
	
-- between
SELECT
	first_name,
	LENGTH(first_name) name_length
FROM
	customer
WHERE 
	first_name LIKE 'A%' AND
	LENGTH(first_name) BETWEEN 3 AND 5
ORDER BY
	name_length;
	
-- not equal
SELECT 
	first_name, 
	last_name
FROM 
	customer 
WHERE 
	first_name LIKE 'Bra%' AND 
	last_name <> 'Motley';
```

### LIMIT

```sql
-- limit 5
SELECT
	film_id,
	title,
	release_year
FROM
	film
ORDER BY
	film_id
LIMIT 5;

-- limit with offset
SELECT
	film_id,
	title,
	release_year
FROM
	film
ORDER BY
	film_id
LIMIT 4 OFFSET 3;
```

### FETCH

因為 LIMIT 不是 SQL standard，所以 postgreSQL 還提供功能一樣的 FETCH 

```sql
OFFSET start { ROW | ROWS }
FETCH { FIRST | NEXT } [ row_count ] { ROW | ROWS } ONLY

-- row=rows, first=next: 可互用
-- start 為正整數，預設為0
-- row_count 為抓幾個 row
```

```sql
-- fetch one row
SELECT
    film_id,
    title
FROM
    film
ORDER BY
    title 
FETCH FIRST ROW ONLY;

-- fetch five rows
SELECT
    film_id,
    title
FROM
    film
ORDER BY
    title 
FETCH FIRST 5 ROW ONLY;

-- fetch five rows with offset five rows
SELECT
    film_id,
    title
FROM
    film
ORDER BY
    title 
OFFSET 5 ROWS 
FETCH FIRST 5 ROW ONLY;
```

### IN

```sql
-- in (PostgreSQL使用IN比OR執行相同查詢快得多)
SELECT customer_id,
	rental_id,
	return_date
FROM
	rental
WHERE
	customer_id IN (1, 2)
ORDER BY
	return_date DESC;
	
-- not in
SELECT
	customer_id,
	rental_id,
	return_date
FROM
	rental
WHERE
	customer_id NOT IN (1, 2);
	
-- subquery

SELECT
	customer_id,
	first_name,
	last_name
FROM
	customer
WHERE
	customer_id IN (
		SELECT customer_id
		FROM rental
		WHERE CAST (return_date AS DATE) = '2005-05-27'
	)
ORDER BY customer_id;
```

### BETWEEN

```sql
-- between( >= low and <= high)
SELECT
	customer_id,
	payment_id,
	amount
FROM
	payment
WHERE
	amount BETWEEN 8 AND 9;
	
-- between date iso8661(YYYY-MM-DD)
SELECT
	customer_id,
	payment_id,
	amount,
 payment_date
FROM
	payment
WHERE
	payment_date BETWEEN '2007-02-07' AND '2007-02-15';
```

### LIKE

-   `%`: 比對任意字串(0~n)
-   `_`: 比對任意字元(1)
-   `~~`: like
-   `~~*`: ilike
-   `!~~`: not like
-   `!~~*`: not ilike

```sql
-- like
SELECT
	'foo' LIKE 'foo', -- true
	'foo' LIKE 'f%', -- true
	'foo' LIKE '_o_', -- true
	'bar' LIKE 'b_'; -- false
	
-- %er%
SELECT
	first_name,
        last_name
FROM
	customer
WHERE
	first_name LIKE '%er%'
ORDER BY 
        first_name;

-- _her%
SELECT
	first_name,
	last_name
FROM
	customer
WHERE
	first_name LIKE '_her%'
ORDER BY 
        first_name;
        
-- not like
SELECT
	first_name,
	last_name
FROM
	customer
WHERE
	first_name NOT LIKE 'Jen%'
ORDER BY 
        first_name
        
-- ilike(case-insensitively)
SELECT
	first_name,
	last_name
FROM
	customer
WHERE
	first_name ILIKE 'BAR%';
```

### IS NULL

在DB世界中，NULL意味著缺少資訊或不適用。NULL不是一個值，因此，你不能將它與任何其他值，進行比較。將NULL與一個值進行比較的結果總是NULL，意味著一個未知的結果。

-   NULL = 1, return NULL
-   NULL = NULL, return NULL

```sql
CREATE TABLE contacts(
    id INT GENERATED BY DEFAULT AS IDENTITY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL,
    phone VARCHAR(15),
    PRIMARY KEY (id)
);

INSERT INTO contacts(first_name, last_name, email, phone)
VALUES ('John','Doe','john.doe@example.com',NULL),
    ('Lily','Bush','lily.bush@example.com','(408-234-2764)');
```

```sql
-- = null (0 row)
SELECT
    id,
    first_name,
    last_name,
    email,
    phone
FROM
    contacts
WHERE
    phone = NULL;
    
-- is null (1 row)
SELECT
    id,
    first_name,
    last_name,
    email,
    phone
FROM
    contacts
WHERE
    phone IS NULL;
    
-- is not null (1 row)
SELECT
    id,
    first_name,
    last_name,
    email,
    phone
FROM
    contacts
WHERE
    phone IS NOT NULL;
```

##  Section 3. Joining Multiple Tables

PostgreSQL join 用於根據相關表之間的共用的 coloumn 的值來組合一個或多個表的 coloumn。共用的 coloumn 通常是第一個表的  primary key 和第二個表的 foreign key。

### Joins

PostgreSQL supports 

-   inner join
-   left join
-   right join
-   full outer join
-   cross join
-   natural join
-   self-join

![img](https://sp.postgresqltutorial.com/wp-content/uploads/2018/12/PostgreSQL-Joins.png)



### Table Aliases

當兩個table有同個coloumn要用別名`**table_name**.column_name`分辨他們

```sql
-- inner join with table alias
SELECT
	c.customer_id,
	first_name,
	amount,
	payment_date
FROM
	customer c
INNER JOIN payment p 
    ON p.customer_id = c.customer_id
ORDER BY 
   payment_date DESC;
   
-- self join with table alias
SELECT
    e.first_name employee,
    m .first_name manager
FROM
    employee e
INNER JOIN employee m 
    ON m.employee_id = e.manager_id
ORDER BY manager;
```

### INNER JOIN

左右表都有才會列出來。

```sql
-- ON pka = fka
SELECT
	c.customer_id,
	first_name,
	last_name,
	email,
	amount,
	payment_date
FROM
	customer c
INNER JOIN payment p 
    ON p.customer_id = c.customer_id
WHERE
    c.customer_id = 2;
    
-- USING same coloumn
SELECT
	customer_id,
	first_name,
	last_name,
	amount,
	payment_date
FROM
	customer
INNER JOIN payment USING(customer_id)
ORDER BY payment_date;

-- inner joint 3 tables
SELECT
	c.customer_id,
	c.first_name customer_first_name,
	c.last_name customer_last_name,
	s.first_name staff_first_name,
	s.last_name staff_last_name,
	amount,
	payment_date
FROM
	customer c
INNER JOIN payment p 
    ON p.customer_id = c.customer_id
INNER JOIN staff s 
    ON p.staff_id = s.staff_id
ORDER BY payment_date;
```

### LEFT JOIN

先依左表資料為主，右表欄位沒有對到的話填NULL

```sql
-- 用 flim table LEFT JOIN inventory table
SELECT
	f.film_id,
	title,
	inventory_id
FROM
	film f
LEFT JOIN inventory i
   ON i.film_id = f.film_id
WHERE i.film_id IS NULL
ORDER BY title;

-- USING same coloumn
SELECT
	f.film_id,
	title,
	inventory_id
FROM
	film f
LEFT JOIN inventory i USING (film_id)
WHERE i.film_id IS NULL
ORDER BY title;
```

### RIGHT JOIN

先依右表資料為主，左表欄位沒有對到的話填NULL

```sql
-- data
DROP TABLE IF EXISTS films;
DROP TABLE IF EXISTS film_reviews;

CREATE TABLE films(
   film_id SERIAL PRIMARY KEY,
   title varchar(255) NOT NULL
);

INSERT INTO films(title)
VALUES('Joker'),
      ('Avengers: Endgame'),
      ('Parasite');

CREATE TABLE film_reviews(
   review_id SERIAL PRIMARY KEY,
   film_id INT,
   review VARCHAR(255) NOT NULL	
);

INSERT INTO film_reviews(film_id, review)
VALUES(1, 'Excellent'),
      (1, 'Awesome'),
      (2, 'Cool'),
      (NULL, 'Beautiful');
```

```sql
SELECT review, title
FROM films
RIGHT JOIN film_reviews USING (film_id)
WHERE title IS NULL;
```

### Self-Join

使用同一個表但不同別名兩次來查詢

```sql
CREATE TABLE employee (
	employee_id INT PRIMARY KEY,
	first_name VARCHAR (255) NOT NULL,
	last_name VARCHAR (255) NOT NULL,
	manager_id INT,
	FOREIGN KEY (manager_id) 
	REFERENCES employee (employee_id) 
	ON DELETE CASCADE
);
INSERT INTO employee (
	employee_id,
	first_name,
	last_name,
	manager_id
)
VALUES
	(1, 'Windy', 'Hays', NULL),
	(2, 'Ava', 'Christensen', 1),
	(3, 'Hassan', 'Conner', 1),
	(4, 'Anna', 'Reeves', 2),
	(5, 'Sau', 'Norman', 2),
	(6, 'Kelsie', 'Hays', 3),
	(7, 'Tory', 'Goff', 3),
	(8, 'Salley', 'Lester', 3);
```

```sql
-- 找主管
SELECT
    e.first_name || ' ' || e.last_name employee,
    m .first_name || ' ' || m .last_name manager
FROM
    employee e
INNER JOIN employee m ON m.employee_id = e.manager_id
ORDER BY manager;

-- 找同樣長度的電影pair
SELECT
    f1.title,
    f2.title,
    f1.length
FROM
    film f1
INNER JOIN film f2 
    ON f1.film_id <> f2.film_id AND 
       f1.length = f2.length
```

### FULL OUTER JOIN

= left join + right join，對不到就填NULL

```sql
DROP TABLE IF EXISTS departments;
DROP TABLE IF EXISTS employees;

CREATE TABLE departments (
	department_id serial PRIMARY KEY,
	department_name VARCHAR (255) NOT NULL
);

CREATE TABLE employees (
	employee_id serial PRIMARY KEY,
	employee_name VARCHAR (255),
	department_id INTEGER
);

INSERT INTO departments (department_name)
VALUES
	('Sales'),
	('Marketing'),
	('HR'),
	('IT'),
	('Production');

INSERT INTO employees (
	employee_name,
	department_id
)
VALUES
	('Bette Nicholson', 1),
	('Christian Gable', 1),
	('Joe Swank', 2),
	('Fred Costner', 3),
	('Sandra Kilmer', 4),
	('Julia Mcqueen', NULL);
```

```sql
SELECT
	employee_name,
	department_name
FROM
	employees e
FULL OUTER JOIN departments d 
        ON d.department_id = e.department_id;
```

### Cross Join

Cartesian Produc(笛卡爾積): 所有可能的有序對之集合 N croos join M = N*M rows。

```sql
-- data
DROP TABLE IF EXISTS T1;
CREATE TABLE T1 (label CHAR(1) PRIMARY KEY);

DROP TABLE IF EXISTS T2;
CREATE TABLE T2 (score INT PRIMARY KEY);

INSERT INTO T1 (label)
VALUES
	('A'),
	('B');

INSERT INTO T2 (score)
VALUES
	(1),
	(2),
	(3);
```

```sql
-- CROSS JOIN
SELECT *
FROM T1
CROSS JOIN T2;

-- equal to
SELECT *
FROM T1, T2;

-- equal to
SELECT *
FROM T1
INNER JOIN T2 ON true;
```

### Natural Join

依相同的coloumn自動建立隱式連接。

```sql
SELECT select_list
FROM T1
NATURAL [INNER, LEFT, RIGHT] JOIN T2;

-- default: INNER
```

```sql
-- data
DROP TABLE IF EXISTS categories;
CREATE TABLE categories (
	category_id serial PRIMARY KEY,
	category_name VARCHAR (255) NOT NULL
);

DROP TABLE IF EXISTS products;
CREATE TABLE products (
	product_id serial PRIMARY KEY,
	product_name VARCHAR (255) NOT NULL,
	category_id INT NOT NULL,
	FOREIGN KEY (category_id) REFERENCES categories (category_id)
);

INSERT INTO categories (category_name)
VALUES
	('Smart Phone'),
	('Laptop'),
	('Tablet');

INSERT INTO products (product_name, category_id)
VALUES
	('iPhone', 1),
	('Samsung Galaxy', 1),
	('HP Elite', 2),
	('Lenovo Thinkpad', 2),
	('iPad', 3),
	('Kindle Fire', 3);
```

```sql
-- Natural Join

SELECT * FROM products
NATURAL JOIN categories;

-- equal
SELECT	* FROM products
INNER JOIN categories USING (category_id);
```

## Section 4. Grouping Data

### GROUP BY

GROUP BY 將 SELECT 得到的結果分成各組。對於每個組，可以應用一個集合函數，例如SUM()來計算總和，或者COUNT()來計算個數。(沒用集合函數的話結果會跟distinctㄧ樣)

```sql
-- Group by id + sum
SELECT
	customer_id,
	SUM (amount)
FROM
	payment
GROUP BY
	customer_id
ORDER BY
	SUM (amount) DESC;

-- group by id + count
SELECT
	staff_id,
	COUNT (payment_id)
FROM
	payment
GROUP BY
	staff_id;

-- group by date
SELECT 
	DATE(payment_date) paid_date, 
	SUM(amount) sum
FROM 
	payment
GROUP BY
	DATE(payment_date);

-- group by with join table
SELECT
	first_name || ' ' || last_name full_name,
	SUM (amount) amount
FROM
	payment
INNER JOIN customer USING (customer_id)
GROUP BY
	full_name
ORDER BY
	amount;
	
-- group by mulitple coloumns
SELECT 
	customer_id, 
	staff_id, 
	SUM(amount) 
FROM 
	payment
GROUP BY 
	staff_id, 
	customer_id
ORDER BY 
    customer_id;
```

### HAVING

HAVING通常與GROUP BY一起使用，根據指定的條件過濾組或集合。

WHERE 是過濾 rows，而HAVING是過濾 groups。

!! Postgres不能在 HAVING、WHERE 裡使用 coloum alias。

```sql
-- having + sum
SELECT
	customer_id,
	SUM (amount)
FROM
	payment
GROUP BY
	customer_id
HAVING
	SUM (amount) > 200;

-- having + count
SELECT
	store_id,
	COUNT (customer_id)
FROM
	customer
GROUP BY
	store_id
HAVING
	COUNT (customer_id) > 300;
```



