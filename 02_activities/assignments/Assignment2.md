# Assignment 2: Design a Logical Model and Advanced SQL

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

#### Submission Parameters:
* Submission Due Date: `April 27, 2025`
* Weight: 70% of total grade
* The branch name for your repo should be: `assignment-two`
* What to submit for this assignment:
    * This markdown (Assignment2.md) with written responses in Section 1
    * Two Entity-Relationship Diagrams (preferably in a pdf, jpeg, png format).
    * One .sql file 
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sql/pulls/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [ ] Create a branch called `assignment-two`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via our Slack at `#cohort-6-help`. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.

***

## Section 1:
You can start this section following *session 1*, but you may want to wait until you feel comfortable wtih basic SQL query writing. 

Steps to complete this part of the assignment:
- Design a logical data model
- Duplicate the logical data model and add another table to it following the instructions
- Write, within this markdown file, an answer to Prompt 3


###  Design a Logical Model

#### Prompt 1
Design a logical model for a small bookstore. 📚

At the minimum it should have employee, order, sales, customer, and book entities (tables). Determine sensible column and table design based on what you know about these concepts. Keep it simple, but work out sensible relationships to keep tables reasonably sized. 

Additionally, include a date table. 

There are several tools online you can use, I'd recommend [Draw.io](https://www.drawio.com/) or [LucidChart](https://www.lucidchart.com/pages/).

**HINT:** You do not need to create any data for this prompt. This is a logical model (ERD) only. 

#### Prompt 2
We want to create employee shifts, splitting up the day into morning and evening. Add this to the ERD.

#### Prompt 3
The store wants to keep customer addresses. Propose two architectures for the CUSTOMER_ADDRESS table, one that will retain changes, and another that will overwrite. Which is type 1, which is type 2? 

**HINT:** search type 1 vs type 2 slowly changing dimensions. 

```
-- Overwriting the address, SCD TYPE 1
CREATE TABLE customer_address (
    customer_id INTEGER PRIMARY KEY,
    street TEXT,
    city TEXT,
    postal_code TEXT,
    country TEXT
);

-- Retaining changes, SCD TYPE 2
CREATE TABLE customer_address (
    address_id INTEGER PRIMARY KEY AUTOINCREMENT,
    customer_id INTEGER,
    street TEXT,
    city TEXT,
    postal_code TEXT,
    country TEXT,
    start_date TEXT,
    end_date TEXT,  -- can be NULL for the current address
);```

***

## Section 2:
You can start this section following *session 4*.
/* ASSIGNMENT 2 */
/* SECTION 2 */

-- COALESCE
/* 1. Our favourite manager wants a detailed long list of products, but is afraid of tables! 
We tell them, no problem! We can produce a list with all of the appropriate details. 

Using the following syntax you create our super cool and not at all needy manager a list:

SELECT 
product_name || ', ' || product_size|| ' (' || product_qty_type || ')'
FROM product

But wait! The product table has some bad data (a few NULL values). 
Find the NULLs and then using COALESCE, replace the NULL with a 
blank for the first problem, and 'unit' for the second problem. 

HINT: keep the syntax the same, but edited the correct components with the string. 
The `||` values concatenate the columns into strings. 
Edit the appropriate columns -- you're making two edits -- and the NULL rows will be fixed. 
All the other rows will remain the same.) */
SELECT 
   product_name || ', ' || COALESCE(product_size, '') || ' (' || COALESCE(product_qty_type, 'unit') || ')' AS detailed_list
FROM product;


--Windowed Functions
/* 1. Write a query that selects from the customer_purchases table and numbers each customer’s  
visits to the farmer’s market (labeling each market date with a different number). 
Each customer’s first visit is labeled 1, second visit is labeled 2, etc. 

You can either display all rows in the customer_purchases table, with the counter changing on
each new market date for each customer, or select only the unique market dates per customer 
(without purchase details) and number those visits. 
HINT: One of these approaches uses ROW_NUMBER() and one uses DENSE_RANK(). */
SELECT  
  customer_id,
  market_date,
  product_id,
  quantity,
  cost_to_customer_per_qty,
  vendor_id,
  transaction_time,
  DENSE_RANK() OVER (
    PARTITION BY customer_id 
    ORDER BY market_date
  ) AS visit_number
 FROM customer_purchases;


