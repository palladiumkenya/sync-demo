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
## Step 1: Configure Master Site

Connect to master

```bash
    docker exec -it mysql-master mysql -uroot -pmauncentral
```
Setup to sync user to be used by slave sites
```bash
    CREATE USER 'syncuser'@'%' IDENTIFIED WITH mysql_native_password BY 'syncpassword';
    GRANT REPLICATION SLAVE ON *.* TO 'syncuser'@'%';
    FLUSH PRIVILEGES;
```

Load replication settings required by sites

```bash
    SHOW MASTER STATUS;
```

Please note the values `File` and `Position` as shown in this example you values might be different
 
| Attribure     | Value             |
| ------------- | ----------------- |
| File          | mysql-bin.000003  |
| Position      | 1351              |
| Binlog_Do_DB  | emrdb             |


## Step 2: Configure Site 1

Connect to site 01

```bash
    docker exec -it mysql-site01 mysql -uroot -pmaunclient
```
Ensure values for `MASTER_LOG_FILE` and `MASTER_LOG_POS` are set to as per the `File` and `Position` in step 1

```bash
    CHANGE MASTER TO
     MASTER_HOST='mysql-master',
     MASTER_USER='syncuser',
     MASTER_PASSWORD='syncpassword',
     MASTER_LOG_FILE='mysql-bin.000003',
     MASTER_LOG_POS=1351;

```

Start slave

```bash
    START SLAVE;

```

Verify replication on slave status has no errors

```bash
   SHOW SLAVE STATUS\G

```

## Step 3: Configure Site 2

Connect to site 02

```bash
    docker exec -it mysql-site02 mysql -uroot -pmaunclient
```
Ensure values for `MASTER_LOG_FILE` and `MASTER_LOG_POS` are set to as per the `File` and `Position` in step 1

```bash
    CHANGE MASTER TO
     MASTER_HOST='mysql-master',
     MASTER_USER='syncuser',
     MASTER_PASSWORD='syncpassword',
     MASTER_LOG_FILE='mysql-bin.000003',
     MASTER_LOG_POS=1351;

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
    CREATE TABLE patients (id INT, name VARCHAR(100));
    INSERT INTO patients VALUES (1, 'Maun Maun');
```