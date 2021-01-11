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

### PostgreSQL Column Alias

```sql
-- Alias
SELECT
    first_name || ' ' || last_name AS "full_name"
FROM
    customer;
 
SELECT 5 * 3 AS "5 * 3";
```

### PostgreSQL ORDER BY

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

### PostgreSQL SELECT DISTINCT

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

