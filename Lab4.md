# Hands-On Lab: Window Functions in PostgreSQL

## Objective
This lab demonstrates how to use **Window Functions** in PostgreSQL, including **Ranking Functions, Aggregate Functions, and Analytical Functions**.

## Prerequisites
- PostgreSQL installed (version 10 or later)
- Basic knowledge of SQL

---
## Part 1: Creating a Sample Dataset

### **Step 1: Create a Sample Table**
```sql
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    salesperson TEXT NOT NULL,
    region TEXT NOT NULL,
    sales_amount DECIMAL(10,2),
    sale_date DATE
);
```

### **Step 2: Insert Sample Data**
```sql
INSERT INTO sales (salesperson, region, sales_amount, sale_date) VALUES
    ('Alice', 'North', 5000, '2024-01-01'),
    ('Bob', 'North', 7000, '2024-01-02'),
    ('Charlie', 'South', 6000, '2024-01-03'),
    ('David', 'South', 8000, '2024-01-04'),
    ('Alice', 'North', 4500, '2024-01-05'),
    ('Bob', 'North', 7500, '2024-01-06'),
    ('Charlie', 'South', 6500, '2024-01-07'),
    ('David', 'South', 8500, '2024-01-08');
```

---
## Part 2: Using Window Functions

### **1. Ranking Functions (`RANK`, `DENSE_RANK`, `ROW_NUMBER`)**
```sql
SELECT salesperson, region, sales_amount,
    RANK() OVER (PARTITION BY region ORDER BY sales_amount DESC) AS rank,
    DENSE_RANK() OVER (PARTITION BY region ORDER BY sales_amount DESC) AS dense_rank,
    ROW_NUMBER() OVER (PARTITION BY region ORDER BY sales_amount DESC) AS row_num
FROM sales;
```

### **2. Running Totals and Moving Averages (`SUM`, `AVG`)**
```sql
SELECT salesperson, region, sales_amount,
    SUM(sales_amount) OVER (PARTITION BY region ORDER BY sale_date) AS running_total,
    AVG(sales_amount) OVER (PARTITION BY region ORDER BY sale_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM sales;
```

### **3. Lag and Lead Analysis (`LAG`, `LEAD`)**
```sql
SELECT salesperson, region, sales_amount, sale_date,
    LAG(sales_amount) OVER (PARTITION BY region ORDER BY sale_date) AS prev_sale,
    LEAD(sales_amount) OVER (PARTITION BY region ORDER BY sale_date) AS next_sale
FROM sales;
```

---
## Conclusion
This lab covered:
- **Ranking Functions**: `RANK`, `DENSE_RANK`, `ROW_NUMBER`
- **Aggregate Functions with Windows**: `SUM`, `AVG`
- **Analytical Functions**: `LAG`, `LEAD`

Window functions allow efficient analysis of data trends, rankings, and running totals without grouping the dataset.

