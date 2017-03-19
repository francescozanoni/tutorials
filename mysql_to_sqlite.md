# MySQL schema conversion to SQLite

### Requirements

* UNIX-like operating system
* MySQL command line client
* SQLite

### 0. Preliminary operations

* if not already available, create the data source:

    1. create the schema to convert:

        ```bash
        $ mysql -h[host] -u[username] -p
        Enter password: [password]
        Welcome to the MySQL [... and the rest of MySQL welcome message]
        ```
        
        ```bash
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

    1. install one of the many [AWK](https://en.wikipedia.org/wiki/AWK) implementations ([GNU Awk](https://www.gnu.org/software/gawk/), in the below example):

        ```bash
        $ sudo apt install gawk
        ```

    1. download mysql2sqlite script from [its GitHub repository](https://github.com/dumblob/mysql2sqlite)

    1. make the script executable:

        ```bash
        $ chmod +x mysql2sqlite.sh
        ```

### 1. Export MySQL data to MySQL dump file

Export data via MySQL client command line interface (or any MySQL client you like, of course):

```bash
$ mysqldump -h[host] -u[username] -p my_database > mysql_dump.sql --compact
Enter password: [password]
```

Output file *mysql_dump.sql* content will look like:

```sql
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
INSERT INTO `users` VALUES (1,'user_1','2017-03-12 09:18:04'),
                           (2,'user_2','2017-03-12 09:18:06'),
                           (3,'user_3','2017-03-12 09:18:08');
```

### 2. Convert MySQL dump file to a SQLite dump file

```bash
$ ./mysql2sqlite.sh mysql_dump.sql > sqlite_dump.sql
```

Output file *sqlite_dump.sql* content will look like:

```sql
PRAGMA synchronous = OFF;
PRAGMA journal_mode = MEMORY;
BEGIN TRANSACTION;
CREATE TABLE `users` (
  `id` integer NOT NULL PRIMARY KEY AUTOINCREMENT,
  `username` varchar(100) DEFAULT NULL,
  `created` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  UNIQUE (`username`)
);
INSERT INTO `users` VALUES (1,'user_1','2017-03-12 09:18:04'),
                           (2,'user_2','2017-03-12 09:18:04'),
                           (3,'user_3','2017-03-12 09:18:04');
END TRANSACTION;
```

Further details about how SQLite handles data types other than TEXT, INTEGER, REAL, NUMERIC and BLOB are available on [SQLite documentation](https://www.sqlite.org/datatype3.html#type_affinity)

### 3. Import SQLite dump file to SQLite database

Import data via SQLite command line interface:

```bash
$ sqlite3 my_database.sqlite
SQLite version 3.8.2 [... and the rest of SQLite welcome message]

sqlite> .read sqlite_dump.sql
memory

sqlite> .exit
```

### 4. Check data have been correctly imported to SQLite

```bash
$ sqlite3 my_database.sqlite
SQLite version 3.8.2 [... and the rest of SQLite welcome message]

sqlite> .tables
users

sqlite> .schema users
CREATE TABLE `users` (
  `id` integer NOT NULL PRIMARY KEY AUTOINCREMENT,
  `username` varchar(100) DEFAULT NULL,
  `created` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  UNIQUE (`username`)
);

sqlite> SELECT * FROM users;
1|user_1|2017-03-12 09:18:04
2|user_2|2017-03-12 09:18:04
3|user_3|2017-03-12 09:18:04

sqlite> .exit
```


### References

* https://github.com/dumblob/mysql2sqlite
* https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html
* https://www.sqlite.org/cli.html