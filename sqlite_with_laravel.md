# SQLite with Laravel

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
        DB_DATABASE=/path/to/example.com/database/database.sqlite

1. create the empty SQLite database file:

    ```bash
    touch database/database.sqlite
    ```

1. since it must be written from both the console user and the web user, set the SQLite database file writable by both:

    ```bash
    chmod 777 database/database.sqlite
    ```

1. to enable foreign key checks, add the following code to file **app/Providers/AppServiceProvider.php**:

    ```php
    public function boot()
    {
        if (DB::connection() instanceof \Illuminate\Database\SQLiteConnection) {
            DB::statement(DB::raw('PRAGMA foreign_keys=1'));
        }
    }
    ```

### References

* https://laravel.com/docs/5.4/database
* http://trzebinski.info/allow-sqlite-to-use-foreign-keys-in-laravel-4/
* http://stackoverflow.com/questions/31228950/laravel-5-1-enable-sqlite-foreign-key-constraints