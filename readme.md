MySQL Replication example
## Pre-requisites

 - docker compose
 - For Windows, WSL required
 - Ubuntu


## Deployment

Checkout and run using docker compose

```bash
   docker compose up -d
```
## Step 1: Configure replication user on hub

Connect to hub

```bash
    docker exec -it mysql-hub mysql -uroot -pmauncentral
```
Setup to sync user to be used by replication
```bash
    CREATE USER 'syncuser'@'%' IDENTIFIED WITH mysql_native_password BY 'syncpassword';
    GRANT REPLICATION SLAVE ON *.* TO 'syncuser'@'%';
    FLUSH PRIVILEGES;
```

Load replication settings get GTID

```bash
    SHOW MASTER STATUS;
```

Please note the values `File` and `Position` as shown in this example you values might be different
 
| Attribure     | Value             |
| ------------- | ----------------- |
| File          | mysql-bin.000003  |
| Position      | 1351              |
| Binlog_Do_DB  | emrdb             |


## Step 2: Configure replication user on spoke

Connect to spoke

```bash
    docker exec -it mysql-spoke mysql -uroot -pmaunclient
```
Setup to sync user to be used by replication
```bash
    CREATE USER 'syncuser'@'%' IDENTIFIED WITH mysql_native_password BY 'syncpassword';
    GRANT REPLICATION SLAVE ON *.* TO 'syncuser'@'%';
    FLUSH PRIVILEGES;
```

Load replication settings

```bash
    SHOW MASTER STATUS;
```

Please note the values `File` and `Position` as shown in this example you values might be different
 
| Attribure     | Value             |
| ------------- | ----------------- |
| File          | mysql-bin.000003  |
| Position      | 1351              |
| Binlog_Do_DB  | emrdb             |


## Step 3: Configure Hub

Connect to hub

```bash
     docker exec -it mysql-hub mysql -uroot -pmauncentral
```
Ensure values for `MASTER_LOG_FILE` and `MASTER_LOG_POS` are set to as per the `File` and `Position` in step 1

```bash
    CHANGE MASTER TO
     MASTER_HOST='mysql-spoke',
     MASTER_USER='syncuser',
     MASTER_PASSWORD='syncpassword',
     MASTER_LOG_FILE='mysql-bin.000003',
     MASTER_LOG_POS=850;

```

Start slave

```bash
    START SLAVE;

```

Verify replication on slave status has no errors

```bash
   SHOW SLAVE STATUS\G

```

## Step 4: Configure spoke

Connect to s

```bash
    docker exec -it mysql-spoke mysql -uroot -pmaunclient
```
Ensure values for `MASTER_LOG_FILE` and `MASTER_LOG_POS` are set to as per the `File` and `Position` in step 1

```bash
    CHANGE MASTER TO
     MASTER_HOST='mysql-hub',
     MASTER_USER='syncuser',
     MASTER_PASSWORD='syncpassword',
     MASTER_LOG_FILE='mysql-bin.000003',
     MASTER_LOG_POS=850;

```

Start Slave

```bash
    START SLAVE;
```

Verify Replication status has no errors

```bash
   SHOW SLAVE STATUS\G
```

## Step 4: Testing Replication

Create a sample table with data and check that data is available in the other databases

```bash
    USE emrdb;
    CREATE TABLE patients (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(100));
    INSERT INTO patients (name) VALUES ('Maun Maun');
```