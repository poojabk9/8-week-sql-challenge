# 🍜 Danny's Dinner 🍜

## Overview

### Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

## Data Set

Danny’s Diner contains 3 tables:

- `sales` – customer_id, order_date, product_id  
- `menu` – product_id, product_name, price  
- `members` – customer_id, join_date  

These tables can be found [on the official challenge site](https://8weeksqlchallenge.com/case-study-1/).

## Questions and Solutions

### Q1. What is the total amount each customer spent at the restaurant?

```sql
 SELECT sales.customer_id, SUM(menu.price) AS money_spent 
 FROM sales
 INNER JOIN menu
 	ON sales.product_id = menu.product_id
 GROUP BY customer_id 
 ORDER BY customer_id ASC;
```






