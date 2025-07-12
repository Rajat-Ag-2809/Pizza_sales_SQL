# 🍕 Pizza Sales Data Analysis using SQL

![](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## Overview
This project focuses on analyzing historical pizza sales data from a fictional pizza store using SQL. The dataset includes details about customer orders, pizza types, sizes, prices, and categories. The aim is to extract actionable business insights through structured queries that can help improve operational efficiency, marketing strategy, and inventory planning.

## Objectives

- Understand Sales Patterns – Analyze order frequency, popular pizza types, and customer preferences.
- Evaluate Revenue Performance – Identify top revenue-generating items and patterns over time.
- Evaluate Revenue Performance – Identify top revenue-generating items and patterns over time.
- Practice SQL Skills – Apply various SQL techniques like aggregation, joins, subqueries, window functions, and CTEs on a realistic dataset.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- *Dataset Link:* [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS orders;
CREATE TABLE orders (
	  order_id INT NOT NULL,
    order_date DATE NOT NULL,
    order_time TIME NOT NULL,
    PRIMARY KEY(order_id) );

DROP TABLE IF EXISTS order_details;
CREATE TABLE order_details (
	  order_details_id INT NOT NULL,
	  order_id INT NOT NULL,
    pizza_id TEXT NOT NULL,
    quantity INT NOT NULL,
    PRIMARY KEY(order_details_id) );
```

## Business Problems and Solutions

### 1. Retrieve the total number of orders placed.
```sql
SELECT count(order_id) AS total_orders FROM orders;
```
### 2. Calculate the total revenue generated from pizza sales.
```sql
SELECT round(sum(orders_details.quantity * pizzas.price), 2) AS total_sales
FROM orders_details JOIN pizzas
ON orders_details.pizza_id = pizzas.pizza_id;
```
### 3. Identify the highest-priced pizza.
```sql
SELECT pizza_types.name, pizzas.price
FROM pizza_types JOIN pizzas
ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;
```
### 4. Identify the most common pizza size ordered.
```sql
SELECT pizzas.size, count(order_details.order_details_id) AS order_count
FROM pizzas JOIN order_details
ON pizzas.pizza_id = order_details.pizza_id 
GROUP BY pizzas.size
ORDER BY order_count DESC;
```
### 5. List the top 5 most ordered pizza types along with their quantities.
```sql
SELECT pizza_types.name, sum(order_details.quantity) AS quantity
FROM pizza_types JOIN pizzas
ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details
ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC
LIMIT 5;
```
### 6. Join the necessary tables to find the total quantity of each pizza category ordered.
```
SELECT pizza_types.category, sum(order_details.quantity) AS quantity
FROM pizza_types JOIN pizzas
ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details
ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY quantity DESC;
```
### 7. Determine the distribution of orders by hour of the day.
```
SELECT HOUR(order_time) AS hour, count(order_id) AS order_count
FROM orders
GROUP BY HOUR(order_time);
```
### 8. Join relevant tables to find the category-wise distribution of pizzas.
```
SELECT category, count(name) 
FROM pizza_types
GROUP BY category;
```
### 9. Group the orders by date and calculate the average number of pizzas ordered per day.
```
SELECT round(AVG(quantity), 0) AS avg_pizza_ordered_per_day
FROM 
(SELECT orders.order_date, sum(order_details.quantity) AS quantity
FROM orders JOIN order_details
ON orders.order_id = order_details.order_id
GROUP BY orders.order_date) AS order_quantity;
```
### 10. Determine the top 3 most ordered pizza types based on revenue.
```
SELECT pizza_types.name, sum(order_details.quantity * pizzas.price) AS revenue
FROM pizza_types JOIN pizzas
ON pizzas.pizza_type_id = pizza_types.pizza_type_id
JOIN order_details
ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;
```
### 11. Calculate the percentage contribution of each pizza type to total revenue.
```
SELECT pizza_types.category, 
round(sum(order_details.quantity * pizzas.price) / (SELECT round(sum(order_details.quantity * pizzas.price), 2) AS total_sales
FROM order_details JOIN pizzas 
ON pizzas.pizza_id = order_details.pizza_id)* 100, 2) AS revenue
FROM pizza_types JOIN pizzas
ON pizzas.pizza_type_id = pizza_types.pizza_type_id
JOIN order_details
ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY revenue DESC;
```
### 12. Analyze the cumulative revenue generated over time.
```
SELECT order_date, sum(revenue) 
OVER(ORDER BY order_date) AS cum_revenue
FROM
(SELECT orders.order_date, 
sum(order_details.quantity * pizzas.price) AS revenue
FROM order_details JOIN pizzas
ON order_details.pizza_id = pizzas.pizza_id
JOIN orders
ON orders.order_id = order_details.order_id
GROUP BY orders.order_date) AS sales;
```
### 13. Determine the top 3 most ordered pizza types based on revenue for each pizza category.
```
SELECT name, revenue 
FROM
(SELECT category, name, revenue,
RANK() OVER(PARTITION BY category ORDER BY revenue DESC) AS rn
FROM
(SELECT pizza_types.category, pizza_types.name, 
sum(order_details.quantity * pizzas.price) AS revenue
FROM pizza_types JOIN pizzas
ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details
ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category, pizza_types.name) AS a) AS b
WHERE rn <= 3;
```
## Findings and Conclusion

- **Order Insights:** The business has strong data for order timing, product popularity, and size preferences.
- **Revenue Analysis:** A few pizzas dominate revenue contribution, and there are clear high performers by category.
- **Customer Behavior:** Peak hours and day-wise ordering trends provide useful data for marketing and staffing.
- **Strategic Planning:** Knowing which category contributes most to revenue helps refine inventory and promotional strategies.
      
