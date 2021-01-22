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

## Section 3. Joining Multiple Tables

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

Cartesian Produc(笛卡爾積): 所有可能的有序對之集合 N croos join M = N * M rows

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

## Section 5. Set Operations

### UNION

結合多個表的結果：聯集

-   需要選取相同順序、數量的 coloumns
-   資料類型必須相容

-   union 會移除重複的 rows；union all 會保留

```sql
-- data
DROP TABLE IF EXISTS top_rated_films;
CREATE TABLE top_rated_films(
	title VARCHAR NOT NULL,
	release_year SMALLINT
);

DROP TABLE IF EXISTS most_popular_films;
CREATE TABLE most_popular_films(
	title VARCHAR NOT NULL,
	release_year SMALLINT
);

INSERT INTO 
   top_rated_films(title,release_year)
VALUES
   ('The Shawshank Redemption',1994),
   ('The Godfather',1972),
   ('12 Angry Men',1957);

INSERT INTO 
   most_popular_films(title,release_year)
VALUES
   ('An American Pickle',2020),
   ('The Godfather',1972),
   ('Greyhound',2020);
```

```sql
-- union(5 rows)
SELECT * FROM top_rated_films
UNION
SELECT * FROM most_popular_films;
-- union all(6 rows)
SELECT * FROM top_rated_films
UNION ALL
SELECT * FROM most_popular_films;

-- union all order by
SELECT * FROM top_rated_films
UNION ALL
SELECT * FROM most_popular_films
ORDER BY release_year;
```

### INTERSECT

結合多個表的結果：交集

-   需要選取相同順序、數量的 coloumns
-   資料類型必須相容

```sql
-- INTERSECT
SELECT *
FROM most_popular_films 
INTERSECT
SELECT *
FROM top_rated_films;
```

### Except

結合多個表的結果：差集

-   需要選取相同順序、數量的 coloumns
-   資料類型必須相容

```sql
-- EXCEPT
SELECT *
FROM top_rated_films
EXCEPT 
SELECT *
FROM most_popular_films;
```

## Section 6. Grouping sets, Cube, and Rollup

### GROUPING SETS

- Grouping sets 可以讓你在同一個查詢中定義多個 group 結果，不然就要查詢很多次必用 union all 連結在一起
- GROUPING() 回傳0/1，可以知道參數是否為當前 grouping sets 的成員

```sql
-- data
DROP TABLE IF EXISTS sales;
CREATE TABLE sales (
    brand VARCHAR NOT NULL,
    segment VARCHAR NOT NULL,
    quantity INT NOT NULL,
    PRIMARY KEY (brand, segment)
);

INSERT INTO sales (brand, segment, quantity)
VALUES
    ('ABC', 'Premium', 100),
    ('ABC', 'Basic', 200),
    ('XYZ', 'Premium', 100),
    ('XYZ', 'Basic', 300);
```

```sql
-- grouping set
SELECT
    brand,
    segment,
    SUM (quantity)
FROM
    sales
GROUP BY
    GROUPING SETS (
        (brand, segment),
        (brand),
        (segment),
        ()
    );
    
-- grouping set + grouping
SELECT
	GROUPING(brand) grouping_brand,
	GROUPING(segment) grouping_segment,
	brand,
	segment,
	SUM (quantity)
FROM
	sales
GROUP BY
	GROUPING SETS (
		(brand),
		(segment),
		()
	)
ORDER BY
	brand,
	segment;
	
-- grouping set + grouping + having
Select
	GROUPING(brand) grouping_brand,
	GROUPING(segment) grouping_segment,
	brand,
	segment,
	SUM (quantity)
FROM
	sales
GROUP BY
	GROUPING SETS (
		(brand),
		(segment),
		()
	)
HAVING GROUPING(brand) = 0	
ORDER BY
	brand,
	segment;
```

### CUBE

CUBE 為 GROUP BY 的子句，用來產生所有 Grouping sets 的組合結果，CUBE(n個數) = 2^n Grouping sets。

```sql
-- 兩個結果為相同
CUBE(c1,c2,c3) 

GROUPING SETS (
    (c1,c2,c3), 
    (c1,c2),
    (c1,c3),
    (c2,c3),
    (c1),
    (c2),
    (c3), 
    ()
 )
```

```sql
-- CUBE ＝ (brnad,segment), (brand), (segment), ()
SELECT
	GROUPING(brand) grouping_brand,
	GROUPING(segment) grouping_segment,
    brand,
    segment,
    SUM (quantity)
FROM
    sales
GROUP BY
    CUBE (brand, segment)
ORDER BY
    brand,
    segment;

-- partial cube = (brand, segment), (brand)
SELECT
    brand,
    segment,
    SUM (quantity)
FROM
    sales
GROUP BY
    brand,
    CUBE (segment)
ORDER BY
    brand,
    segment;
```

