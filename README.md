# FoodYum Grocery Store Data Analysis

FoodYum is a prominent grocery store chain in the United States, offering a wide range of products including produce, meat, dairy, baked goods, snacks, and other household food staples. This README provides information and SQL queries to perform data analysis on the FoodYum product data.

# Table of Contents
1. [Identifying Missing year_added Values](#identifying-missing-year_added-values)
2. [Ensuring Data Accuracy](#ensuring-data-accuracy)
3. [Finding Min and Max Prices for Each Product Type](#finding-min-and-max-prices-for-each-product-type)
4. [Detailed Analysis of Meat and Dairy Products](#detailed-analysis-of-meat-and-dairy-products)

# Identifying Missing year_added Values
To determine how many products have missing year_added values, I used the following query:
```
SELECT COUNT(*) AS missing_year
FROM products
WHERE year_added IS NULL;
```
This query outputs a single column, missing_year, indicating the number of products with missing year_added values.

# Ensuring Data Accuracy
Before starting the analysis, the data was cleaned so that each column name matched the given criteria. Without updating the original table, this was done using the following query:
```
SELECT
    product_id,
    COALESCE(product_type, 'Unknown') AS product_type,
    COALESCE(NULLIF(REPLACE(brand, '-', ''), ''), 'Unknown') AS brand,
    COALESCE(
      ROUND(CAST(REGEXP_REPLACE(weight, '[^\d.]', '', 'g') AS DECIMAL(10, 2)), 2), ROUND((SELECT PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY CAST(REGEXP_REPLACE(weight, '[^\d.]', '', 'g') AS DECIMAL(10, 2))) FROM products), 2)
            ) AS weight,
    COALESCE(
      TO_CHAR(CAST(price AS DECIMAL(10, 2)), '9999999999.99'),
      TO_CHAR(CAST((SELECT PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY price) FROM products) AS DECIMAL(10, 2)), '9999999999.99')
            ) AS price,
    COALESCE(average_units_sold, 0) AS average_units_sold,
    COALESCE(year_added, 2022) AS year_added,
    COALESCE(UPPER(stock_location), 'Unknown') AS stock_location
FROM products;
```
1. Missing product_type values were replaced with "Unknown".
2. Missing brand values were replaced with "Unknown".
3. The word "grams" was removed from the fields that contained it. The weight value is then rounded to two decimal places while missing values are replaced with the overall median weight.
4. Price values are rounded to two decimal places where values such as "11" and "5.2" become "11.00" or "5.20".
5. Missing average_units_sold values are replaced with 0.
6. Missing year_added values are replaced with "2022".
7. Missing stock_location values are replaced with "Unknown" while being uniformly formatted to all uppercase values.

# Finding Min and Max Prices for Each Product Type
To find the price range for each product type, I find the minimum and maximum price for each product using the following query:
```
SELECT product_type,
    MIN(price) AS min_price,
    MAX(price) AS max_price
FROM products
GROUP BY product_type;
```
This query groups by product_type to display the minimum and maximum price for each product type.

# Detailed Analysis of Meat and Dairy Products
To find the product ID and price of meat and dairy products where the average units sold was greater than ten, I used the following query:
```
SELECT
  product_id,
  price,
  average_units_sold
FROM products
WHERE (product_type = 'Meat' OR product_type = 'Dairy')
  AND average_units_sold > 10;
```
This query returns the product ID, price, and average units sold of only meat and diary products where the average units sold is greater than 10. 

