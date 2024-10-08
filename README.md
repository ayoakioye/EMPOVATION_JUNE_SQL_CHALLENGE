# INTRODUCTION
I recently participated in an SQL challenge with EMPOVATION. Throughout this challenge, I not only learnt new concepts but also connected with fellow participants, enhancing my SQL skills along the way.

The dataset provided is an electronics stores dataset conprisig of the customer table, the sales table , the stores table , the products table , the categories table and the exchanged rates table. The datasets provides the customer details, the stores details , the products details and the  company's business transaction between the year 2016 to 2021 across eight contries in Australia, Europe and North America.

# DATA DICTIONARY
SALES TABLE

Order Number:	Unique ID for each orders

Line Item:	Identifies individual products purchased as part of an order

Order Date:	Date the order was placed	

Delivery Date:	Date the order was delivered	

CustomerKey:	Unique key identifying which customer placed the order

StoreKey:	Unique key identifying which store processed the order

ProductKey:	Unique key identifying which product was purchased

Quantity:	Number of items purchased

Currency Code:	Currency used to process the order

CATEGORIES TABLE

CategoryKey:	Key to identify product categories

Category:	Product category name

SubcategoryKey:	Key to identify product subcategories

Subcategory:	Product subcategory name

CUSTOMER TABLE
Gender:	Customer gender

Name:	Customer full name

City:	Customer city

State Code:	Customer state (abbreviated)

State:	Customer state (full)

Zip Code:	Customer zip code

Country:  Customer country

Continent:	Customer continent

Birthday:	Customer date of birth

PRODUTS TABLE

ProductKey:	Primary key to identify products

Product Name:	Product name

Brand: Product brand

Color:	Product color

Unit Cost USD:	Cost to produce the product in USD

Unit Price USD:	Product list price in USD

SubcategoryKey:	Key to identify product subcategories

Subcategory:	Product subcategory name

STORES TABLE

StoreKey:	Primary key to identify stores

Country:	Store country

State:	Store state

Square Meters:	Store footprint in square meters

Open Date:	Store open date

EXCHANGE RATES TABLE
Date:	Date

Currency:	Currency code

Exchange:	Exchange rate compared to USD


# DATA CLEANING
####  REMOVING DUPLICATES; I CHEACKED FOR  IN EVERY TABLE, THE CATEGORIES TABLE WAS THE ONLY TABLE WITH DUPLICATESCustomerKey	Primary key to identify customers

![](DUPLICATES_IN_THE_CATEGORIES_TABLE.png)

### I CLEANED THE TABLE IN MICROSOFT EXCEL

![](REMOVING_DUPLICATES_CAT_EXCEL.png)

### RECHEACKED FOR DUPLICATES

![](DUPLICATES_GONE.png)

## EXPLORATORY DATA ANALYSIS
#### QUESTION 1:Write a query to count the total number of orders per customer order in desc
`SELECT C.NAME, COUNT(DISTINCT ORDER_NUMBER) AS TOTAL_ORDERS
FROM CUSTOMERS C
JOIN SALES S 
ON C.CUSTOMERKEY = S.CUSTOMERKEY
GROUP BY C.NAME
ORDER BY COUNT(DISTINCT ORDER_NUMBER) DESC;`

![](QUES_1.png)

#### QUESTION 2: Write a SQL Query to list the products sold in 2020
`SELECT DISTINCT P.PRODUCT_NAME 
FROM PRODUCTS P JOIN SALES S 
ON P.PRODUCTKEY = S.PRODUCTKEY 
WHERE YEAR(ORDER_DATE) = 2020;`

![](QUES_2.png)

#### QUESTION 3: Write a query to find all customer details from califonia
`SELECT CUSTOMERKEY, NAME, CITY, STATE 
FROM CUSTOMERS
WHERE STATE = 'CALIFORNIA';`

![](QUES_3.png)

#### QUESTION 4:Write a query to calculate the total sales quantity f0r products 2115
`SELECT P.PRODUCT_NAME, P.PRODUCTKEY, SUM(QUANTITY) AS TOTAL_SALES_QUANTITY 
FROM PRODUCTS P 
JOIN SALES S
ON P.PRODUCTKEY = S.PRODUCTKEY
WHERE  P.PRODUCTKEY = 2115
GROUP BY P.PRODUCTKEY, P.PRODUCT_NAME;`

![](QUES_4.png)

#### QUESTION 5:Write a query to retrive the top 5 stores with the most sales tranasactions
`SELECT TOP 5 ST.STOREKEY, COUNT(*) AS NUMBER_OF_TRANASACTIONS 
FROM STORES ST JOIN SALES S
ON ST.STOREKEY = S.STOREKEY
GROUP BY ST.STOREKEY
ORDER BY  NUMBER_OF_TRANASACTIONS DESC;`

