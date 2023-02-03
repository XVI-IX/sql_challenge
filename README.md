1. Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.

    Query:

    ```sql
    SELECT DISTINCT(markets)
    FROM gdb023.dim_customer
    WHERE customer = 'Atliq Exclusive'
    AND region = 'APAC';
    ```

2. What is the percentage of unique product increase in 2021 vs. 2020?

    Query:

    - Step 1:

        ```sql
        WITH count_2020 AS (
          SELECT count(distinct(product_code)) as count_2020, fiscal_year
          from fact_sales_monthly
          where fiscal_year = '2020'),
          count_2021 AS (
          SELECT count(distinct(product_code)) as count_2021, fiscal_year
          from fact_sales_monthly
          where fiscal_year = '2021')
        ```

    - Step 2:

        ```sql
          SELECT a.count_2020 as unique_products_2020,
          b.count_2021 as unique_products_2021,
        (((b.count_2021 - a.count_2020) / a.count_2020) * 100) as percentage_chg
          FROM count_2020 as a
          cross join count_2021 as b;
              ```

3. Provide a report with all the unique product counts for each segment and sort them in descending order of product counts.

    Query:

    - Step 1:

        ```sql
        SELECT COUNT(DISTINCT(product_code)) as product_code, segment
        FROM dim_product 
        GROUP BY segment
        ORDER BY 1 DESC;
        ```

4. Follow-up: Which segment had the most increase in unique products in 2021 vs 2020?

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

5. Get the products that have the highest and lowest manufacturing costs.

Query:

- Step 1: highest manufacturing costs

    ```sql
    SELECT dim_product.product_code, product, manufacturing_cost
    FROM  dim_product
    INNER JOIN fact_manufacturing_cost
    ON 
    dim_product.product_code = fact_manufacturing_cost.product_code
    ORDER BY manufacturing_cost DESC
    LIMIT 1;
    ```

    --Step 2: lowest manufacturing costs

    ```sql
    SELECT dim_product.product_code, product, manufacturing_cost
    FROM  dim_product
    INNER JOIN fact_manufacturing_cost
    ON 
    dim_product.product_code = fact_manufacturing_cost.product_code
    ORDER BY manufacturing_cost
    LIMIT 1;
    ```
