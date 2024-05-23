/*

SQL_Target_Case_Study

*/

--Import the dataset and do usual exploratory analysis steps like checking the structure & characteristics of the dataset:

--1. Data type of all the columns in the “customers” table

SELECT  
        column_name,
        data_type 
FROM    saurav_dixit.INFORMATION_SCHEMA.COLUMNS 
WHERE   table_name = ‘customers’

        

--2. Get the time range between which the orders were placed.

SELECT   
        MIN(order_purchase_timestamp) AS first_order_placed,
        MAX(order_purchase_timestamp) AS last_order_placed 
FROM    `saurav_dixit.orders'



--3. Count the number of Cities and States in our dataset.

SELECT COUNT(DISTINCT geolocation_city) AS city_count,
       COUNT(DISTINCT geolocation_state) AS  state_count 
FROM `saurav_dixit.geolocation'



--In-depth Exploration:

--1. Is there a growing trend in the no. of orders placed over the past years?

SELECT EXTRACT(YEAR FROM order_purchase_timestamp) AS order_year, 
       COUNT(*) AS total_orders 
FROM `saurav_dixit.orders' 
GROUP BY 1 
ORDER BY 1


--2. Can we see some kind of monthly seasonality in terms of the no. of orders being placed ?

SELECT  FORMAT_DATE ('%B', order_purchase_timestamp) AS order_month, 
        COUNT(*) AS total_orders 
FROM `saurav_dixit.orders'
GROUP BY 1 
ORDER BY 1


--3. During what time of the day, do the Brazilian customers mostly place their orders? 
(Dawn, Morning, Afternoon or Night) o 0-6 hrs : Dawn o 7-12 hrs : Mornings o 13-18 hrs : Afternoon o 19-23 hrs : Night

SELECT  (CASE WHEN  EXTRACT(HOUR FROM order_purchase_timestamp) BETWEEN 0 AND 6 THEN "Dawn" 
              WHEN  EXTRACT(HOUR FROM order_purchase_timestamp) BETWEEN 7 AND 12 THEN "Morning" 
              WHEN  EXTRACT(HOUR FROM order_purchase_timestamp) BETWEEN 13 AND 18 THEN "Afternoon" ELSE "Night" END) AS time_of_day, 
        COUNT(*) AS orders_count 
FROM `saurav_dixit.orders' 
GROUP BY 1 
ORDER BY 1


--Evolution of E-commerce orders in the Brazil region:

--1. Get the month on month no. of orders placed in each state.

SELECT EXTRACT(YEAR FROM o.order_purchase_timestamp) AS year, 
       EXTRACT(MONTH FROM o.order_purchase_timestamp) AS month, 
       c.customer_state, 
       COUNT(*) AS order_count 
FROM `saurav_dixit.orders' o 
INNER JOIN `saurav_dixit.customers` c 
ON o.customer_id = c.customer_id 
GROUP BY  1,2,3 
ORDER BY 1


--2. How are the customers distributed across all the states?

SELECT customer_state, 
       COUNT(*) AS customer_distributed 
FROM `saurav_dixit.customers' 
GROUP BY 1 
ORDER BY 2 DESC


-- Impact on Economy: Analyze the money movement by e-commerce by looking at Order prices, freight and others.

--1. Get the % increase in the cost of orders from year 2017 to 2018 (include months between Jan to Aug only). 
     You can use the "payment_value" column in the payments table to get the cost of orders.

SELECT ROUND(SUM(CASE WHEN EXTRACT(YEAR FROM o.order_purchase_timestamp) = 2017 THEN p.payment_value ELSE 0 END), 3) AS cost_2017, 
       ROUND(SUM(CASE WHEN EXTRACT(YEAR FROM o.order_purchase_timestamp) = 2018 THEN p.payment_value ELSE 0 END), 3) AS cost_2018, 
       ROUND((SUM(CASE WHEN EXTRACT(YEAR FROM o.order_purchase_timestamp) = 2018 THEN p.payment_value END) - SUM(CASE WHEN EXTRACT(YEAR FRO M o.order_purchase_timestamp) = 2017 THEN p.payment_value 
       end))/sum(case when extract(year from o.order_purchase_timestamp) = 2017 then p.payment_value end)), 3) * 100 as percentage_increase  
FROM `saurav_dixit.orders` o 
INNER JOIN `saurav_dixit.payments` p 
ON o.order_id = p.order_id 
WHERE EXTRACTYEAR FROM o.order_purchase_timestamp) IN (2017, 2018) AND EXTRACT(MONTH FROM o.order_purchase_timestamp) BETWEEN  1 AND 8


--2. Calculate the Total & Average value of order price for each state.

SELECT s.seller_state,
       ROUND(SUM(oi.price), 2) AS total_price,
       ROUND(AVG(oi.price), 2) AS avg_price 
FROM `saurav_dixit.sellers` s 
INNER JOIN `saurav_dixit.order_items` oi 
ON s.seller_id = oi.seller_id 
GROUP BY s.seller_state 
ORDER BY 2 DESC


--3. Calculate the Total & Average value of order freight for each state.

SELECT s.seller_state, 
ROUND(SUM(oi.freight_value), 2) AS total_freight_value, 
ROUND(AVG(oi.freight_value), 2) AS avg_freight_value 
FROM `saurav_dixit.sellers` s 
INNER JOIN `saurav_dixit.order_items` oi 
ON s.seller_id = oi.seller_id 
GROUP BY s.seller_state 
ORDER BY 2 DESC


--Analysis based on sales, freight and delivery time.

--1. Find the no. of days taken to deliver each order from the order’s purchase date as delivery time. Also, calculate the difference (in days) between the estimated & actual delivery date of an order. 
     Do this in a single query. You can calculate the delivery time and the difference between the estimated & actual delivery date using the given formula: 
     * time_to_deliver = order_delivered_customer_date - order_purchase_timestamp 
     * diff_estimated_delivery = order_estimated_delivery_date - order_delivered_customer_date.

SELECT order_delivered_customer_date, 
       order_purchase_timestamp, 
       date_diff(order_delivered_customer_date , order_purchase_timestamp, day) AS time_to_deliver, 
       order_estimated_delivery_date, 
       order_delivered_customer_date, 
       date_diff(order_estimated_delivery_date, order_delivered_customer_date, day) AS diff_estimated_delivery 
FROM `saurav_dixit.orders'