### ROLLUP

ROLLUP 為 GROUP BY 的子句，用來產生部分、有層次(例如年->月->日) Grouping sets 的組合結果，ROLLUP(n個數) = n+1 Grouping sets。

```sql
-- 兩個結果為相同
ROLLUP(c1,c2,c3) 

GROUPING SETS (
    (c1,c2,c3), 
    (c1,c2),
    (c1),
    ()
 )
```

```sql
-- ROLLUP(年、月、日)
SELECT
    EXTRACT (YEAR FROM rental_date) y,
    EXTRACT (MONTH FROM rental_date) M,
    EXTRACT (DAY FROM rental_date) d,
    COUNT (rental_id)
FROM
    rental
GROUP BY
    ROLLUP (
        EXTRACT (YEAR FROM rental_date),
        EXTRACT (MONTH FROM rental_date),
        EXTRACT (DAY FROM rental_date)
    );
```

```sql
-- select 順序很重要
---- (brand,segement), (brand), ()
SELECT
    brand,
    segment,
    SUM (quantity)
FROM
    sales
GROUP BY
    ROLLUP (brand, segment)
ORDER BY
    brand,
    segment;
    
---- (segment,brand), (segment), ()
SELECT
    segment,
    brand,
    SUM (quantity)
FROM
    sales
GROUP BY
    ROLLUP (brand, segment)
ORDER BY
    brand,
    segment;
    
-- partial rollup = (segment,brand), (segment)
SELECT
    segment,
    brand,
    SUM (quantity)
FROM
    sales
GROUP BY
    segment,
    ROLLUP (brand)
ORDER BY
    segment,
    brand;
```

## Section 7. Subquery

### Subquery

子查詢將原本需要兩步驟的查詢結合，另外也可以用在 SELECT, INSERT, DELETE, UPDATE中。

PostgresSQL 執行順序，執行子查詢->拿到結果->塞回主查詢中

```sql
-- subquery: 找高於平均評分的電影
SELECT
	film_id,
	title,
	rental_rate
FROM
	film
WHERE
	rental_rate > (
		SELECT
			AVG (rental_rate)
		FROM
			film
	);

-- subquery: 依照dvd還回的時間來找電影
SELECT
	film_id,
	title
FROM
	film
WHERE
	film_id IN (
		SELECT
			inventory.film_id
		FROM
			rental
		INNER JOIN inventory ON inventory.inventory_id = rental.inventory_id
		WHERE
			return_date BETWEEN '2005-05-29'
		AND '2005-05-30'
	);
```

EXISTS 只關心是否至少有一筆資料存在並直接返回true(效率較高)，所以通常會這樣寫`EXISTS (SELECT 1 FROM tbl WHERE condition);`

```sql
-- subquery + exists = 221.92
SELECT
	first_name,
	last_name
FROM
	customer
WHERE
	EXISTS (
		SELECT
			1
		FROM
			payment
		WHERE
			payment.customer_id = customer.customer_id
	);

-- distinct + innerjoin = 393.99
SELECT
       DISTINCT first_name, last_name
FROM
	customer 
INNER JOIN payment USING(customer_id);
```

### ANY

用在與子查詢結果做比較

- 子查詢必須回傳剛好一個 coloumn
- ANY  之前必須使用關係運算子，也就是大於、等於、小於那些
- 如果符合條件回傳 true 否則回傳 false
- `=ANY` == `IN`
- `<> ANY` != NOT IN; 
  - `x <> ANY (a,b,c)`
  - = `x <> a OR <> b OR x <> c`

```sql
-- 尋找影片長度大於 Min(Max(category))
SELECT title
FROM film
WHERE length >= ANY(
    SELECT MAX( length )
    FROM film
    INNER JOIN film_category USING(film_id)
    GROUP BY  category_id );

```

### ALL

用在與子查詢結果做比較

- ALL  之前必須使用關係運算子，也就是大於、等於、小於那些
  - `>`: 尋找比子查詢最大值還大的值
  - `<`: 尋找比子查詢最小值還小的值
  - `=`: 尋找和子查詢任一值相等的值
  - `!=`: 尋找和子查詢任一值都不相等的值

```sql
-- 影片長度大於相同分級裡的平均長度
SELECT
    film_id,
    title,
    length
FROM
    film
WHERE
    length > ALL (
            SELECT
                ROUND(AVG (length),2)
            FROM
                film
            GROUP BY
                rating
    )
ORDER BY
    length;
```

### EXISTS

用在測試子查詢中是否存在任合一筆資料

- 如果子查詢至少有一筆資料則回傳 true，如果沒有則回傳 false。
- 通常與相關的子查詢一起使用
- select 的內容不是重點， EXISTS 只關心是否存在任合一筆資料，所以會用 `select 1`

