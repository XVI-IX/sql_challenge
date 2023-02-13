# SQL CHALLENGE QUERIES

1. **Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.**

    Query:

    ```sql
    SELECT DISTINCT(market)
    FROM dim_customer
    WHERE customer = 'Atliq Exclusive'
    AND region = 'APAC';
    ```

    **Results**
    | market      |
    |-------------|
    | India       |
    | Indonesia   |
    | Japan       |
    | Philiphines |
    | South Korea |
    | Australia   |
    | Newzealand  |
    | Bangladesh  |

2. **What is the percentage of unique product increase in 2021 vs. 2020?**

    **Query:**

    ```sql
    WITH count_2020 AS (
        SELECT count(distinct(product_code)) as count_2020, fiscal_year
        from fact_sales_monthly
        where fiscal_year = '2020'),
        count_2021 AS (
        SELECT count(distinct(product_code)) as count_2021, fiscal_year
        from fact_sales_monthly
        where fiscal_year = '2021')
        SELECT a.count_2020 as unique_products_2020,
        b.count_2021 as unique_products_2021,
    (((b.count_2021 - a.count_2020) / a.count_2020) * 100) as percentage_chg
        FROM count_2020 as a
        cross join count_2021 as b;
            ```

**Results**

    | unique_products_2020 | unique_products_2021 | percentage_chg |
    |----------------------|----------------------|----------------|
    |                  245 |                  334 |        36.3265 |

3. **Provide a report with all the unique product counts for each segment and sort them in descending order of product counts.**

    **Query:**

    ```sql
    SELECT COUNT(DISTINCT(product_code)) as product_code, segment
    FROM dim_product 
    GROUP BY segment
    ORDER BY 1 DESC;
    ```

    **Results**

    | product_code | segment     |
    |--------------|-------------|
    |          129 | Notebook    |
    |          116 | Accessories |
    |           84 | Peripherals |
    |           32 | Desktop     |
    |           27 | Storage     |
    |            9 | Networking  |

4. **Follow-up: Which segment had the most increase in unique products in 2021 vs 2020?**

    Query:

    ```sql
    WITH count_2020 AS (
    SELECT COUNT(DISTINCT(a.product_code)) as product_2020,
    a.segment as segment
    FROM dim_product as a
    LEFT JOIN fact_sales_monthly as b
    ON a.product_code = b.product_code
    WHERE b.fiscal_year = '2020'
    GROUP BY segment
    ORDER BY 1 DESC),
    count_2021 AS(
    SELECT COUNT(DISTINCT(a.product_code)) as product_2021,
    a.segment as segment
    FROM dim_product as a
    LEFT JOIN fact_sales_monthly as b
    ON a.product_code = b.product_code
    WHERE b.fiscal_year = '2021'
    GROUP BY a.segment
    ORDER BY 1 DESC)

    SELECT DISTINCT(a.segment), a.product_2020, b.product_2021
    FROM count_2020 as a
    JOIN count_2021 as b;
    ```

    **Results**

    | segment     | product_2020 | product_2021 |
    |-------------|--------------|--------------|
    | Networking  |            6 |          108 |
    | Desktop     |            7 |          108 |
    | Storage     |           12 |          108 |
    | Peripherals |           59 |          108 |
    | Accessories |           69 |          108 |
    | Notebook    |           92 |          108 |
    | Networking  |            6 |          103 |
    | Desktop     |            7 |          103 |
    | Storage     |           12 |          103 |
    | Peripherals |           59 |          103 |
    | Accessories |           69 |          103 |
    | Notebook    |           92 |          103 |
    | Networking  |            6 |           75 |
    | Desktop     |            7 |           75 |
    | Storage     |           12 |           75 |
    | Peripherals |           59 |           75 |
    | Accessories |           69 |           75 |
    | Notebook    |           92 |           75 |
    | Networking  |            6 |           22 |
    | Desktop     |            7 |           22 |
    | Storage     |           12 |           22 |
    | Peripherals |           59 |           22 |
    | Accessories |           69 |           22 |
    | Notebook    |           92 |           22 |
    | Networking  |            6 |           17 |
    | Desktop     |            7 |           17 |
    | Storage     |           12 |           17 |
    | Peripherals |           59 |           17 |
    | Accessories |           69 |           17 |
    | Notebook    |           92 |           17 |
    | Networking  |            6 |            9 |
    | Desktop     |            7 |            9 |
    | Storage     |           12 |            9 |
    | Peripherals |           59 |            9 |
    | Accessories |           69 |            9 |
    | Notebook    |           92 |            9 |

