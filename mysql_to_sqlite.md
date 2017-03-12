# MySQL schema conversion to SQLite

### 0. Preliminary operations

* if not already available,

    1. create the schema to convert:

        ```bash
        $ mysql -h[host] -u[username] -p
        Enter password: [password]
        Welcome to the MySQL [... and the rest of MySQL welcome message]
        
        mysql> CREATE SCHEMA my_database;
        Query OK, 1 row affected (0.00 sec)
        
        mysql> USE my_database;
        Database changed
        ```

    1. together with a table:

        ```bash
        mysql> CREATE TABLE users (
            id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
            username VARCHAR(100) UNIQUE,
            created TIMESTAMP DEFAULT NOW()
        ) DEFAULT CHARSET=utf8;
        Query OK, 0 rows affected (0.12 sec)
        ```

    1. and populate it with a few records:

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

* install one of the many AWK implementations:

    ```bash
    sudo apt install gawk
    ```
