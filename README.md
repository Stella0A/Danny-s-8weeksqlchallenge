# My entry for the first case study in Data with Danny's 8weeksqlchallenge @https://8weeksqlchallenge.com/.

## Case Study #1 - Danny's Diner

![image](https://user-images.githubusercontent.com/81259955/173590583-9cc31564-509c-4f19-9b72-25f5957d78ce.png)


## Introduction

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

## Problem statement 

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

1.sales
2.menu
3.members

## Solution

First I created a database that will house the data tables to be used and set the named database as default to be used when running the queries

The USE statement tells MySQL to use the named database as the default (current) database for subsequent statements. 

	CREATE DATABASE dannys_diner;
	
	USE dannys_diner;
	
## Creating the tables and inserting values into it

## sales

	CREATE table sales
	(customer_id VARCHAR(1),
	order_date DATE,
	product_id INT);
	INSERT INTO sales
	VALUES('A','2021-01-01','1'),
	('A','2021-01-01','2'),
	('A','2021-01-07','2'),
	('A','2021-01-10','3'),
	('A','2021-01-11','3'),
	('A','2021-01-11','3'),
	('B','2021-01-01','2'),
	('B','2021-01-02','2'),
	('B','2021-01-04','1'),
	('B','2021-01-11','1'),
	('B','2021-01-16','3'),
	('B','2021-02-01','3'),
	('C','2021-01-01','3'),
	('C','2021-01-01','3'),
	('C','2021-01-07','3');
	
## menu
	CREATE table menu
	(product_id INT references Sales(product_id),
	product_name VARCHAR(15),
	price INT);
	INSERT INTO menu
	VALUES('1','Sushi','10'),
	('2','Curry','15'),
	('3','Ramen','12');

## member
		CREATE table member
		(customer_id VARCHAR(1) references Sales(customer_id),
		join_date DATE);
		INSERT INTO member
		VALUES('A','2021-01-07'),
		('B','2021-01-09');

## Runing sql queries
	SELECT * FROM sales;
	SELECT * FROM menu;
	SELECT * FROM member;
	
## Case study questions

## What is the total amount each customer spent at the restaurant?

	SELECT sal.customer_id, SUM(men.price) AS total_spent
	FROM sales sal
	JOIN menu men
	ON men.product_id = sal.product_id
	GROUP BY sal.customer_id
	ORDER BY total_spent DESC;
	
customer_id | total_spent |
----------- | :--------:  | 
   A   	    |	76	  |
   B 	    |	74        |
   C  	    |	36        |


## How many days has each customer visited the restaurant?

	SELECT customer_id,
	COUNT(DISTINCT order_date) AS visits
	FROM sales
	GROUP BY customer_id
	ORDER BY visits DESC;

customer_id | visits |
----------- | :----: | 
   B  	    |	6    |
   A	    |	4    |
   C  	    |	2    |
   
## What was the first item from the menu purchased by each customer?

	WITH ranked_products AS (
	SELECT mb.customer_id,m.product_name,s.order_date,
	DENSE_RANK() OVER(PARTITION BY mb.customer_id ORDER BY s.order_date) AS purchased_first
	FROM menu  m
	JOIN sales s
	ON m.product_id=s.product_id
	JOIN member mb
	ON s.customer_id=mb.customer_id)
	SELECT customer_id,product_name,purchased_first,order_date
	FROM ranked_products
	WHERE purchased_first = 1
	GROUP BY customer_id,purchased_first;
	
customer_id | product_name| purchased_first | order_date |
----------- |-----------  | -----------     | :--------: | 
A 	    | Sushi 	  | 	1 	    |2021-01-01  |
B 	    | Curry       |     1           | 2021-01-01 |




## What is the most purchased item on the menu and how many times was it purchased by all customers?

	SELECT men.product_name,
	COUNT(sal.product_id) AS times_purchased
	FROM dannys_diner.sales sal
	JOIN dannys_diner.menu men ON men.product_id = sal.product_id
	GROUP BY men.product_name
	ORDER BY times_purchased DESC
	LIMIT 1;
product_name | times_purchased |
-----------  |:-------------:  | 
 Ramen       |   8             |


## Which item was the most popular for each customer?

	SELECT men.product_name,
	COUNT(sal.product_id) AS most_popular_item
	FROM dannys_diner.sales sal
	JOIN dannys_diner.menu men ON men.product_id = sal.product_id
	GROUP BY men.product_name
	ORDER BY most_popular_item DESC
	LIMIT 1;
	
product_name | most_popular_item |
-----------  |:-------------:    | 
 Ramen       |   8               |

## Which item was purchased first by the customer after they became a member?

	WITH ranked_products AS (
	SELECT mb.customer_id,m.product_name,s.order_date,
	DENSE_RANK() OVER(PARTITION BY mb.customer_id ORDER BY s.order_date) AS purchased_first
	FROM menu  m
	JOIN sales s
	ON m.product_id=s.product_id
	JOIN member mb
	ON s.customer_id=mb.customer_id
	WHERE s.order_date >= mb.join_date)
	SELECT customer_id,product_name,purchased_first,order_date
	FROM ranked_products
	WHERE purchased_first = 1
	GROUP BY customer_id,purchased_first;
	
customer_id | product_name| purchased_first | order_date |
----------- |-----------  | -----------     | :--------: | 
A 	    | Sushi 	  | 	1 	    |2021-01-07  |
B 	    | Curry       |     1           | 2021-01-11 |




## Which item was purchased just before the customer became a member?

	WITH ranked_products AS (
	SELECT mb.customer_id,m.product_name,s.order_date,
	DENSE_RANK() OVER(PARTITION BY mb.customer_id ORDER BY s.order_date) AS purchased_first
	FROM menu  m
	JOIN sales s
	ON m.product_id=s.product_id
	JOIN member mb
	ON s.customer_id=mb.customer_id
	WHERE s.order_date < mb.join_date)
	SELECT customer_id,product_name,purchased_first,order_date
	FROM ranked_products
	WHERE purchased_first = 1
	GROUP BY customer_id,purchased_first;

customer_id | product_name| purchased_first | order_date |
----------- |-----------  | -----------     | :--------: | 
A 	    | Sushi 	  | 	1 	    |2021-01-01  |
B 	    | Curry       |     1           | 2021-01-01 |

## What is the total items and amount spent for each member before they became a member?

	WITH overalltotal_payments AS (
	SELECT mb.customer_id,m.product_name,m.price
	FROM menu  m
	JOIN sales s
	ON m.product_id=s.product_id
	JOIN member mb
	ON s.customer_id=mb.customer_id
	WHERE s.order_date < mb.join_date)
	SELECT customer_id,COUNT(product_name) AS total_items,SUM(price) as Amount_spent
	FROM overalltotal_payments
	GROUP BY customer_id
	ORDER BY Amount_spent;

customer_id |total_items |Amount_spent|
----------- |----------- | :--------: | 
A 	    |  25	 |	 25   |
B           |  3         | 40         |


## If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

	WITH points_per_purchase AS(
	SELECT sal.customer_id,men.product_name,men.price,
	    CASE
	WHEN men.product_name = 'sushi' THEN 2
		ELSE 1
	END AS multiplier,(men.price * 10) AS points
	FROM dannys_diner.sales sal
	JOIN dannys_diner.menu men ON men.product_id = sal.product_id)
	SELECT customer_id,SUM((points * multiplier)) AS total_points
	FROM points_per_purchase
	GROUP BY customer_id
	ORDER BY customer_id;

 customer_id |  total_points |
 ----------- | :--------:    |
    A        |    860        |
    B        |    940 	     |
    C 	     |  360          |


## In the first week after a customer joins the program (including their join date) they earn 2x points on all items,
 not just sushi - how many points do customer A and B have at the end of January?
 
	WITH points_per_purchase AS(
	SELECT sal.customer_id,mem.join_date,Month(mem.join_date) AS month,
	    CASE
	WHEN sal.order_date >= mem.join_date THEN 2
		ELSE 1
	END AS points
	FROM dannys_diner.sales sal
	JOIN dannys_diner.member mem ON mem.customer_id = sal.customer_id)
	SELECT customer_id,SUM(points) AS total_points
	FROM points_per_purchase
	WHERE month = 1
	GROUP BY customer_id
	ORDER BY customer_id;

customer_id |	total_points |
----------- | :------------: |
A	    |     10	     |
B	    |      9	     |


## Create a table adding the customer_id,order_date,product_name,price,and add a column to show customers who are members

	SELECT sal.customer_id,sal.order_date,men.product_name,men.price,
	CASE
	WHEN sal.order_date >= mem.join_date THEN 'Y'
	ELSE 'N'
	END AS member
	FROM sales sal
	LEFT JOIN member mem ON mem.customer_id = sal.customer_id
	JOIN menu men ON men.product_id = sal.product_id
	ORDER BY customer_id,order_date,product_name;

customer_id 	|	order_date	| product_name	| price	 | member |
-----------     | -----------           | -----------   | ------ | :----: |
A		|      2021-01-01	|  Curry	|  15	 |   N    |
A		| 	2021-01-01	|   Sushi	|  10	 |   N    |
A		|	 2021-01-07	|    Curry	|  15	 |    Y   |
A		|	 2021-01-10	|   Ramen	| 12	 |  Y     |
A		|	 2021-01-11	|   Ramen	| 12	 |    Y   |
A		|	 2021-01-11	|   Ramen	| 12	 |   Y    |
B		|	 2021-01-01	|    Curry	| 15	 |    N   |
B		|	 2021-01-02	|   Curry	| 15	 |     N  |
B		|	 2021-01-04	|  Sushi	| 10	 |    N   |
B		|	 2021-01-11	|   Sushi	| 10	 |    Y   |
B		|	 2021-01-16	|   Ramen	| 12	 |    Y   |
B		|	 2021-02-01	|   Ramen	| 12	 |    Y   |
C		|	 2021-01-01	|   Ramen	| 12	 |   N    |
C		|	 2021-01-01	|   Ramen	| 12	 |  N     |
C		|	 2021-01-07	|  Ramen	| 12	 |  N     |

## Create a table adding the customer_id,order_date,product_name,price,and add a column to show ranking values for the records when customers are not yet part of the loyalty program

	With customers_list AS (
	SELECT sal.customer_id,sal.order_date,men.product_name,men.price,
	CASE
	WHEN sal.order_date >= mem.join_date THEN 'Y'
	ELSE 'N'
	END AS member
	FROM sales sal
	LEFT JOIN member mem ON mem.customer_id = sal.customer_id
	JOIN menu men ON men.product_id = sal.product_id)
	SELECT *,
	CASE
	WHEN member = 'N' THEN 'Null'
	ELSE DENSE_RANK() OVER (ORDER BY product_name) 
	END AS ranking
	FROM customers_list
	ORDER BY customer_id,order_date,product_name
	
customer_id 	|	order_date	| product_name	| price	 | member | ranking |
-----------     | -----------           | -----------   | ------ |------  | :----:  |
A		|      2021-01-01	|  Curry	|  15	 |   N    | Null    |
A		| 	2021-01-01	|   Sushi	|  10	 |   N    | Null    |
A		|	 2021-01-07	|    Curry	|  15	 |    Y   | 1       |
A		|	 2021-01-10	|   Ramen	| 12	 |  Y     | 2       |
A		|	 2021-01-11	|   Ramen	| 12	 |    Y   | 2       |
A		|	 2021-01-11	|   Ramen	| 12	 |   Y    | 2       |
B		|	 2021-01-01	|    Curry	| 15	 |    N   | Null    |
B		|	 2021-01-02	|   Curry	| 15	 |     N  | Null    |
B		|	 2021-01-04	|  Sushi	| 10	 |    N   | Null    |
B		|	 2021-01-11	|   Sushi	| 10	 |    Y   | 3       |
B		|	 2021-01-16	|   Ramen	| 12	 |    Y   | 2       |
B		|	 2021-02-01	|   Ramen	| 12	 |    Y   | 2       |
C		|	 2021-01-01	|   Ramen	| 12	 |   N    | Null    |
C		|	 2021-01-01	|   Ramen	| 12	 |  N     | Null    |
C		|	 2021-01-07	|  Ramen	| 12	 |  N     | Null    |
