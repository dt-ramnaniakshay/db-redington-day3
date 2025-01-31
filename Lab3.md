# Hands-On Lab: Replication in PostgreSQL

## Objective
This lab demonstrates **Replication in PostgreSQL**, including **Streaming Replication** and **Read Replicas** for high availability and load balancing.

## Prerequisites
- Two PostgreSQL servers (Primary & Replica)
- PostgreSQL installed (version 10 or later)
- Network connectivity between the servers
- Superuser privileges

---
## Part 1: Configuring the Primary Server

### **Step 1: Enable WAL Archiving and Replication**
Modify the `postgresql.conf` file on the **primary server**:
```ini
wal_level = replica
archive_mode = on
archive_command = 'cp %p /var/lib/postgresql/archive/%f'
max_wal_senders = 5
wal_keep_size = 128
hot_standby = on
```
Restart PostgreSQL:
```sh
sudo systemctl restart postgresql
```

### **Step 2: Create a Replication User**
```sql
CREATE ROLE replica_user WITH REPLICATION LOGIN PASSWORD 'replica_password';
```

### **Step 3: Configure `pg_hba.conf`**
Add the following line to allow the replica to connect:
```ini
host replication replica_user replica_ip/32 md5
```
Reload PostgreSQL:
```sh
sudo systemctl reload postgresql
```

---
## Part 2: Setting Up the Replica Server

### **Step 1: Stop PostgreSQL on the Replica**
```sh
sudo systemctl stop postgresql
```

### **Step 2: Take a Base Backup from the Primary Server**
```sh
pg_basebackup -h primary_ip -U replica_user -D /var/lib/postgresql/15/main -P -R
```

### **Step 3: Configure `recovery.conf` (If PostgreSQL <12)**
```ini
standby_mode = 'on'
primary_conninfo = 'host=primary_ip port=5432 user=replica_user password=replica_password'
trigger_file = '/var/lib/postgresql/trigger_file'
```

### **Step 4: Start the Replica Server**
```sh
sudo systemctl start postgresql
```

### **Step 5: Verify Replication Status**
On the **Primary Server**:
```sql
SELECT * FROM pg_stat_replication;
```
On the **Replica Server**:
```sql
SELECT * FROM pg_stat_wal_receiver;
```

---
## Part 3: Setting Up a Read Replica

### **Step 1: Enable Hot Standby**
Ensure `hot_standby = on` is set in `postgresql.conf` (Replica Server):
```ini
hot_standby = on
```

### **Step 2: Test Read Queries on Replica**
Connect to the replica and run:
```sql
SELECT * FROM pg_stat_activity;
SELECT COUNT(*) FROM some_large_table;
```
If successful, the replica is now a **read-only replica**.

---
## Conclusion
This lab covered:
- Configuring **Streaming Replication**
- Setting up a **Read Replica** for query offloading
- Verifying replication status

Replication in PostgreSQL ensures high availability and load balancing for production environments.