![](QUES_5.png)

#### QUESTION 6: Write a query to find the average price of products in each category
`SELECT CATEGORY, AVG(UNIT_PRICE_USD) AS AVG_PRICE_OF_PRODUCTS_CATEGORY
FROM PRODUCTS
GROUP BY CATEGORY;`

![](QUES_6.png)

#### QUESTION 7:Write a query to count the numbers of orders placed by each gender
`SELECT COUNT(DISTINCT ORDER_NUMBER) AS NUMBER_OF_ORDERS, GENDER 
FROM SALES S JOIN CUSTOMERS C
ON S.CUSTOMERKEY = C.CUSTOMERKEY
GROUP BY GENDER
ORDER BY COUNT(DISTINCT ORDER_NUMBER) DESC;`

![](QUES_7.png)

#### QUESTION 8:SELECT P.PRODUCTKEY,P.PRODUCT_NAME
`FROM PRODUCTS P
LEFT JOIN SALES S 
ON P.PRODUCTKEY = S.PRODUCTKEY
WHERE S.PRODUCTKEY IS NULL;`

![](QUES_8.png)

#### QUESTION 9:Write a query to show the total amounts in usd, round to 2 decomal points for orders made in other currencies, using the exchange rate table to convert the prices
`SELECT S.ORDER_NUMBER,ROUND(SUM(S.QUANTITY * P.UNIT_PRICE_USD * COALESCE(E.EXCHANGE,1)),2) AS TOTAL_ORDER_IN_USD
FROM PRODUCTS  P 
JOIN SALES S
	ON P.PRODUCTKEY = S.PRODUCTKEY
LEFT JOIN EXCHANGE_RATES E 
	ON S.CURRENCY_CODE = E.CURRENCY AND S.ORDER_DATE = E.DATE
GROUP BY S.ORDER_NUMBER;`

![](QUES_9.png)

#### QUESTION 10:WRITE A QUERY TO ANALYZE WHETHER LARGER STORES IN TERMS OF SQUARE METERS ) HAVE HIGHER SALES VOLUMES.
`SELECT ST.STOREKEY,ST.SQUARE_METERS, SUM(QUANTITY)  AS SALES_VOLUME,AVG(QUANTITY) AS AVG_SALES_VOLUME
FROM STORES ST JOIN SALES S		
	ON ST.STOREKEY = S.STOREKEY JOIN PRODUCTS P
		ON P.PRODUCTKEY = S.PRODUCTKEY
		WHERE ST.STOREKEY != 0
GROUP BY ST.STOREKEY, SQUARE_METERS
ORDER BY ST.SQUARE_METERS DESC;`

![](QUES_10.png)

#### QUESTION 11:WRITE A QUERY TO SEGMENT CUSTOMERS INTO GROUPS BASED ON THIER PURCHASE BEHAVIORS
`SELECT C.CUSTOMERKEY, C.NAME, C.GENDER, C.STATE, COUNT(S.ORDER_NUMBER) AS NUMBER_OF_ORDER, SUM(S.QUANTITY*P.UNIT_PRICE_USD) AS TOTAL_SPEND,
CASE
	WHEN SUM( S.QUANTITY * P.UNIT_PRICE_USD) > 1000 THEN 'HIGH SPENDER'
	WHEN SUM( S.QUANTITY * P.UNIT_PRICE_USD) BETWEEN 500 AND 1000 THEN 'MEDUIM SPENDER'
	ELSE 'LOW SPENDER'
END AS SPEND_CATEGORY,
CASE 
	WHEN COUNT(S.ORDER_NUMBER) > 10 THEN 'ODOGWU'
	WHEN COUNT (S.ORDER_NUMBER) BETWEEN 5 AND 10  THEN 'MAKE I CHOP FIRST'
	ELSE 'GOD ABEG'
END AS BUY_CATEGORY
FROM CUSTOMERS C
JOIN SALES S ON C.CUSTOMERKEY = S.CUSTOMERKEY
JOIN PRODUCTS P ON S.PRODUCTKEY = P.PRODUCTKEY
GROUP BY  C.CUSTOMERKEY, C.NAME, C.GENDER, C.STATE
ORDER BY TOTAL_SPEND DESC;`

![](QUES_11.png)

#### QUESTION 12:WRITE A QUERY TO CALCULATE THE TOTAL SALES VOLUME FOR EACH STORE, THEN RANK STORES BASED ON THIER SALES VOLUME
`SELECT ST.STOREKEY,
SUM(S.QUANTITY) AS TOTAL_SALES_VOLUME,
RANK() OVER(ORDER BY  SUM(S.QUANTITY)DESC) AS SALES_RANK
FROM STORES ST
JOIN SALES S ON ST.STOREKEY = S.STOREKEY
JOIN PRODUCTS P ON P.PRODUCTKEY = S.PRODUCTKEY
GROUP BY  ST.STOREKEY
ORDER BY TOTAL_SALES_VOLUME DESC;`