```sql
-- 尋找曾經交易金額大於11的顧客姓名
SELECT first_name,
       last_name
FROM customer c
WHERE EXISTS
    (SELECT 1
     FROM payment p
     WHERE p.customer_id = c.customer_id
       AND amount > 11 )
ORDER BY first_name,
         last_name;
         
-- 尋找交易金額從未大於11的顧客姓名
SELECT first_name,
       last_name
FROM customer c
WHERE NOT EXISTS
    (SELECT 1
     FROM payment p
     WHERE p.customer_id = c.customer_id
       AND amount > 11 )
ORDER BY first_name,
         last_name;
```

```sql
-- EXISTS( SELECT NULL) = true
SELECT
	first_name,
	last_name
FROM
	customer
WHERE
	EXISTS( SELECT NULL )
ORDER BY
	first_name,
	last_name;
```

## Section 8. Common Table Expressions

### CTE

CTE 是一個臨時的結果集，你可以在另一個SQL句中引用，包括SELECT、INSERT、UPDATE以及DELETE。

```sql
WITH cte_name (column_list) AS (
    CTE_query_definition 
)
statement;
```

- 建立 cte_name 臨時表然後用在 statement 中
- 不指定 column_list 的話 CTE_query_definition 裡查的 column 就會變 CTE 的 column

```sql
-- create cte -> use cte
WITH cte_film AS (
    SELECT 
        film_id, 
        title,
        (CASE 
            WHEN length < 30 THEN 'Short'
            WHEN length < 90 THEN 'Medium'
            ELSE 'Long'
        END) length    
    FROM
        film
) SELECT 
    film_id,
    title,
    length
FROM 
    cte_film
WHERE
    length = 'Long'
ORDER BY 
    title;
```

```sql
-- cte + join
WITH cte_rental AS (
    SELECT staff_id,
        COUNT(rental_id) rental_count
    FROM   rental
    GROUP  BY staff_id
) SELECT s.staff_id,
    first_name,
    last_name,
    rental_count
FROM staff s
    INNER JOIN cte_rental USING (staff_id);
```

```Sql
-- cte + window function
WITH cte_film AS  (
    SELECT film_id,
        title,
        rating,
        length,
        RANK() OVER (
            PARTITION BY rating
            ORDER BY length DESC) 
        length_rank
    FROM 
        film
) SELECT *
FROM cte_film
WHERE length_rank = 1;
```

CTE 優點

