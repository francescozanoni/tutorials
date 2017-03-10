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