5. **Get the products that have the highest and lowest manufacturing costs.**

    **Query:**

    **Step 1: highest manufacturing costs**

    ```sql
    SELECT dim_product.product_code, product, manufacturing_cost
    FROM  dim_product
    INNER JOIN fact_manufacturing_cost
    ON 
    dim_product.product_code = fact_manufacturing_cost.product_code
    ORDER BY manufacturing_cost DESC
    LIMIT 1;
    ```

    **Results**

    | product_code | product              | manufacturing_cost |
    |--------------|----------------------|--------------------|
    | A6120110206  | AQ HOME Allin1 Gen 2 |           240.5364 |
    

    **Step 2: lowest manufacturing costs**

    ```sql
    SELECT dim_product.product_code, product, manufacturing_cost
    FROM  dim_product
    INNER JOIN fact_manufacturing_cost
    ON 
    dim_product.product_code = fact_manufacturing_cost.product_code
    ORDER BY manufacturing_cost
    LIMIT 1;
    ```

    **Results**

    | product_code | product               | manufacturing_cost |
    |--------------|-----------------------|--------------------|
    | A2118150101  | AQ Master wired x1 Ms |             0.8920 |

6. **Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market.**

    **Query:**

    ```sql
            SELECT customer, AVG(pre_invoice_discount_pct) AS average_discount_percentage
            FROM dim_customer AS d
            INNER JOIN fact_pre_invoice_deductions AS f
            ON d.customer_code = f.customer_code
            WHERE f.fiscal_year = '2021' AND d.market = 'India'
            GROUP BY d.customer
            ORDER BY 2 DESC
            LIMIT 5;
    ```

    | customer    | average_discount_percentage |
    |-------------|-----------------------------|
    | Flipkart    |                  0.30830000 |
    | Viveks      |                  0.30380000 |
    | Ezone       |                  0.30280000 |
    | Croma       |                  0.30250000 |
    | Vijay Sales |                  0.27530000 |

7. **Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month.**

    **Query:**

    ```sql
    SELECT YEAR(s.date) as year, MONTHNAME(s.date) AS month,
    SUM(s.sold_quantity * g.gross_price) AS gross_amount
    FROM fact_sales_monthly AS s
    INNER JOIN dim_customer AS c
    ON s.customer_code = c.customer_code
    INNER JOIN fact_gross_price AS g
    ON s.product_code = g.product_code
    WHERE c.customer = "Atliq Exclusive"
    GROUP BY year, month;
    ```

    **Results**

    | year | month     | gross_amount  |
    |------|-----------|---------------|
    | 2019 | September |  9092670.3392 |
    | 2019 | October   | 10378637.5961 |
    | 2019 | November  | 15231894.9669 |
    | 2019 | December  |  9755795.0577 |
    | 2020 | January   |  9584951.9393 |
    | 2020 | February  |  8083995.5479 |
    | 2020 | March     |   766976.4531 |
    | 2020 | April     |   800071.9543 |
    | 2020 | May       |  1586964.4768 |
    | 2020 | June      |  3429736.5712 |
    | 2020 | July      |  5151815.4020 |
    | 2020 | August    |  5638281.8287 |
    | 2020 | September | 19530271.3028 |
    | 2020 | October   | 21016218.2095 |
    | 2020 | November  | 32247289.7946 |
    | 2020 | December  | 20409063.1769 |
    | 2021 | January   | 19570701.7102 |
    | 2021 | February  | 15986603.8883 |
    | 2021 | March     | 19149624.9239 |
    | 2021 | April     | 11483530.3032 |
    | 2021 | May       | 19204309.4095 |
    | 2021 | June      | 15457579.6626 |
    | 2021 | July      | 19044968.8164 |
    | 2021 | August    | 11324548.3409 |


8. **In which quarter of 2020, got the maximum total_sold_quantity**

    **Query:**

    ```sql
            SELECT QUARTER(date) AS Quarter, 
            SUM(sold_quantity) AS total_sold_quantity
            FROM fact_sales_monthly
            GROUP BY QUARTER(date)
            ORDER BY 2 DESC;
    ```

    **Results**

    | Quarter | total_sold_quantity |
    |---------|---------------------|
    |       4 |            25872947 |
    |       3 |            16271564 |
    |       1 |            14565784 |
    |       2 |            14227176 |

9. **Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution**

    **Query:**

    ```sql
        WITH total_gross AS (
        SELECT SUM(s.sold_quantity * g.gross_price) as total
        FROM fact_sales_monthly as s
        INNER JOIN fact_gross_price as g
        ON s.product_code = g.product_code),
        gross_sales AS (
        SELECT p.segment, SUM(s.sold_quantity * g.gross_price) as gross
        FROM dim_product as p
        INNER JOIN fact_gross_price as g
        ON p.product_code = g.product_code
        INNER JOIN fact_sales_monthly AS s
        ON p.product_code = s.product_code
        GROUP BY p.segment)
        SELECT g.segment AS channel,
        g.gross AS gross_sales_mln,
        g.gross / t.total as pct
        FROM gross_sales AS g
        CROSS JOIN total_gross as t;
    ```

    **Result:**

    | channel     | gross_sales_mln | pct        |
    |-------------|-----------------|------------|
    | Accessories |  991732980.1052 | 0.26718989 |
    | Desktop     |  109503332.3507 | 0.02950208 |
    | Networking  |  250856533.1612 | 0.06758506 |
    | Notebook    | 1258627137.3246 | 0.33909576 |
    | Peripherals |  803942190.6192 | 0.21659583 |
    | Storage     |  297053756.7692 | 0.08003138 |