/* 2. Reverse the numbering of the query from a part so each customer’s most recent visit is labeled 1, 
then write another query that uses this one as a subquery (or temp table) and filters the results to 
only the customer’s most recent visit. */
SELECT  
  customer_id,
  market_date,
  product_id,
  quantity,
  cost_to_customer_per_qty,
  vendor_id,
  transaction_time,
  DENSE_RANK() OVER (
    PARTITION BY customer_id 
    ORDER BY market_date DESC
  ) AS visit_number
FROM customer_purchases;

SELECT  *
FROM (
						SELECT  
						  customer_id,
						  market_date,
						  product_id,
						  quantity,
						  cost_to_customer_per_qty,
						  vendor_id,
						  transaction_time,
						  DENSE_RANK() OVER (
							PARTITION BY customer_id 
							ORDER BY market_date DESC
						  ) AS visit_number
						FROM customer_purchases
)  x
WHERE x.visit_number = 1;

-- If only the date of the first visit is required
SELECT  DISTINCT
						customer_id,
						  market_date
FROM (
						SELECT  
						  customer_id,
						  market_date,
						  product_id,
						  quantity,
						  cost_to_customer_per_qty,
						  vendor_id,
						  transaction_time,
						  DENSE_RANK() OVER (
							PARTITION BY customer_id 
							ORDER BY market_date DESC
						  ) AS visit_number
						FROM customer_purchases
)  x
WHERE x.visit_number = 1;