![](QUES_12.png)

#### QUESTION 13:WRITE A QUERY TO RETRIEVE DAILY SALES VOLUME THEN CALCULATE A RUNNINMG TOTAL SALES OVER TIME , OREDERED BY DATE
`SELECT S.ORDER_DATE, SUM(S.QUANTITY) AS DAILY_TOTAL_SALES_VOLUME, 
SUM(SUM(S.QUANTITY)) OVER (ORDER BY S.ORDER_DATE) AS RUNNING_TOTAL_SALES
FROM SALES S
JOIN PRODUCTS P ON S.PRODUCTKEY = P.PRODUCTKEY
GROUP BY S.ORDER_DATE
ORDER BY S.ORDER_DATE  DESC;`

![](QUES_13.png)

#### QUESTION 14:WRITE A QUERY TO CALCULATE THE LIFETIME VALUE OF EACH CUSTOMER BASED ON THIER COUNTRY
`WITH CUSTOMER_LTV AS (
	SELECT C.CUSTOMERKEY,C.COUNTRY, SUM(S.QUANTITY * P.UNIT_PRICE_USD) AS TOTAL_SPEND
	FROM CUSTOMERS C
	JOIN SALES S ON C.CUSTOMERKEY = S.CUSTOMERKEY
	JOIN PRODUCTS P ON S.PRODUCTKEY = P.PRODUCTKEY
	GROUP BY C.CUSTOMERKEY, C.COUNTRY
	),
 COUNTRY_LTV AS(
	SELECT COUNTRY, AVG(TOTAL_SPEND) AS AVERAGE_LTV
	FROM CUSTOMER_LTV 
	GROUP BY COUNTRY
	)
 SELECT 
	COUNTRY,AVERAGE_LTV, RANK() OVER(ORDER BY AVERAGE_LTV DESC) AS LTVRANK
FROM COUNTRY_LTV
GROUP BY COUNTRY, AVERAGE_LTV
ORDER BY LTVRANK;`

![](QUES_14.png)

#### QUESTION 15:WRITE A QUERY TO CALCULATE THE LIFETIME VALUE OF EACH CUSTOMER BASED ON TOTAL AMOUNT THEY'VE SPENT
`SELECT C.CUSTOMERKEY, C.NAME, SUM(S.QUANTITY * P.UNIT_PRICE_USD) AS LIFE_TIME_VALUE
FROM CUSTOMERS C
JOIN SALES S ON C.CUSTOMERKEY = S.CUSTOMERKEY
JOIN PRODUCTS P ON S.PRODUCTKEY = P.PRODUCTKEY
GROUP BY C.CUSTOMERKEY, C.NAME
ORDER BY LIFE_TIME_VALUE DESC;`

![](QUES_15.png)

#### QUESTION 16:WRITE A QUERY TO CALCULATE THE TOTAL ANNUAL SALES PER PRODUCT CATEGORY FOR THE CURRENT YEAR AND THE PREVIOUS YEAR, AND THEN USE WINDOW FUNCTIONS TO CALCULATE THE YEAR_OVER_YEAR GROWTH PERCENTAGE
`WITH SALES_DATA AS (
	SELECT P.CATEGORYKEY, C.CATEGORY,YEAR(S.ORDER_DATE) AS ORDER_YEAR, 
			SUM(S.QUANTITY * P.UNIT_PRICE_USD) AS TOTAL_SALES
	FROM SALES S
	JOIN PRODUCTS P ON S.PRODUCTKEY = P.PRODUCTKEY
	JOIN CATEGORIES C ON P.CATEGORYKEY = C.CATEGORYKEY
	GROUP BY P.CATEGORYKEY,C.CATEGORY,YEAR (S.ORDER_DATE)
),
ANNUAL_SALES AS (
	SELECT CATEGORYKEY, CATEGORY,ORDER_YEAR,TOTAL_SALES,
			LAG(TOTAL_SALES,1) OVER (PARTITION BY CATEGORYKEY ORDER BY ORDER_YEAR) AS PREVIOUS_YEAR_SALES
	FROM SALES_DATA
)
SELECT CATEGORYKEY,CATEGORY,ORDER_YEAR,TOTAL_SALES,PREVIOUS_YEAR_SALES,
       CASE
			WHEN PREVIOUS_YEAR_SALES IS NULL THEN NULL
			ELSE ((TOTAL_SALES  - PREVIOUS_YEAR_SALES) / PREVIOUS_YEAR_SALES) * 100
		END AS YEAR_OVER_YEAR_GROWTH_PERCENTAGE
FROM ANNUAL_SALES
WHERE ORDER_YEAR IN (2020, 2021)
ORDER BY CATEGORYKEY, ORDER_YEAR;`

