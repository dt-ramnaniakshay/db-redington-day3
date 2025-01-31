Sure! Below is a **step-by-step hands-on lab** on **Point-in-Time Recovery (PITR)** in PostgreSQL 16. PITR allows you to recover a PostgreSQL database to a specific point in time, which can be useful in scenarios where you need to restore the database to a particular state due to accidental data loss or corruption.

### **Hands-On Lab: Point-in-Time Recovery (PITR) in PostgreSQL 16**

#### **Prerequisites**:
- PostgreSQL 16 must be installed on your system.
- You need to have access to the `postgres` user or a user with sufficient privileges to perform backups and restore operations.
- The `archive_mode` should be enabled, and `archive_command` should be set up for WAL (Write-Ahead Logging) archiving.

#### **Step 1: Enable WAL Archiving for PITR**

First, enable WAL archiving on the PostgreSQL server to allow Point-in-Time Recovery.

1. Open the `postgresql.conf` file (typically located in `/etc/postgresql/{version}/main/postgresql.conf` on Ubuntu or `/var/lib/pgsql/{version}/data/postgresql.conf` on CentOS).
   
   ```bash
   sudo nano /etc/postgresql/16/main/postgresql.conf
   ```

2. Make sure the following settings are configured to enable WAL archiving:
   ```plaintext
   archive_mode = on
   archive_command = 'cp %p /var/lib/postgresql/archive/%f'
   ```

   - **`archive_mode = on`**: Enables archiving of WAL files.
   - **`archive_command`**: Specifies how to store WAL files. This example uses `cp` to copy the WAL files to `/var/lib/postgresql/archive/`.



3. 
   ```bash
   cd /var/lib/postgresql
   sudo mkdir archive data backup
    ```

4. 

```bash
   cd  /etc/postgresql/16/main
   sudo nano pg_hba.conf
```
you will see something like this

```bash
local   all             all                                     peer
```

change it to this

```
local   all             all                                     md5
```

1. Restart PostgreSQL for the changes to take effect:

   ```bash
   sudo systemctl restart postgresql
   ```

#### **Step 2: Take a Base Backup Using `pg_basebackup`**

To perform PITR, you need to take a **base backup** of your database, which serves as the starting point for recovery.

1. Take a base backup using `pg_basebackup`. Run the following command to create a backup of the `company_db` database:

   ```bash
   pg_basebackup -U postgres -D /var/lib/postgresql/backup -F tar -z -P
   ```

   - **`-D /var/lib/postgresql/backup/`**: Specifies the directory where the backup will be stored.
   - **`-Ft`**: Specifies the format (tar in this case).
   - **`-z`**: Compresses the backup.
   - **`-P`**: Shows progress during the backup process.

2. Ensure the backup is complete. You should see a file named `base.tar.gz` in the `/var/lib/postgresql/backup/` directory.

#### **Step 3: Enable WAL Archiving and Wait for WAL Files**

Ensure WAL files are being generated and archived as expected. Check the `/var/lib/postgresql/archive/` directory for archived WAL files:

```bash
ls /var/lib/postgresql/archive/
```

WAL files will be stored in this directory as `.gz` files (e.g., `000000010000000200000001.gz`).

#### **Step 4: Simulate Data Loss or Corruption**

Now, let’s simulate a scenario where some data is accidentally deleted or corrupted.

1. Connect to the PostgreSQL instance:

   ```bash
   psql -U postgres -d company_db
   ```

2. Create a table and insert some data for this demo:

   ```sql
   CREATE TABLE employees (
       id SERIAL PRIMARY KEY,
       name VARCHAR(50)
   );
   
   INSERT INTO employees (name) VALUES
   ('Alice'),
   ('Bob'),
   ('Charlie');
   ```

3. Check the data:

   ```sql
   SELECT * FROM employees;
   ```

   You should see:
   ```plaintext
   id |  name
   ----+--------
    1 | Alice
    2 | Bob
    3 | Charlie
   ```

4. Simulate data loss by deleting all records:

   ```sql
   DELETE FROM employees;
   ```

5. Verify the data is deleted:

   ```sql
   SELECT * FROM employees;
   ```

   The result should be empty (no rows).

#### **Step 5: Perform Point-in-Time Recovery (PITR)**

Now, let’s perform a Point-in-Time Recovery to restore the database to the state before the data was deleted.

1. **Stop the PostgreSQL Server**:
   Stop the PostgreSQL service to prepare for the restore.

   ```bash
   sudo systemctl stop postgresql
   ```

2. **Restore the Base Backup**:
   Extract the base backup taken earlier and restore it to the data directory.

   ```bash
   tar -xvf /var/lib/postgresql/backup/base.tar.gz -C /var/lib/postgresql/data/
   ```

3. **Set up the Recovery Target Time**:
   Create a `recovery.conf` file in the PostgreSQL data directory to specify the point in time you want to recover to. In this example, let’s assume we want to restore the database to the time **before** the `DELETE` operation.

   In the `recovery.conf` file, specify the **restore target time** and the location of the archived WAL files:

   ```plaintext
   restore_target_time = '2025-01-29 12:00:00'  # Adjust to the correct time
   recovery_target_action = pause
   archive_cleanup_command = 'pg_archivecleanup /var/lib/postgresql/archive %r'
   restore_command = 'cp /var/lib/postgresql/archive/%f %p'
   ```

   - **`restore_target_time`**: Set this to the point in time you want to restore to (before the `DELETE`).
   - **`restore_command`**: Specifies how to retrieve archived WAL files.
   - **`archive_cleanup_command`**: Cleans up old WAL files after recovery.

4. **Start PostgreSQL**:
   Start the PostgreSQL service, which will now begin applying WAL files up to the specified point in time.

   ```bash
   sudo systemctl start postgresql
   ```

   PostgreSQL will automatically stop recovering once it reaches the `restore_target_time` specified in the `recovery.conf` file.

5. **Check the Data**:
   After recovery, connect to PostgreSQL and verify the data has been restored:

   ```bash
   psql -U postgres -d company_db
   ```

   Run the following query:

   ```sql
   SELECT * FROM employees;
   ```

   You should see the data that was present before the delete operation.

   ```plaintext
   id |  name
   ----+--------
    1 | Alice
    2 | Bob
    3 | Charlie
   ```

#### **Step 6: Clean up the Recovery Configuration**

Once the recovery is complete, remove the `recovery.conf` file, as it is no longer needed for normal operations.

```bash
rm /var/lib/postgresql/data/recovery.conf
```

You may also want to perform a checkpoint or restart PostgreSQL to make sure it’s back to normal operation.

```bash
sudo systemctl restart postgresql
```

---

### **Conclusion:**

In this lab, you've successfully:
1. **Enabled WAL archiving** for Point-in-Time Recovery.
2. Took a **base backup** of the database.
3. Simulated a **data loss** scenario.
4. Performed a **Point-in-Time Recovery** to restore the database to a specific point before the data was deleted.

Point-in-Time Recovery is a powerful feature that can help recover from accidental data loss or corruption by rolling back to a specific time. Make sure to regularly archive your WAL files and take periodic backups for disaster recovery.