- 複雜查詢的可讀性
- 可以做遞迴查詢
- 可以搭配 [window function](https://www.postgresqltutorial.com/postgresql-window-function/)

### Recursive Query

```sql
-- 語法
WITH RECURSIVE cte_name AS(
    CTE_query_definition -- non-recursive term
    UNION [ALL]
    CTE_query definion  -- recursive term
) SELECT * FROM cte_name;
```

Recursive CTE 運作流程

1. 執行 non-recursive term 建立基本結果集(R0)
2. 執行 recursive term 建立遞迴結果集(Ri)直到回傳空結果集，Ri 當 input；Ri+1當 output；i=0..n
3. 回傳 UNION [ALL] 的結果 R0,R1...RN

```sql
-- data
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
	employee_id serial PRIMARY KEY,
	full_name VARCHAR NOT NULL,
	manager_id INT
);

INSERT INTO employees (
	employee_id,
	full_name,
	manager_id
)
VALUES
	(1, 'Michael North', NULL),
	(2, 'Megan Berry', 1),
	(3, 'Sarah Berry', 1),
	(4, 'Zoe Black', 1),
	(5, 'Tim James', 1),
	(6, 'Bella Tucker', 2),
	(7, 'Ryan Metcalfe', 2),
	(8, 'Max Mills', 2),
	(9, 'Benjamin Glover', 2),
	(10, 'Carolyn Henderson', 3),
	(11, 'Nicola Kelly', 3),
	(12, 'Alexandra Climo', 3),
	(13, 'Dominic King', 3),
	(14, 'Leonard Gray', 4),
	(15, 'Eric Rampling', 4),
	(16, 'Piers Paige', 7),
	(17, 'Ryan Henderson', 7),
	(18, 'Frank Tucker', 8),
	(19, 'Nathan Ferguson', 8),
	(20, 'Kevin Rampling', 8);
```

```sql
-- 找主管員工關係
WITH RECURSIVE subordinates AS (
	SELECT
		employee_id,
		manager_id,
		full_name
	FROM
		employees
	WHERE
		employee_id = 2
	UNION
    SELECT
	    e.employee_id,
    	e.manager_id,
	    e.full_name
    FROM
    	employees e
	    INNER JOIN subordinates s ON s.employee_id = e.manager_id
) SELECT
	*
FROM
	subordinates;
```

## Section 9. Modifying Data

### INSERT

```sql
INSERT INTO table_name(column1, column2, …)
VALUES (value1, value2, …);

-- return
>>> INSERT oid count
```

```sql
-- data
DROP TABLE IF EXISTS links;

CREATE TABLE links (
	id SERIAL PRIMARY KEY,
	url VARCHAR(255) NOT NULL,
	name VARCHAR(255) NOT NULL,
	description VARCHAR (255),
        last_update DATE
);
```

```sql
-- insert single row and omit optional
INSERT INTO links (url, name)
VALUES('https://www.postgresqltutorial.com','PostgreSQL Tutorial');

-- insert contain single quote
INSERT INTO links (url, name)
VALUES('http://www.oreilly.com','O''Reilly Media');

-- insert date value YYYY-MM-DD
INSERT INTO links (url, name, last_update)
VALUES('https://www.google.com','Google','2013-06-01');

-- Geting the last insert id
INSERT INTO links (url, name)
VALUES('http://www.postgresql.org','PostgreSQL') 
RETURNING id;

-- Geting the last insert id
INSERT INTO links (url, name)
VALUES('http://www.example.com','Example') 
RETURNING *;
```

### INSERT Multiple Rows

```sql
INSERT INTO table_name (column_list)
VALUES
    (value_list_1),
    (value_list_2),
    ...
    (value_list_n)
RETURNING * | output_expression AS output_name;
```

```sql
-- data
DROP TABLE IF EXISTS links;

CREATE TABLE links (
    id SERIAL PRIMARY KEY,
    url VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    description VARCHAR(255)
);
```

```sql
-- insert multiple rows
INSERT INTO 
    links (url, name)
VALUES
    ('https://www.google.com','Google'),
    ('https://www.yahoo.com','Yahoo'),
    ('https://www.bing.com','Bing')
RETURNING id, url;

-- insert without coloumn list
INSERT INTO 
    links
VALUES
    (Default, 'https://www.searchencrypt.com/','SearchEncrypt','Search Encrypt'),
    (Default, 'https://www.startpage.com/','Startpage','The world''s most private search engine')
RETURNING *;
```

### UPDATE

```sql
UPDATE table_name
SET column1 = value1,
    column2 = value2,
    ...
WHERE condition
RETURNING * | output_expression AS output_name;

-- return
UPDATE count
```

```sql
-- data
DROP TABLE IF EXISTS courses;

CREATE TABLE courses(
	course_id serial primary key,
	course_name VARCHAR(255) NOT NULL,
	description VARCHAR(500),
	published_date date
);

INSERT INTO 
	courses(course_name, description, published_date)
VALUES
	('PostgreSQL for Developers','A complete PostgreSQL for Developers','2020-07-13'),
	('PostgreSQL Admininstration','A PostgreSQL Guide for DBA',NULL),
	('PostgreSQL High Performance',NULL,NULL),
	('PostgreSQL Bootcamp','Learn PostgreSQL via Bootcamp','2013-07-11'),
	('Mastering PostgreSQL','Mastering PostgreSQL in 21 Days','2012-06-30');
```

```sql
--update row
UPDATE courses
SET published_date = '1970-01-01' 
WHERE published_date is NULL
RETURNING *;
```

### UPDATE join

當你需要根據其他table的資料來更新這個table資料時。

```sql
UPDATE t1
SET t1.c1 = new_value
FROM t2
WHERE t1.c2 = t2.c2;
```

```sql
-- data
DROP TABLE IF EXISTS product_segment;
CREATE TABLE product_segment (
    id SERIAL PRIMARY KEY,
    segment VARCHAR NOT NULL,
    discount NUMERIC (4, 2)
);
INSERT INTO 
    product_segment (segment, discount)
VALUES
    ('Grand Luxury', 0.05),
    ('Luxury', 0.06),
    ('Mass', 0.1);
    
-- 
DROP TABLE IF EXISTS product;
CREATE TABLE product(
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL,
    price NUMERIC(10,2),
    net_price NUMERIC(10,2),
    segment_id INT NOT NULL,
    FOREIGN KEY(segment_id) REFERENCES product_segment(id)
);
INSERT INTO 
    product (name, price, segment_id) 
VALUES 
    ('diam', 804.89, 1),
    ('vestibulum aliquet', 228.55, 3),
    ('lacinia erat', 366.45, 2),
    ('scelerisque quam turpis', 145.33, 3),
    ('justo lacinia', 551.77, 2),
    ('ultrices mattis odio', 261.58, 3),
    ('hendrerit', 519.62, 2),
    ('in hac habitasse', 843.31, 1),
    ('orci eget orci', 254.18, 3),
    ('pellentesque', 427.78, 2),
    ('sit amet nunc', 936.29, 1),
    ('sed vestibulum', 910.34, 1),
    ('turpis eget', 208.33, 3),
    ('cursus vestibulum', 985.45, 1),
    ('orci nullam', 841.26, 1),
    ('est quam pharetra', 896.38, 1),
    ('posuere', 575.74, 2),
    ('ligula', 530.64, 2),
    ('convallis', 892.43, 1),
    ('nulla elit ac', 161.71, 3);
```

```sql
-- 根據 segment 打折
UPDATE product as p
SET net_price = price - price * discount
FROM product_segment as s
WHERE p.segment_id = s.id
RETURNING *;
```

### DELETE

```sql
DELETE FROM table_name
WHERE condition
RETURNING * | output_expression AS output_name;

-- return
DELETE count
```

```sql
-- data
DROP TABLE IF EXISTS links;

CREATE TABLE links (
    id serial PRIMARY KEY,
    url varchar(255) NOT NULL,
    name varchar(255) NOT NULL,
    description varchar(255),
    rel varchar(10),
    last_update date DEFAULT now()
);

INSERT INTO  
   links 
VALUES 
   ('1', 'https://www.postgresqltutorial.com', 'PostgreSQL Tutorial', 'Learn PostgreSQL fast and easy', 'follow', '2013-06-02'),
   ('2', 'http://www.oreilly.com', 'O''Reilly Media', 'O''Reilly Media', 'nofollow', '2013-06-02'),
   ('3', 'http://www.google.com', 'Google', 'Google', 'nofollow', '2013-06-02'),
   ('4', 'http://www.yahoo.com', 'Yahoo', 'Yahoo', 'nofollow', '2013-06-02'),
   ('5', 'http://www.bing.com', 'Bing', 'Bing', 'nofollow', '2013-06-02'),
   ('6', 'http://www.facebook.com', 'Facebook', 'Facebook', 'nofollow', '2013-06-01'),
   ('7', 'https://www.tumblr.com/', 'Tumblr', 'Tumblr', 'nofollow', '2013-06-02'),
   ('8', 'http://www.postgresql.org', 'PostgreSQL', 'PostgreSQL', 'nofollow', '2013-06-02');
```

```sql
-- delete by id 
DELETE FROM links
WHERE id = 7
RETURNING *;

-- delete multiple use in 
DELETE FROM links
WHERE id IN (6,5)
RETURNING *;

-- delete all rows
DELETE FROM links;
```

### Delete join

```sql
DELETE FROM t1
USING t2
WHERE t1.id = t2.id
```

```sql
-- data
DROP TABLE IF EXISTS contacts;
CREATE TABLE contacts(
   contact_id serial PRIMARY KEY,
   first_name varchar(50) NOT NULL,
   last_name varchar(50) NOT NULL,
   phone varchar(15) NOT NULL
);


DROP TABLE IF EXISTS blacklist;
CREATE TABLE blacklist(
    phone varchar(15) PRIMARY KEY
);


INSERT INTO contacts(first_name, last_name, phone)
VALUES ('John','Doe','(408)-523-9874'),
       ('Jane','Doe','(408)-511-9876'),
       ('Lily','Bush','(408)-124-9221');


INSERT INTO blacklist(phone)
VALUES ('(408)-523-9874'),
       ('(408)-511-9876');
```

```sql
-- delte with using clause(not sql standard)
DELETE FROM contacts 
USING blacklist
WHERE contacts.phone = blacklist.phone;

-- delete join using subquery
DELETE FROM contacts
WHERE phone IN (SELECT phone FROM blacklist);
```

### UPSERT

如果存在就update不存在就insert，類似mysql的 [insert on duplicate key update statement](http://www.mysqltutorial.org/mysql-insert-or-update-on-duplicate-key-update/) 

```sql
INSERT INTO table_name(column_list) 
VALUES(value_list)
ON CONFLICT target action;
-- action: DO NOTHING || DO UPDATE
```

```sql
-- data
DROP TABLE IF EXISTS customers;

CREATE TABLE customers (
	customer_id serial PRIMARY KEY,
	name VARCHAR UNIQUE,
	email VARCHAR NOT NULL,
	active bool NOT NULL DEFAULT TRUE
);
INSERT INTO 
    customers (name, email)
VALUES 
    ('IBM', 'contact@ibm.com'),
    ('Microsoft', 'contact@microsoft.com'),
    ('Intel', 'contact@intel.com');
```

```sql
-- upsert ON CONFLICT ON CONSTRAINT + do nothing
INSERT INTO customers (NAME, email)
VALUES('Microsoft','hotline@microsoft.com') 
ON CONFLICT ON CONSTRAINT customers_name_key 
DO NOTHING;

-- upsert ON CONFLICT(target) + do update
INSERT INTO customers (name, email)
VALUES('Microsoft','hotline@microsoft.com') 
ON CONFLICT (name) 
DO 
   UPDATE SET email = EXCLUDED.email || ';' || customers.email;
```

## Section 10. Transactions

交易是由一個或多個操作組成的工作單元，Transaction 用四個特性 ACID 來保證正確可靠

- Atomicity: 一個交易(transaction)中的所有操作，不是全部完成，就是全部不完成，不會結束在中間某個環節。

- Consistency: 在交易開始之前和結束以後，資料庫的完整性沒有被破壞。

- Isolation: 資料庫允許多個並行交易同時對其資料進行讀寫和修改的能力，隔離性可以防止多個交易並行執行時由於交叉執行而導致資料的不一致。交易隔離分為不同級別，包括
  - Read uncommitted
  - Read committed
  - Repeatable read
  - Serializable

- Durability: 交易處理結束後，對資料的修改就是永久的，即便系統故障也不會遺失。

```sql
-- begin 
BEGIN TRANSACTION;
BEGIN WORK;
BEGIN
-- commit
COMMIT TRANSACTION;
COMMIT WORK;
COMMIT;
-- rollback
ROLLBACK TRANSACTION;
ROLLBACK WORK;
ROLLBACK;
```

```sql
-- data
DROP TABLE If EXISTS accounts;

CREATE TABLE accounts (
    id INT GENERATED BY DEFAULT AS IDENTITY,
    name VARCHAR(100) NOT NULL,
    balance DEC(15,2) NOT NULL,
    PRIMARY KEY(id)
);
```

在 COMMIT 之前另一個 session 還看不到結果，如果它也同時執行交易時會等待 commit 或 rollback

```sql
-- start a transaction
BEGIN;

-- insert a new row into the accounts table
INSERT INTO accounts(name,balance)
VALUES('Alice',10000);

-- commit the change (or roll it back later)
COMMIT;
```

```sql
-- bank account transfer with commit
-- start a transaction
BEGIN;

-- deduct 1000 from account 1
UPDATE accounts 
SET balance = balance - 1000
WHERE id = 1;

-- add 1000 to account 2
UPDATE accounts
SET balance = balance + 1000
WHERE id = 2; 

-- select the data from accounts
SELECT id, name, balance
FROM accounts;

-- commit the transaction
COMMIT;
```

```sql
-- bank account transfer with rollback
-- begin the transaction
BEGIN;

-- deduct the amount from the account 1
UPDATE accounts 
SET balance = balance - 1500
WHERE id = 1;

-- add the amount from the account 3 (instead of 2)
UPDATE accounts
SET balance = balance + 1500
WHERE id = 3; 

-- roll back the transaction
ROLLBACK;
```

## Section 11. Import & Export Data

### Import CSV File Into PostgreSQL Table

- person.csv

```
First Name,Last Name,Date Of Birth,Email
John,Doe,1995-01-05,john.doe@postgresqltutorial.com
Jane,Doe,1995-02-05,jane.doe@postgresqltutorial.com
```

```sql
-- data
Drop TABLE IF EXISTS persons;
CREATE TABLE persons (
  id SERIAL,
  first_name VARCHAR(50),
  last_name VARCHAR(50),
  dob DATE,
  email VARCHAR(255),
  PRIMARY KEY (id)
);
```

```sql
\copy persons(first_name, last_name, dob, email)
FROM './data/persons.csv'
DELIMITER ','
CSV HEADER;
```

### Export PostgreSQL Table to CSV file

```sql
\copy persons
to './data/export_persons.csv'
DELIMITER ','
CSV HEADER;
```

```sql
\copy persons(first_name,last_name,email) 
to './data/export_persons_partial.csv'
DELIMITER ','
CSV HEADER;
```

```sql
\copy (SELECT email FROM persons)
to './data/export_persons_email.csv'
with csv
```

```sql
-- 清掉db內容
TRUNCATE TABLE persons 
RESTART IDENTITY;
-- 刪掉 db
DROP TABLE persons;
-- 刪掉 export 資料
\! rm data/export_persons*
```

## Section 12. Managing Tables

### Data types

[參考doc比較好](https://www.postgresql.org/docs/current/arrays.html)

- boolean
  - `boolean` or `bool`
  - 1, yes, y, t, ture == true
  - 0, no, false, f  == false
- character
  - `char(N)`: 固定長度，輸入長度<N填充空，輸入長度>N回報錯誤
  - `varchar(N)`: 變動長度
  - `text`: 理論上無限長度
- numeric
  - interger
    - `smallint`: 2-byte有符號整數，-32,768 ~ 32,767
    - `int`: 4-byte有符號整數，-2,147,483,648 ~ 2,147,483,647
    - `serial`: 像是MySQL的`AUTO_INCREAMENT`，SQLite的`AUTOINCREAMENT`
  - float
    - `float(n)`: n精度浮點數，
    - `real` / `float8`: 4byte 浮點數/ 8byte浮點數
    - `numeric` / `numeric(p,s)`: 自訂長度p以且s位小數
- temporal data
  - `date`
  - `time`
  - `timestamp`
  - `timestamptz`: timestamp with the timezone
  - `interval`
- Array
- JSON
  - `JSON`
  - `JSONB`
- `UUID`
- special data types 
  - box
  - line
  - point
  - lseg: line segment
  - polygon: closed geometric
  - inet: IPV4
  - macaddr

### CREATE TABLE

```sql
CREATE TABLE [IF NOT EXISTS] table_name (
   column1 datatype(length) column_contraint,
   column2 datatype(length) column_contraint,
   column3 datatype(length) column_contraint,
   table_constraints
);
```

Column Constraints 

- NULL
- UNIQUE
- PRIMARY KEY
- CHECK
- FOREIGNKEY

```sql
DROP TABLE IF EXISTS accounts, roles, account_roles;
-- 帳戶
CREATE TABLE accounts (
	user_id serial PRIMARY KEY,
	username VARCHAR ( 50 ) UNIQUE NOT NULL,
	password VARCHAR ( 50 ) NOT NULL,
	email VARCHAR ( 255 ) UNIQUE NOT NULL,
	created_on TIMESTAMP NOT NULL,
        last_login TIMESTAMP 
);

-- 角色
CREATE TABLE roles(
   role_id serial PRIMARY KEY,
   role_name VARCHAR (255) UNIQUE NOT NULL
);

-- 帳戶角色
CREATE TABLE account_roles (
  user_id INT NOT NULL,
  role_id INT NOT NULL,
  grant_date TIMESTAMP,
    
  PRIMARY KEY (user_id, role_id),
  FOREIGN KEY (role_id)
      REFERENCES roles (role_id),
  FOREIGN KEY (user_id)
      REFERENCES accounts (user_id)
);
```

### SELECT INTO ＆ CREATE TABLE AS

建立新的表並插入查詢的資料。

```sql
SELECT
    select_list
INTO [ TEMPORARY | TEMP | UNLOGGED ] [ TABLE ] new_table_name
FROM
    table_name
WHERE
    search_condition;
    
--

CREATE TABLE IF NOT EXISTS new_table_name (column_name_list) 
AS query;
```

```sql
-- 長度<60分的電影放到暫存表(session scope)
SELECT
    film_id,
    title,
    length 
INTO TEMP TABLE short_film
FROM
    film
WHERE
    length < 60
ORDER BY
    title;
   
-- PL/pgSQL 用 CREATE TABLE AS
CREATE TABLE short_film AS
SELECT
    film_id,
    title,
    length 
FROM
    film
WHERE
    length < 60
ORDER BY
    title;
```

### Auto-increment Column

```sql
CREATE TABLE table_name(
    id SERIAL
);

-- equal
CREATE SEQUENCE table_name_id_seq;

CREATE TABLE table_name (
    id integer NOT NULL DEFAULT nextval('table_name_id_seq')
);

ALTER SEQUENCE table_name_id_seq
OWNED BY table_name.id;
```

將 SERIAL 指定到某個 coloumn PostgreSQL 執行以下動作:

- 建立 sequence 物件
- 給 coloumn 添加一個 NOT NULL 的限制
- 將 sequence 的擁有者分給 coloumn

SERIAL 

- SMALLSERIAL: 2bytes
- SERIAL: 4 bytes, 1 to 2,147,483,647
- BIGSERIAL: 8 bytes

```sql
-- 水果
DROP TABLE IF EXISTS fruits;
CREATE TABLE fruits(
   id SERIAL PRIMARY KEY,
   name VARCHAR NOT NULL
);

-- 新增橘子
INSERT INTO fruits(name) 
VALUES('Orange');

-- 新增蘋果 + id with default
INSERT INTO fruits(id,name) 
VALUES(DEFAULT,'Apple')
RETURNING id;

-- 最近一次產生的序列值
SELECT currval(pg_get_serial_sequence('fruits', 'id'));

```

### Sequences

根據指定的規範產生有序的整數列表。

```sql
CREATE SEQUENCE [ IF NOT EXISTS ] sequence_name
    [ AS { SMALLINT | INT | BIGINT } ]
    [ INCREMENT [ BY ] increment ]
    [ MINVALUE minvalue | NO MINVALUE ] 
    [ MAXVALUE maxvalue | NO MAXVALUE ]
    [ START [ WITH ] start ] 
    [ CACHE cache ] 
    [ [ NO ] CYCLE ]
    [ OWNED BY { table_name.column_name | NONE } ]
```

```sql
-- ascending sequence
DROP SEQUENCE IF EXISTS mysequence;
CREATE SEQUENCE mysequence
INCREMENT 5
START 100
CACHE 3; -- prepreallocated and stored in memory for faster access. 

SELECT nextval('mysequence');
SELECT nextval('mysequence');
SELECT nextval('mysequence');

-- descending sequence
DROP SEQUENCE IF EXISTS mysequence2;
CREATE SEQUENCE mysequence2
INCREMENT -1
MINVALUE 1 
MAXVALUE 3
START 3
CYCLE;
SELECT nextval('mysequence2');
SELECT nextval('mysequence2');
SELECT nextval('mysequence2');

-- sequence associated with a table column
DROP TABLE IF EXISTS order_details;
DROP SEQUENCE IF EXISTS order_item_id;

CREATE TABLE order_details(
    order_id SERIAL,
    item_id INT NOT NULL,
    item_text VARCHAR NOT NULL,
    price DEC(10,2) NOT NULL,
    PRIMARY KEY(order_id, item_id)
);
CREATE SEQUENCE order_item_id
START 10
INCREMENT 10
MINVALUE 10
OWNED BY order_details.item_id;

INSERT INTO 
    order_details(order_id, item_id, item_text, price)
VALUES
    (100, nextval('order_item_id'),'DVD Player',100),
    (100, nextval('order_item_id'),'Android TV',550),
    (100, nextval('order_item_id'),'Speaker',250)
RETURNING *;
```

```sql
-- 列出所有 sequence
SELECT
    relname sequence_name
FROM 
    pg_class 
WHERE 
    relkind = 'S';
```

```sql
-- drop table 時也會把 sequence 刪掉
DROP TABLE order_details;
SELECT nextval('order_item_id');
```

### Identity Column

GENERATED AS IDENTITY 限制是符合SQL標準的老式 SERIAL 之變體。

```sql
-- constraint
column_name type GENERATED { ALWAYS | BY DEFAULT } AS IDENTITY[ ( sequence_option ) ]
-- add
ALTER TABLE table_name 
ALTER COLUMN column_name 
ADD GENERATED { ALWAYS | BY DEFAULT } AS IDENTITY { ( sequence_option ) }
-- change
ALTER TABLE table_name 
ALTER COLUMN column_name 
{ SET GENERATED { ALWAYS| BY DEFAULT } | 
  SET sequence_option | RESTART [ [ WITH ] restart ] }
-- remove
ALTER TABLE table_name 
ALTER COLUMN column_name 
DROP IDENTITY [ IF EXISTS ]
```

```sql
-- GENERATED  ALWAYS v.s. GENERATED BY DEFAULT

-- GENERATED ALWAYS
DROP TABLE IF EXISTS color;
CREATE TABLE color (
    color_id INT GENERATED ALWAYS AS IDENTITY,
    color_name VARCHAR NOT NULL
);
-- auto generate id
INSERT INTO color(color_name)
VALUES ('Red');
-- error, because GENERATED ALWAYS
INSERT INTO color (color_id, color_name)
VALUES (2, 'Green'); 
-- use OVERRIDING SYSTEM VALUE fix error
INSERT INTO color (color_id, color_name)
OVERRIDING SYSTEM VALUE 
VALUES(2, 'Green');
-- or, alter to generated by default
ALTER TABLE color
ALTER COLUMN color_id
SET GENERATED BY DEFAULT;
INSERT INTO color
VALUES (3, 'White');
```

```sql
-- sequence option example
DROP TABLE IF EXISTS color;

CREATE TABLE color (
    color_id INT GENERATED BY DEFAULT AS IDENTITY 
    (START WITH 10 INCREMENT BY 10),
    color_name VARCHAR NOT NULL
);

INSERT INTO color (color_name)
VALUES ('Orange'),('Purple')
Returning *;
```

```sql
DROP TABLE IF EXISTS shape;
CREATE TABLE shape (
    shape_id INT NOT NULL,
    shape_name VARCHAR NOT NULL
);
-- Adding identity
ALTER TABLE shape 
ALTER COLUMN shape_id ADD GENERATED ALWAYS AS IDENTITY;

\d shape

-- Change identity
ALTER TABLE shape
ALTER COLUMN shape_id
SET GENERATED BY DEFAULT
RESTART WITH 100
SET INCREMENT 10;

\d shape

INSERT INTO shape (shape_name)
VALUES ('Square'), ('Circle')
Returning *;

-- remove identity
ALTER TABLE shape
ALTER COLUMN shape_id
DROP IDENTITY IF EXISTS;

\d shape
```