![](QUES_16.png)

### QUESTION 17:WRITE A SQL QUERY TO FIND EACH CUSTOMER'S PURCHASE RANK WITHIN THE STORE THEY BOUGHT FROM, BASED ON THE TOTAL PRICE OF THE ORDER(QUANTITY * UNIT PRICE)
`WITH CUSTOMERTOTALSALES AS (
	SELECT S.CUSTOMERKEY, S.STOREKEY, SUM(S.QUANTITY * P.UNIT_PRICE_USD) AS TOTAL_PURCHASE_PRICE, 
		RANK() OVER (PARTITION BY S.CUSTOMERKEY,S.STOREKEY ORDER BY SUM(S.QUANTITY * P.UNIT_PRICE_USD) DESC) AS PURCHASE_RANK
	FROM SALES S INNER JOIN PRODUCTS P ON S.PRODUCTKEY = P.PRODUCTKEY
	GROUP BY S.CUSTOMERKEY,S.STOREKEY
	)
   SELECT C.CUSTOMERKEY, C.NAME AS CUSTOMER_NAME, S.STOREKEY, S.COUNTRY AS STORE_COUNTRY,CTS.TOTAL_PURCHASE_PRICE,CTS.PURCHASE_RANK
FROM
  CUSTOMERTOTALSALES CTS
    INNER JOIN CUSTOMERS C ON CTS.CUSTOMERKEY = C.CUSTOMERKEY
    INNER JOIN STORES S ON CTS.STOREKEY = S.STOREKEY
ORDER BY
     CTS.CUSTOMERKEY, CTS.PURCHASE_RANK;`

![](QUES_17.png)
### QUESTION 18:CUSTOMER RETENSION ANALYSIS
`WITH FIRSTPURCHASEDATE AS (
    SELECT CUSTOMERKEY, MIN(ORDER_DATE) AS FIRST_PURCHASE_DATES
    FROM SALES
	GROUP BY CUSTOMERKEY
),
REPEAT_PURCHASES AS (
	 SELECT S.CUSTOMERKEY,COUNT(*) AS REPEAT_PURCHASE_COUNT
     FROM SALES S
			JOIN  FIRSTPURCHASEDATE FPD ON S.CUSTOMERKEY = FPD.CUSTOMERKEY
	 WHERE S.ORDER_DATE BETWEEN FPD.FIRST_PURCHASE_DATES AND DATEADD(DAY, 90, FPD.FIRST_PURCHASE_DATES)
	 GROUP BY S.CUSTOMERKEY
)
SELECT C.GENDER,  DATEDIFF(YEAR, C.BIRTHDAY, GETDATE()) AS AGE, C.CITY, C.STATE_CODE, C.COUNTRY AS CUSTOMER_COUNTRY,
		COUNT(*) AS TOTAL_CUSTOMER, COALESCE( RP.REPEAT_PURCHASE_COUNT, 0) AS RETAINED_CUSTOMERS,
		CASE 
			WHEN COUNT(*) > 0 THEN (CAST(COALESCE(RP.REPEATE_PURCHASE_COUNT, 0) AS FLOAT)/ COUNT(*)) * 100
			ELSE 0
		END AS RETENTION_RATE
FROM
  CUSTOMERS C
LEFT JOIN REPEAT_PURCHASES AS RP 
 ON C.CUSTOMERKEY = RP.CUSTOMERKEY
GROUP BY
    C.GENDER,DATEDIFF(YEAR, C.BIRTHDAY, GETDATE()),C.CITY,C.STATE_CODE,C.COUNTRY, COALESCE(RP.REPEAT_PURCHASE_COUNT, 0)
ORDER BY
   C.GENDER,DATEDIFF(YEAR, C.BIRTHDAY, GETDATE()),C.CITY,C.STATE_CODE,C.COUNTRY;`
   
![](QUES_18.png)

   # INSIGHTS
   1) Store key 0 achieved the highest sales volume, totaling $13,165 across transactions.
  2) Many unsold products belonged to the Home Appliance category, largely because of the category's high product prices.
   3) The business saw a higher number of male buyers compared to female buyers.
   4) The online store achieved its highest sales volume.
   5) Gaspare Trevisan (Male), from Salemon, has the highest number of orders: 36 orders.
   6) Matthew Flemming (Male), from California, has the highest total spend of $61,871.70.
   7) The business recorded its highest daily sales on December 21, 2019

# RECOMMENDATIONS
1)Sufficient attention should be devoted to the online store, including customer satisfaction, app graphics, and seamless transactions.

2)More men's clothing should be sold compared to women's.

3)Prices for products in the home appliances category should be reviewed and adjusted so that people from low-income communities and those in low-income jobs can afford more home appliances.

 # THANK YOUUUUU