--2. Find out the top 5 states with the highest & lowest average freight value.

(SELECT s.seller_state, ROUND(AVG(oi.freight_value), 2) AS avg_freight_value 
FROM `saurav_dixit.order_items` oi 
INNER JOIN `saurav_dixit.sellers` s 
ON oi.seller_id = s.seller_id 
GROUP BY s.seller_state order by avg_freight_value limit 5 ) 
UNION ALL
(SELECT s.seller_state, ROUND(AVG(oi.freight_value), 2) AS avg_freight_value 
FROM `saurav_dixit.order_items` oi 
INNER JOIN `saurav_dixit.sellers` s 
ON oi.seller_id = s.seller_id 
GROUP BY  s.seller_state 
ORDER BY avg_freight_value 
LIMIT 5 
OFFSET 18)


--3. Find out the top 5 states with the highest & lowest average delivery time.

(SELECT c.customer_state, 
        ROUND(AVG(DATE_DIFF(o.order_delivered_customer_date , o.order_purchase_timestamp, DAY)), 2) AS avg_delivery_time 
FROM `saurav_dixit.orders` o 
INNER JOIN `saurav_dixit.customers` c 
ON o.customer_id = c.customer_id 
GROUP BY c.customer_state 
ORDER BY avg_delivery_time 
LIMIT 5) 
UNION ALL
(SELECT c.customer_state, 
        ROUND(AVG(DATE_DIFF(o.order_delivered_customer_date , o.order_purchase_timestamp, DAY)), 2) AS avg_delivery_time 
FROM `saurav_dixit.orders` o 
INNER JOIN `saurav_dixit.customers` c 
ON o.customer_id = c.customer_id 
GROUP BY c.customer_state 
ORDER BY avg_delivery_time 
LIMIT 5 
OFFSET 18)


-- 4.Find out the top 5 states where the order delivery is really fast as compared to the estimate date of delivery.

SELECT c.customer_state, 
       ROUND(AVG(DATE_DIFF(o.order_delivered_carrier_date , o.order_estimated_delivery_date, DAY)), 2) AS avg_delivery_days 
FROM `saurav_dixit.orders` o 
INNER JOIN `saurav_dixit.customers` c 
ON o.customer_id = c.customer_id 
GROUP BY c.customer_state 
ORDER BY avg_delivery_days 
LIMIT 5


--Analysis based on the payments:

--1. Find the month on month no. of orders placed using different payment types.

SELECT EXTRACT(YEAR FROM o.order_purchase_timestamp) AS year, 
       EXTRACT(MONTH FROM o.order_purchase_timestamp) AS month, 
       p.payment_type, 
       COUNT(*) AS order_count 
FROM `saurav_dixit.orders` o 
INNER JOIN `saurav_dixit.payments` p
ON o.order_id = p.order_id 
GROUP BY 1,2,3 
ORDER BY 1,2


--2. Find the no. of orders placed on the basis of the payment installments that have been paid.

SELECT p.payment_type, 
       COUNT(DISTINCT o.order_id) AS no_of_orders 
FROM `saurav_dixit.orders` o 
INNER JOIN `saurav_dixit.payments` p 
ON o.order_id = p.order_id 
WHERE p.payment_installments > 0
GROUP BY p.payment_type
ORDER BY no_of_orders DESC




    



      
      
