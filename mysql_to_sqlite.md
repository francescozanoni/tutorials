# MySQL schema conversion to SQLite

### 0. Preliminary operations

* if not already available,

    * create the schema to convert:

        ```bash
$ mysql -u[username] -p
Enter password: [password]
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 37
Server version: 5.5.54-0ubuntu0.14.04.1 (Ubuntu)
Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE SCHEMA my_database;
Query OK, 1 row affected (0.00 sec)

mysql> USE my_database;
Database changed
        ```

    * together with a table:

        ```bash
mysql> CREATE TABLE users (
    id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(100) UNIQUE,
    created TIMESTAMP DEFAULT NOW()
) DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.12 sec)
        ```

    * and populate it with a few records:

        ```bash
mysql> INSERT INTO users (username) VALUES ('user_1');
Query OK, 1 row affected (0.05 sec)

mysql> INSERT INTO users (username) VALUES ('user_2');
Query OK, 1 row affected (0.05 sec)

mysql> INSERT INTO users (username) VALUES ('user_3');
Query OK, 1 row affected (0.05 sec)
        ```

        ```bash
mysql> EXIT;
        ```

