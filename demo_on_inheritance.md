### Inheritance in PostgreSQL

Inheritance in PostgreSQL is a feature that allows tables to inherit from other tables, similar to object-oriented programming (OOP) inheritance. It enables a child table to inherit the structure and data of a parent table. This allows for data model designs that can be more flexible and reusable.

When a table inherits from another, it automatically gets all the columns and constraints from the parent table, while still allowing for specific columns or constraints to be added to the child table.

---


### Syntax for Creating Inherited Tables

#### 1. **Create a Parent Table**
   A parent table is a normal table that will act as the base for other tables to inherit from.

   ```sql
   CREATE TABLE parent_table (
       id SERIAL PRIMARY KEY,
       name VARCHAR(100),
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```

#### 2. **Create a Child Table**
   A child table is created with the `INHERITS` clause, and it inherits from the parent table. You can define additional columns in the child table.

   ```sql
   CREATE TABLE child_table (
       description TEXT
   ) INHERITS (parent_table);
   ```

   Here, `child_table` inherits all the columns (`id`, `name`, and `created_at`) from `parent_table` and also has a new column `description`.

---

### Example Usage

#### 1. **Creating Parent and Child Tables:**

```sql
-- Create the parent table
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL
);

-- Create the child table, inheriting from the products table
CREATE TABLE electronics (
    warranty_period INT
) INHERITS (products);
```

Here, `electronics` inherits the columns `product_id`, `product_name`, and `price` from the `products` table, and adds an additional column `warranty_period`.

#### 2. **Inserting Data into Parent and Child Tables:**

```sql
-- Insert into the parent table
INSERT INTO products (product_name, price) VALUES
('Product A', 100.00),
('Product B', 150.00);

-- Insert into the child table
INSERT INTO electronics (product_name, price, warranty_period) VALUES
('TV', 500.00, 2),
('Smartphone', 300.00, 1);
```

#### 3. **Querying Data from Parent and Child Tables:**

- Querying from the parent table will return data from both the parent and the child tables:

```sql
SELECT * FROM products;
```

This query will return all rows from both the `products` and `electronics` tables, since `electronics` inherits from `products`.

- If you only want data from the parent table (without child rows), you can use the `ONLY` keyword:

```sql
SELECT * FROM ONLY products;
```

This will return only rows from the `products` table and exclude rows from the `electronics` table.

#### 4. **Dropping a Table:**

If you drop a parent table, it does not automatically drop the child tables. You must manually drop the child tables as well:

```sql
-- Drop the parent table (does not drop child tables)
DROP TABLE products;

-- Drop the child table
DROP TABLE electronics;
```

---



### When to Use Table Inheritance:
- **Use cases** where you want to create a hierarchy of tables sharing common columns but with specific variations in some child tables.
- It is helpful in scenarios such as:
  - When you want to have a base set of attributes (columns) for similar objects but need to extend the child tables with additional attributes.
  - For example, a general `vehicles` table with common attributes and specific `cars`, `trucks`, and `motorcycles` tables inheriting from it.

---