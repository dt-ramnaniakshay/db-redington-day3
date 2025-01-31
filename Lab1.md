# Hands-On Lab: Logical Backup in PostgreSQL

## Objective
This lab demonstrates how to perform **logical backups** in PostgreSQL using `pg_dump` and `pg_dumpall`, along with restoring the backup using `psql`.

## Prerequisites
- PostgreSQL installed (version 10 or later)
- A PostgreSQL user with appropriate privileges
- Basic knowledge of SQL and PostgreSQL

---
## Part 1: Creating a Sample Database

### Step 1: Connect to PostgreSQL
```sh
psql -U postgres
```

### Step 2: Create a Sample Database
```sql
CREATE DATABASE company_db;
```

### Step 3: Connect to `company_db` and Create a Table
```sh
\c company_db
```

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    department TEXT NOT NULL,
    salary DECIMAL(10,2)
);
```

### Step 4: Insert Sample Data
```sql
INSERT INTO employees (name, department, salary) VALUES
    ('Alice', 'Engineering', 60000),
    ('Bob', 'Marketing', 50000),
    ('Charlie', 'HR', 45000);
```

---
## Part 2: Performing Logical Backup

### **1. Backup a Single Database Using `pg_dump`**
```sh
pg_dump -U postgres -d company_db -F c -f company_db_backup.dump
```
- `-U postgres` → Connect as user `postgres`
- `-d company_db` → Backup the `company_db` database
- `-F c` → Use **custom format**
- `-f company_db_backup.dump` → Output file

### **2. Backup a Single Table**
```sh
pg_dump -U postgres -d company_db -t employees -F c -f employees_backup.dump
```

### **3. Backup All Databases Using `pg_dumpall`**
```sh
pg_dumpall -U postgres > all_databases_backup.sql
```

---
### before restore make sure you create the database with exact same name `create database company_db;`
### not its not required in the case of restoring all database;
---

## Part 3: Restoring Logical Backup

### **1. Restore a Single Database**
```sh
pg_restore -U postgres -d company_db -F c company_db_backup.dump
```

### **2. Restore a Single Table**
```sh
pg_restore -U postgres -d company_db -F c -t employees employees_backup.dump
```

### **3. Restore All Databases**
```sh
psql -U postgres -f all_databases_backup.sql
```

---
## Conclusion
This lab covered PostgreSQL **logical backup methods** using:
- `pg_dump` for **single database and table backups**
- `pg_dumpall` for **all databases backup**
- `pg_restore` and `psql` for **restoration**

These methods help ensure data safety and quick recovery in case of failures.

