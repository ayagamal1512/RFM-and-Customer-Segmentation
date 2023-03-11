--Q1: what is the average price unit and total revenue per year?

SELECT DISTINCT (Years) , Round(avg(price*quantity)) AS Avg_price_unit,
                          sum(price*quantity) AS total_revenue
FROM
  (SELECT INVOICE,
          STOCKCODE,
          QUANTITY,
          INVOICEDATE,
          PRICE,
          CUSTOMER_ID,
          COUNTRY,
          to_char(to_date(invoicedate, 'mm/dd/yyyy hh24:mi'), 'yyyy') AS Years
   FROM tableretail)
GROUP BY Years
ORDER BY total_revenue DESC,
         avg_price_unit DESC;

--Q2: how many invoices we make per month?

SELECT distinct(months_of_year) ,
       count(invoice) over(PARTITION BY months_of_year order by months_of_year) AS no_of_invoices_per_day
FROM
  (SELECT INVOICE,
          STOCKCODE,
          QUANTITY,
          INVOICEDATE,
          PRICE,
          CUSTOMER_ID,
          COUNTRY,
          to_char(to_date(invoicedate, 'mm/dd/yyyy hh24:mi'), 'MM/YYYY') AS months_of_year
   FROM tableretail);

-- Q3: top 10 stock revenue per months of year
select* 
from
(WITH t1 AS
  (SELECT to_char(to_date(invoicedate, 'mm/dd/yyyy hh24:mi'), 'MM/YYYY') AS months_of_year,
          stockcode,
          sum(quantity*price) AS revenue
   FROM tableretail
   GROUP BY invoicedate,
            stockcode)
SELECT months_of_year,
       stockcode,
       revenue,
       rank () OVER (PARTITION BY months_of_year
                     ORDER BY revenue DESC) AS rank_
FROM t1
ORDER BY months_of_year DESC)
where Rank_ <= 10;

--Q4: what is the last highest quantity for which an order was sold?

SELECT invoice,
       quantity,
       LAG(quantity, 1) OVER (PARTITION BY invoice
                              ORDER BY quantity DESC) last_highest_quantity
FROM tableretail;

--Q5: How many days after the first purchase of a customer was the next purchase made?

SELECT invoicedate,
       customer_id,
       round(to_date(invoicedate, 'MM/DD/YYYY hh24:mi')- FIRST_VALUE(to_date(invoicedate, 'MM/DD/YYYY hh24:mi')) OVER (PARTITION BY customer_id
                                                                                                                       ORDER BY invoicedate)) next_order_gap
FROM tableretail
ORDER BY customer_id,
         next_order_gap

--Q6: Running total revenue and total transaction for each invoice in the 1st Q per day 

SELECT distinct(invoicedate),
       invoice,
       SUM (quantity*price) OVER (PARTITION BY invoice
                                  ORDER BY invoicedate) running_invoice_total_revenue,
                                  COUNT (invoice) over(PARTITION BY invoice
                                                                ORDER BY invoicedate) AS total__transaction
FROM tableretail
WHERE invoicedate BETWEEN '12/1/2010 15:38' AND '3/1/2011 14:53';

--segmentation
WITH segmentation  AS
(
SELECT customer_id,
       NTILE(5) OVER(ORDER BY Frequency) AS Frequency,
       NTILE(5) OVER(ORDER BY Recency) AS Recency,
       NTILE(5) OVER(ORDER BY Monetory) AS Monetary,
       NTILE(5) OVER(ORDER BY r_score) AS r_Score,
       NTILE(5) OVER(ORDER BY Fm_Score) AS Fm_Score       
FROM( SELECT customer_id,
             Last_Date,
             ROUND(TO_DATE('12/9/2011 12:30','MM/DD/YYYY HH24:MI')-Last_Date) Recency,
             Frequency,
             Monetory,
             ROUND(AVG(Frequency) over()) AS r_Score,
             ROUND(AVG(Monetory) over()) AS Fm_Score
             
       FROM (SELECT customer_id,
                    MAX(TO_DATE(invoicedate, 'MM/DD/YYYY HH24:MI')) AS Last_Date,
                    COUNT(invoice) AS Frequency,
                    SUM(price*quantity) AS Monetory
             FROM tableretail
             GROUP BY customer_id
             ORDER BY customer_id) ) 
)
SELECT customer_id,
            Frequency,
            Recency,
            Monetary,
             r_Score,
             Fm_Score,
            CASE WHEN Recency = 5 AND Monetary = 5 THEN 'Champions'
                     WHEN Recency = 4 AND Monetary = 5 THEN 'Champions'
                     WHEN Recency = 5 AND Monetary = 4 THEN 'Champions'
                     WHEN Recency = 5 AND Monetary = 2 THEN 'Potential Loyalists'
                     WHEN Recency = 4 AND Monetary = 2 THEN 'Potential Loyalists'
                     WHEN Recency = 4 AND Monetary = 3 THEN 'Potential Loyalists'
                     WHEN Recency = 3 AND Monetary = 3 THEN 'Potential Loyalists'
                     WHEN Recency = 5 AND Monetary = 3 THEN 'Loyal Customers'
                     WHEN Recency = 4 AND Monetary = 4 THEN 'Loyal Customers'
                     WHEN Recency = 3 AND Monetary = 5 THEN 'Loyal Customers'
                     WHEN Recency = 3 AND Monetary = 4 THEN 'Loyal Customers'
                     WHEN Recency = 5 AND Monetary = 1 THEN 'Recent Customers'
                     WHEN Recency = 4 AND Monetary = 1 THEN 'Promising'
                     WHEN Recency = 3 AND Monetary = 1 THEN 'Promising'
                     WHEN Recency = 3 AND Monetary = 2 THEN 'Customers Needing Attention'
                     WHEN Recency = 2 AND Monetary = 3 THEN 'Customers Needing Attention'
                     WHEN Recency = 2 AND Monetary = 2 THEN 'Customers Needing Attention'
                     WHEN Recency = 2 AND Monetary = 5 THEN 'At Risk'
                     WHEN Recency = 2 AND Monetary = 4 THEN 'At Risk'
                     WHEN Recency = 1 AND Monetary = 3 THEN 'At Risk'
                     WHEN Recency = 1 AND Monetary = 5 THEN 'Cannot Lose Them'
                     WHEN Recency = 1 AND Monetary = 4 THEN 'Cannot Lose Them'
                     WHEN Recency = 1 AND Monetary = 2 THEN 'Hibernating'
                     WHEN Recency = 1 AND Monetary = 1 THEN 'Lost'
            END segmentation
FROM segmentation
ORDER BY Recency DESC, Monetary DESC;
