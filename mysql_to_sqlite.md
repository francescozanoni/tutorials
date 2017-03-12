# MySQL schema conversion to SQLite

### 0. Preliminary operations

* if not already available,

    * create the schema to convert:

        ```bash
$ mysql -u[username] -p
Enter password: [password]
Welcome to the MySQL [... and the rest of MySQL welcome message]
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

