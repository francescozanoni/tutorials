# MySQL schema conversion to SQLite

### 0. Preliminary operations

* if not already available, create the data source:

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

* retrieve [mysql2sqlite](https://github.com/dumblob/mysql2sqlite) script and add its dependencies:

    1. install one of the many [AWK](https://en.wikipedia.org/wiki/AWK) implementations (GNU Awk, in the below example):

        ```bash
        sudo apt install gawk
        ```

    1. download mysql2sqlite script from its GitHub repository

    1. make the script executable:

        ```bash
        chmod +x mysql2sqlite
        ```

### 1. Export source data to MySQL dump file

```bash
mysqldump -h[host] -u[username] -p my_database > mysql_dump.sql --compact
Enter password: [password]
```

File *mysql_dump.sql* content will look like:

```
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(100) DEFAULT NULL,
  `created` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;
INSERT INTO `users` VALUES (1,'user_1','2017-03-12 09:18:04'),(2,'user_2','2017-03-12 09:18:04'),(3,'user_3','2017-03-12 09:18:04');
```

### References

* https://github.com/dumblob/mysql2sqlite
* https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html