/* 3. Using a COUNT() window function, include a value along with each row of the 
customer_purchases table that indicates how many different times that customer has purchased that product_id. */
-- I calculated two new columns, one for the total and another one for the cumulative
SELECT  *,
  COUNT(product_id) OVER (
    PARTITION BY customer_id, product_id 
    --ORDER BY product_id
  ) AS product_id_total_times_bought_by_customer,
  COUNT(*) OVER (
    PARTITION BY customer_id, product_id
    ORDER BY market_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS product_id_cumulative_times_bought_by_customer_C
FROM customer_purchases;


-- String manipulations
/* 1. Some product names in the product table have descriptions like "Jar" or "Organic". 
These are separated from the product name with a hyphen. 
Create a column using SUBSTR (and a couple of other commands) that captures these, but is otherwise NULL. 
Remove any trailing or leading whitespaces. Don't just use a case statement for each product! 

| product_name               | description |
|----------------------------|-------------|
| Habanero Peppers - Organic | Organic     |

Hint: you might need to use INSTR(product_name,'-') to find the hyphens. INSTR will help split the column. */

SELECT 
  product_name,
  --INSTR(product_name, '-') AS hyphen_pos,
  --SUBSTR(product_name, INSTR(product_name, '-')) AS from_hyphen_on,
  /*CASE 
    WHEN INSTR(product_name, '-') = 0 THEN NULL
    ELSE TRIM(SUBSTR(product_name, INSTR(product_name, '-') + 1))
  END AS description_2,*/
  
  TRIM(SUBSTR(product_name, NULLIF(INSTR(product_name, '-'), 0) + 1)) AS description
FROM product;


/* 2. Filter the query to show any product_size value that contain a number with REGEXP. */
SELECT  *
FROM (
					SELECT  *,
					  TRIM(SUBSTR(product_name, NULLIF(INSTR(product_name, '-'), 0) + 1)) AS description
					FROM product
)  x
WHERE x.product_size REGEXP '[0-9]';

-- UNION
/* 1. Using a UNION, write a query that displays the market dates with the highest and lowest total sales.

HINT: There are a possibly a few ways to do this query, but if you're struggling, try the following: 
1) Create a CTE/Temp Table to find sales values grouped dates; 
2) Create another CTE/Temp table with a rank windowed function on the previous query to create 
"best day" and "worst day"; 
3) Query the second temp table twice, once for the best day, once for the worst day, 
with a UNION binding them. */
DROP TABLE IF EXISTS temp.new_customer_purchases;

CREATE TABLE temp.new_customer_purchases AS
SELECT 
 market_date,
 SUM(quantity * cost_to_customer_per_qty) AS total_sales_per_day
FROM customer_purchases
GROUP BY market_date;

SELECT * FROM (
  SELECT *, RANK() OVER (ORDER BY total_sales_per_day DESC) AS rank
  FROM temp.new_customer_purchases
) WHERE rank = 1

UNION ALL

SELECT * FROM (
  SELECT *, RANK() OVER (ORDER BY total_sales_per_day ASC) AS rank
  FROM temp.new_customer_purchases
) WHERE rank = 1;


/* DROP TABLE IF EXISTS temp.new_new_customer_purchases_max;

CREATE TABLE temp.new_new_customer_purchases_max AS
SELECT *
FROM (
  SELECT *,
         RANK() OVER (ORDER BY total_sales_per_day DESC) AS tspd_max
  FROM temp.new_customer_purchases
) sub_max
WHERE tspd_max = 1;


DROP TABLE IF EXISTS temp.new_new_customer_purchases_min;

CREATE TABLE temp.new_new_customer_purchases_min AS
SELECT *
FROM (
  SELECT *,
         RANK() OVER (ORDER BY total_sales_per_day ASC) AS tspd_min
  FROM temp.new_customer_purchases
) sub_min
WHERE tspd_min = 1; */



/* SECTION 3 */

-- Cross Join
/*1. Suppose every vendor in the `vendor_inventory` table had 5 of each of their products to sell to **every** 
customer on record. How much money would each vendor make per product? 
Show this by vendor_name and product name, rather than using the IDs.

HINT: Be sure you select only relevant columns and rows. 
Remember, CROSS JOIN will explode your table rows, so CROSS JOIN should likely be a subquery. 
Think a bit about the row counts: how many distinct vendors, product names are there (x)?
How many customers are there (y). 
Before your final group by you should have the product of those two queries (x*y).  */

/* SELECT DISTINCT vendor_id  --there are only 3 vendors in vendor_inventory
FROM vendor_inventory;
SELECT DISTINCT product_id   --there are only 8 product ids  in vendor_inventory
FROM vendor_inventory;
SELECT DISTINCT customer_id  --there are 26 customers on record
FROM customer; */
	
WITH x AS (
  SELECT 
    vi.vendor_id,
    vi.product_id,
    vi.original_price,
    COUNT(c.customer_id) * vi.original_price * 5 AS total_money_made
  FROM customer c
  CROSS JOIN (
    SELECT DISTINCT vendor_id, product_id, original_price
    FROM vendor_inventory
  ) AS vi
  GROUP BY vi.vendor_id, vi.product_id, vi.original_price
)

SELECT 
  --x.vendor_id,
  --x.product_id,
  --x.original_price,
  x.total_money_made,
  v.vendor_name,
  p.product_name
FROM x
INNER JOIN vendor as v 
	ON x.vendor_id = v.vendor_id
INNER JOIN product as p 
	ON x.product_id = p.product_id;


-- INSERT
/*1.  Create a new table "product_units". 
This table will contain only products where the `product_qty_type = 'unit'`. 
It should use all of the columns from the product table, as well as a new column for the `CURRENT_TIMESTAMP`.  
Name the timestamp column `snapshot_timestamp`. */

/* Below is a version that I built using temp.table, just in case! After it, you can find the non-temp one */

DROP TABLE IF EXISTS temp.product_units;
CREATE TABLE temp.product_units AS  --if you don't want the table to be temp, don't write temp in any of the lines, the previous line should also be overlooked
SELECT 
  *, 
  CURRENT_TIMESTAMP AS snapshot_timestamp
FROM product
WHERE product_qty_type = 'unit';
	

/*2. Using `INSERT`, add a new row to the product_units table (with an updated timestamp). 
This can be any product you desire (e.g. add another record for Apple Pie). */
INSERT INTO temp.product_units (
  product_id, product_name, product_size, product_category_id, product_qty_type, snapshot_timestamp
)
VALUES (24, 'Dark_chocolate', '1 lb', 2, 'unit', CURRENT_TIMESTAMP);


-- DELETE
/* 1. Delete the older record for the whatever product you added. 

HINT: If you don't specify a WHERE clause, you are going to have a bad time.*/

WITH oldest_record AS (
  SELECT snapshot_timestamp
  FROM temp.product_units
  WHERE product_id = 24
  ORDER BY snapshot_timestamp ASC
  LIMIT 1
)
DELETE FROM temp.product_units
WHERE product_id = 24
  AND snapshot_timestamp = (SELECT snapshot_timestamp FROM oldest_record);

-- UPDATE
/* 1.We want to add the current_quantity to the product_units table.  (LATEST MARKET_DATE QUANTITY?)
First, add a new column, current_quantity to the table using the following syntax.

ALTER TABLE product_units
ADD current_quantity INT;

Then, using UPDATE, change the current_quantity equal to the last quantity value from the vendor_inventory details.

HINT: This one is pretty hard. 
First, determine how to get the "last" quantity per product.  
Second, coalesce null values to 0 (if you don't have null values, figure out how to rearrange your query so you do.) 
Third, SET current_quantity = (...your select statement...), remembering that WHERE can only accommodate one column. 
Finally, make sure you have a WHERE statement to update the right row, 
	you'll need to use product_units.product_id to refer to the correct row within the product_units table. 
When you have all of these components, you can run the update statement. */

ALTER TABLE temp.product_units
ADD current_quantity INT;
 
  WITH last_quantity_per_product AS (
  SELECT *
  FROM (
    SELECT *,
           ROW_NUMBER() OVER (
             PARTITION BY product_id 
             ORDER BY market_date DESC
           ) AS rn
    FROM vendor_inventory
  )
  WHERE rn = 1
), joined AS (
  SELECT  
    pu.product_id,
    COALESCE(lq.quantity, 0) AS no_null_current_quantity
  FROM temp.product_units pu
  LEFT JOIN last_quantity_per_product lq
    ON pu.product_id = lq.product_id
)

UPDATE temp.product_units pu
SET current_quantity = (
  SELECT j.no_null_current_quantity
  FROM joined j
  WHERE j.product_id = pu.product_id
); 

-- FINAL VERSION WITHOUT TEMP TABLE
-- INSERT
/*1.  Create a new table "product_units". 
This table will contain only products where the `product_qty_type = 'unit'`. 
It should use all of the columns from the product table, as well as a new column for the `CURRENT_TIMESTAMP`.  
Name the timestamp column `snapshot_timestamp`. */


CREATE TABLE product_units AS  
SELECT 
  *, 
  CURRENT_TIMESTAMP AS snapshot_timestamp
FROM product
WHERE product_qty_type = 'unit';
	

/*2. Using `INSERT`, add a new row to the product_units table (with an updated timestamp). 
This can be any product you desire (e.g. add another record for Apple Pie). */
INSERT INTO product_units (
  product_id, product_name, product_size, product_category_id, product_qty_type, snapshot_timestamp
)
VALUES (24, 'Dark_chocolate', '1 lb', 2, 'unit', CURRENT_TIMESTAMP);


-- DELETE
/* 1. Delete the older record for the whatever product you added. 

HINT: If you don't specify a WHERE clause, you are going to have a bad time.*/

--DELETE FROM product_units
--WHERE product_id = 24;
WITH oldest_record AS (
  SELECT snapshot_timestamp
  FROM product_units
  WHERE product_id = 24
  ORDER BY snapshot_timestamp ASC
  LIMIT 1
)
DELETE FROM product_units
WHERE product_id = 24
  AND snapshot_timestamp = (SELECT snapshot_timestamp FROM oldest_record);

-- UPDATE
/* 1.We want to add the current_quantity to the product_units table.  (LATEST MARKET_DATE QUANTITY?)
First, add a new column, current_quantity to the table using the following syntax.

ALTER TABLE product_units
ADD current_quantity INT;

Then, using UPDATE, change the current_quantity equal to the last quantity value from the vendor_inventory details.

HINT: This one is pretty hard. 
First, determine how to get the "last" quantity per product.  
Second, coalesce null values to 0 (if you don't have null values, figure out how to rearrange your query so you do.) 
Third, SET current_quantity = (...your select statement...), remembering that WHERE can only accommodate one column. 
Finally, make sure you have a WHERE statement to update the right row, 
	you'll need to use product_units.product_id to refer to the correct row within the product_units table. 
When you have all of these components, you can run the update statement. */

ALTER TABLE product_units
ADD current_quantity INT;
 
  WITH last_quantity_per_product AS (
  SELECT *
  FROM (
    SELECT *,
           ROW_NUMBER() OVER (
             PARTITION BY product_id 
             ORDER BY market_date DESC
           ) AS rn
    FROM vendor_inventory
  )
  WHERE rn = 1
), joined AS (
  SELECT  
    pu.product_id,
    COALESCE(lq.quantity, 0) AS no_null_current_quantity
  FROM product_units pu
  LEFT JOIN last_quantity_per_product lq
    ON pu.product_id = lq.product_id
)

UPDATE product_units pu
SET current_quantity = (
  SELECT j.no_null_current_quantity
  FROM joined j
  WHERE j.product_id = pu.product_id
);

