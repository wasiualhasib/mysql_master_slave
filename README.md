# mysql_master_slave

### Information
MasterIP: 192.168.43.100

SlaveIP: 192.168.43.101

ReplicationUser: replicator



### Step 1: Install MySQL on Both Servers

If MySQL is not already installed, install it using the following commands on both servers:

```bash
sudo dnf install @mysql
sudo systemctl start mysqld
sudo systemctl enable mysqld
```

### Step 2: Configure the Master Server (192.168.43.100)

1. **Edit the MySQL configuration file** (`/etc/my.cnf` or `/etc/my.cnf.d/server.cnf`):

```bash
sudo nano /etc/my.cnf
```

2. **Add the following lines to configure the master server**:

```ini
[mysqld]
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
bind-address = 192.168.43.100
```

3. **Restart MySQL** to apply the changes:

```bash
sudo systemctl restart mysqld
```

4. **Create a replication user** on the master server:

```sql
mysql -u root -p
```

```sql
CREATE USER 'replicator'@'192.168.43.101' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'192.168.43.101';
FLUSH PRIVILEGES;
```

5. **Lock the master database and get the binary log file and position**:

```sql
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
```

Note the `File` and `Position` values. You will need these for the slave configuration. Keep the session open to maintain the lock.

### Step 3: Configure the Slave Server (192.168.43.101)

1. **Edit the MySQL configuration file** on the slave server (`/etc/my.cnf` or `/etc/my.cnf.d/server.cnf`):

```bash
sudo nano /etc/my.cnf
```

2. **Add the following lines to configure the slave server**:

```ini
[mysqld]
server-id = 2
relay-log = /var/log/mysql/mysql-relay-bin.log
bind-address = 192.168.43.101
```

3. **Restart MySQL** to apply the changes:

```bash
sudo systemctl restart mysqld
```

4. **Configure the slave server to connect to the master**:

```sql
mysql -u root -p
```

```sql
CHANGE MASTER TO
MASTER_HOST='192.168.43.100',
MASTER_USER='replicator',
MASTER_PASSWORD='password',
MASTER_LOG_FILE='mysql-bin.000001',  # Use the File value from SHOW MASTER STATUS
MASTER_LOG_POS=123;  # Use the Position value from SHOW MASTER STATUS
```

5. **Start the slave replication**:

```sql
START SLAVE;
```

6. **Verify the replication status**:

```sql
SHOW SLAVE STATUS\G
```

Look for `Slave_IO_Running` and `Slave_SQL_Running` both being `Yes`.

### Step 4: Unlock the Master Database

After setting up the slave, you can unlock the master database:

```sql
UNLOCK TABLES;
```

### Step 5: Final Verification

On the master server, make some changes (e.g., insert data into any database), and verify that these changes appear on the slave server.

### Troubleshooting

- **Check MySQL logs** on both servers for errors (`/var/log/mysqld.log`).
- **Ensure network connectivity** between master and slave.
- **Check firewall rules** to ensure that the slave can connect to the master on the MySQL port (default is 3306).

### Additional Security and Optimization

- **Use SSL/TLS** for secure replication traffic.
- **Monitor replication** to ensure it is working as expected.
- **Consider additional configurations** for performance tuning and failover scenarios.

This updated setup will bind the MySQL server on the master to IP `192.168.43.100` and on the slave to IP `192.168.43.101`, ensuring proper network configuration for replication.
