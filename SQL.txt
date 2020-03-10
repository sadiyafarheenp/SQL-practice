/*
Selecting columns 'occured_at','account_id','channel' from the table
'web_events'. We are displaying only 15 rows
*/

SELECT occurred_at,account_id,channel
  FROM web_events
  LIMIT 15;


/*
ORDER BY
Query to return the top 5 orders in terms of largest total_amt_usd.
Including the columns: the id, account_id, and total_amt_usd.
*/
SELECT id, account_id, total_amt_usd
 FROM orders
ORDER BY total_amt_usd DESC
LIMIT 5;
/*
we can ORDER BY more than one column at a time.
Query that displays the order ID, account ID, and total dollar amount for all
the orders, sorted first by the account ID (in ascending order),
and then by the total dollar amount (in descending order)
*/
SELECT id, account_id,total_amt_usd
 FROM orders
ORDER BY account_id, total_amt_usd DESC;




/*WHERE  statement for numeric data:
Pulls the first 10 rows and all columns from the orders table
that have a total_amt_usd less than 500
*/
SELECT *
  FROM  orders
WHERE total_amt_usd <= 500
LIMIT 10;
/*WHERE statement for non-numeric data.
Text data  needs to be in single colons.
Filter the accounts table to include the company name, website, and
the primary point of contact (primary_poc)
just for the Exxon Mobil company in the accounts table.
*/
SELECT name, website, primary_poc
FROM accounts
WHERE name = 'Exxon Mobil';



/*Derived Columns:Creating a new column that is a combination of existing columns is known as a derived column (or "calculated" or "computed" column).
Usually you want to give a name, or "alias," to your new column using the AS keyword.
Create a column that divides the standard_amt_usd by the standard_qty to find the unit price
for standard paper for each orderand name it as unit_price. Limit the results to the first 10 orders,
and include the id and account_id fields
*/
SELECT id,account_id,(standard_amt_usd/standard_qty) AS unit_price
 FROM orders
LIMIT 10;


/*LOGIC OPERATORS- LIKE
Use the accounts table to find
All the companies whose names start with 'C'
*/
SELECT name
FROM accounts
WHERE name LIKE 'C%';
/*Use the accounts table to find
All companies whose names contain the string 'one' somewhere in the name
*/
SELECT name
FROM accounts
WHERE name LIKE '%one%';
/*Use the accounts table to find
All companies whose names end with 's'
*/
SELECT name
FROM accounts
WHERE name LIKE '%s';

/*IN Clause:
Use the web_events table to find all information regarding individuals who were
contacted via the channel of organic or adwords.
*/
SELECT *
 FROM web_events
WHERE channel IN ('organic','adwords');


/*NOT clause:
Use the accounts table to find:
All the companies whose names do not start with 'C'
*/
SELECT name
FROM accounts
WHERE name NOT LIKE 'C%';

/*
Use the web_events table to find all information regarding individuals
who were contacted via any method except using organic or adwords methods
*/
SELECT *
FROM web_events
WHERE channel NOT IN ('organic', 'adwords');


/*AND and BETWEEN clause:
Use the web_events table to find all information regarding individuals who were
 contacted via the organic or adwords channels, and started their account at
 any point in 2016, sorted from newest to oldest.
 */
SELECT *
FROM web_events
WHERE channel IN ('organic','adwords')
AND occurred_at BETWEEN '2016-1-1' AND '2016-12-31'
ORDER BY occurred_at DESC;


/*OR Clause:
Find all the company names that start with a 'C' or 'W',
and the primary contact contains 'ana' or 'Ana', but it doesn't contain 'eana'.
*/
SELECT *
    FROM accounts
    WHERE (name LIKE 'C%' OR name LIKE 'W%')
    AND (primary_poc LIKE '%ana%' OR primary_poc 			LIKE'%Ana%')
    AND  primary_poc NOT LIKE '%eana%';



/*JOIN clause:
Try pulling standard_qty, gloss_qty, and poster_qty from the orders table,
and the website and the primary_poc from the accounts table.
*/
SELECT orders.standard_qty , orders.poster_qty, accounts.website,accounts.primary_poc
 FROM orders
 JOIN accounts
 ON orders.account_id = accounts.id;
/*
Provide a table that provides the region for each sales_rep along with their
associated accounts. Your final table should include three columns:
the region name, the sales rep name, and the account name.
Sort the accounts alphabetically (A-Z) according to account name.
*/
SELECT a.name account , s.name sales_rep,  r.name region
 FROM sales_reps s
 JOIN accounts a
 ON  a.sales_rep_id = s.id
 JOIN region r
 ON s.region_id = r.id
 ORDER BY account;
