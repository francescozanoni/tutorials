# SQLite with Laravel

This tutorial describes how to use SQLite as Laravel database engine.

### Requirements

* UNIX-like operating system
* PHP (exact version depends on Laravel version)
* PHP extensions:

    * PDO
    * SQLite
    * SQLite driver of PDO

* [Composer](https://getcomposer.org/) dependency manager
* SQLite (one of the several ways of accessing SQLite database file directly)

### Usage

1. create Laravel application via [Composer](https://getcomposer.org/) (for the scope of this tutorial, development dependencies are not required):

    ```bash
    composer create-project --prefer-dist laravel/laravel example.com --no-dev
    chmod -R 777 storage/*
    chmod -R 777 bootstrap/cache
    ```

1. customize file **.env** as required (for the scope of this tutorial, only the following few parameters are required):

        APP_ENV=local
        APP_DEBUG=true
        APP_KEY=base64:jgTzCe6Iv1eYmCM57jmpzGnBeRBHfPmsGI1MXftjCAY=
        APP_URL=http://example.com

        DB_CONNECTION=sqlite
        DB_DATABASE=/absolute/path/to/example.com/database/database.sqlite

    DB_DATABASE parameter is actually not required, in case Laravel's default value (**database/database.sqlite**) is fine

1. create the empty SQLite database file:

    ```bash
    touch database/database.sqlite
    ```

1. since it must be written from both the console user and the web user, set the SQLite database file writable by both:

    ```bash
    chmod 777 database/database.sqlite
    ```

1. to enable foreign key checks (disabled by default), add the following code to file **app/Providers/AppServiceProvider.php**:

    ```php
    public function boot()
    {
        if (DB::connection() instanceof \Illuminate\Database\SQLiteConnection) {
            DB::statement(DB::raw('PRAGMA foreign_keys=1'));
        }
    }
    ```
1. as a database write test, run database migrations:

    ```bash
    php artisan migrate
    ```

1. and, via SQLite command line, check the database content:

    ```bash
    sqlite3 database/database.sqlite

    SQLite version 3.8.2 2013-12-06 14:53:30
    Enter ".help" for instructions
    Enter SQL statements terminated with a ";"

    sqlite> .header ON

    sqlite> SELECT * FROM migrations;
    migration|batch
    2014_10_12_000000_create_users_table|1

    sqlite> .exit
    ```

### References

* https://laravel.com/docs/5.4/database
* https://sqlite.org/pragma.html#pragma_foreign_keys
* http://trzebinski.info/allow-sqlite-to-use-foreign-keys-in-laravel-4/
* http://stackoverflow.com/questions/31228950/laravel-5-1-enable-sqlite-foreign-key-constraints