/*Provide the name for each region for every order, as well as the account name
and the unit price they paid (total_amt_usd/total) for the order. Your final table
should have 3 columns: region name, account name, and unit price.
A few accounts have 0 for total,
so I divided by (total + 0.01) to assure not dividing by zero.
*/
SELECT r.name region, a.name account,
       o.total_amt_usd/(o.total + 0.01) unit_price
FROM region r
JOIN sales_reps s
ON s.region_id = r.id
JOIN accounts a
ON a.sales_rep_id = s.id
JOIN orders o
ON o.account_id = a.id;

/*SELECT DISTINCT clause:
What are the different channels used by account id 1001? Your final table
should have only 2 columns: account name and the different channels.
*/
SELECT DISTINCT a.name, w.channel
FROM accounts a
JOIN web_events w
ON a.id = w.account_id
WHERE a.id = '1001';


/*SQL Aggregation:AVG,MIN,MAX:
When was the earliest and recent order ever placed? You only need to return the date.
*/
SELECT MIN(occurred_at)
FROM orders;
SELECT MAX(occurred_at)
FROM orders;
/*Find the mean (AVERAGE) amount spent per order on each paper type,
 as well as the mean amount of each paper type purchased per order.
 */
 SELECT AVG(standard_qty) mean_standard, AVG(gloss_qty) mean_gloss,
           AVG(poster_qty) mean_poster, AVG(standard_amt_usd) mean_standard_usd,
           AVG(gloss_amt_usd) mean_gloss_usd, AVG(poster_amt_usd) mean_poster_usd
FROM orders;
/*Finding Median value*/
SELECT *
FROM (SELECT total_amt_usd
      FROM orders
      ORDER BY total_amt_usd
      LIMIT 3457) AS Table1
ORDER BY total_amt_usd DESC
LIMIT 2;


/*GROUP BY Clause:
Find the total sales in usd for each account. You should include two columns -
the total sales for each company's orders in usd and the company name.
*/
SELECT a.name, SUM(total_amt_usd) total_sales
FROM orders o
JOIN accounts a
ON a.id = o.account_id
GROUP BY a.name;
/*Determine the number of times a particular channel was used in the web_events
 table for each sales rep. Your final table should have three columns - the name
 of the sales rep, the channel,and the number of occurrences. Order your table
 with the highest number of occurrences first.
 */
SELECT s.name AS sales_man, w.channel AS channel,COUNT(w.channel) AS channel_count
FROM sales_reps s
JOIN accounts a
ON a.sales_rep_id = s.id
JOIN web_events w
ON w.account_id = a.id
GROUP BY channel,sales_man
ORDER BY channel_count DESC;



/*HAVING Clause:
How many of the sales reps have more than 5 accounts that they manage
*/
SELECT s.id, s.name, COUNT(*) num_accounts
FROM accounts a
JOIN sales_reps s
ON s.id = a.sales_rep_id
GROUP BY s.id, s.name
HAVING COUNT(*) > 5
ORDER BY num_accounts;



/*CASE Clause:
We would like to identify top performing sales reps, which are sales reps
associated with more than 200 orders. Create a table with the sales rep name,
the total number of orders, and a column with top or not depending on if they
have more than 200 orders. Place the top sales people first in your final table.
*/
SELECT s.name sales_man, COUNT(*) order_count,
	CASE
    WHEN COUNT(*) > 200 THEN 'Top'
    ELSE 'Not'
    END AS Performance
 FROM sales_reps s
 JOIN accounts a
 ON a.sales_rep_id = s.id
 JOIN orders o
 ON o.account_id =a.id
 GROUP BY s.name
 ORDER BY 2 DESC;

 /*The previous didn't account for the middle, nor the dollar amount associated with the sales.
Management decides they want to see these characteristics represented as well. We would like to
identify top performing sales reps, which are sales reps associated with more than 200 orders
or more than 750000 in total sales. The middle group has any rep with more than 150 orders or
500000 in sales. Create a table with the sales rep name, the total number of orders, total
 sales across all orders, and a column with top, middle, or low depending on this criteria.
 Place the top sales people based on dollar amount of sales first in your final table.
*/
SELECT s.name sales_man, COUNT(*) order_count, SUM(o.total_amt_usd) total_sales,
	CASE
    WHEN COUNT(*) > 200 OR SUM(o.total_amt_usd) > 		750000 THEN 'Top'
    WHEN COUNT(*) > 150 OR SUM(o.total_amt_usd) > 		500000 THEN 'middle'
    ELSE 'low'
    END AS Performance
 FROM sales_reps s
 JOIN accounts a
 ON a.sales_rep_id = s.id
 JOIN orders o
 ON o.account_id =a.id
 GROUP BY s.name
 ORDER BY total_sales DESC
 ;


/*DATE
Which year did Parch & Posey have the greatest sales in terms of total
 number of orders?
 */
 SELECT DATE_PART('year', occurred_at) ord_year,  COUNT(*) total_sales
FROM orders
GROUP BY 1
ORDER BY 2 DESC;
/*
In which month of which year did Walmart spend the most on gloss paper in
terms of dollars?
*/
SELECT DATE_TRUNC('month', o.occurred_at) ord_date, SUM(o.gloss_amt_usd) tot_spent
FROM orders o
JOIN accounts a
ON a.id = o.account_id
WHERE a.name = 'Walmart'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;



/*SUB_QUERY:
to get a table that shows the average number of events a day for each channel.
*/
SELECT channel, AVG(events) AS average_events
FROM (SELECT DATE_TRUNC('day',occurred_at) AS day,
             channel, COUNT(*) as events
      FROM web_events
      GROUP BY 1,2) sub
GROUP BY channel
ORDER BY 2 DESC;
/*Pulling all orders on each type of papers, which was made on the month when
the first order was made*/
SELECT AVG(standard_qty) AS standard,
		AVG(gloss_qty) AS gloss,
        AVG(poster_qty) AS poster,
        SUM(total_Amt_usd)
FROM orders
WHERE DATE_TRUNC('month',occurred_at)=
	(SELECT DATE_TRUNC('month',min(occurred_at))AS 		min_month
	FROM orders)
  ;



/*WITH clause:
Essentially a WITH statement performs the same task as a Subquery.
What is the lifetime average amount spent in terms of total_amt_usd for the
 top 10 total spending accounts?
 */
 WITH top_ten AS
	(SELECT a.name account,
		SUM(o.total_amt_usd) total_amt
 	FROM accounts a
 	JOIN orders o
 	ON o.account_id = a.id
	GROUP BY account
	ORDER BY total_amt DESC
	LIMIT 10)


SELECT account,AVG(total_amt)
FROM top_ten
GROUP BY account
ORDER BY 2 DESC;



/*DATA CLEANING:
LEFT,RIGHT,LENGTH
In the accounts table, there is a column holding the website for each company. The last three
digits specify what type of web address they are using. A list of extensions (and pricing) is
provided here. Pull these extensions and provide how many of each website type exist in the
accounts table.
*/
SELECT
RIGHT (website,3) AS domain,COUNT(*)
FROM accounts
GROUP BY 1
ORDER BY 2 DESC
;
/*
Use the accounts table and a CASE statement to create two groups: one group of company names
that start with a number and a second group of those company names that start with a letter.
What proportion of company names start with a letter?
*/
SELECT SUM(num) nums, SUM(letter) letters
FROM (SELECT name, CASE WHEN LEFT(UPPER(name), 1) IN ('0','1','2','3','4','5','6','7','8','9')
                       THEN 1 ELSE 0 END AS num,
         CASE WHEN LEFT(UPPER(name), 1) IN ('0','1','2','3','4','5','6','7','8','9')
                       THEN 0 ELSE 1 END AS letter
      FROM accounts) t1;


/*STRPOS,LENGTH:
For every rep name in the sales_reps table, provide first and last name columns.
*/
SELECT name,
LEFT(name,STRPOS(name,' ')-1) Firstname,
RIGHT(name, LENGTH(name)-STRPOS(name,' ')) Lastname
FROM sales_reps;


/*CONCAT Clause:
Each company in the accounts table wants to create an email address for each primary_poc.
The email address should be the first name of the primary_poc . last name primary_poc @
company name .com.
*/
WITH t1 AS
(SELECT name,
LEFT(primary_poc,STRPOS(primary_poc,' ')-1) first_name,
RIGHT(primary_poc,LENGTH(primary_poc)-STRPOS(primary_poc,' ')) last_name
FROM accounts)

SELECT first_name, last_name,
CONCAT(first_name,'.',last_name,'@',name,'.com')
 AS email
FROM t1
;

/*REPLACE:
Removing space
you can create an email address that will work by removing all of the spaces
in the account name, but otherwise your solution should be just as above:
*/
WITH t1 AS
(SELECT name,
LEFT(primary_poc,STRPOS(primary_poc,' ')-1) first_name,
RIGHT(primary_poc,LENGTH(primary_poc)-STRPOS(primary_poc,' ')) last_name
FROM accounts)

SELECT first_name, last_name,
CONCAT(first_name,'.',last_name,'@',REPLACE(name,' ',''),'.com')
 AS email
FROM t1
;

/*CAST Clause:*/
SELECT date,
(SUBSTR(date,7,4) ||'-'||SUBSTR(date,1,2) ||'-'|| SUBSTR(date,4,2)
  ) :: DATE new_date
FROM sf_crime_data
LIMIT 10;
