# SAML authentication/authorization with Laravel

This tutorial describes how to implement user authentication and authorization (simply "auth" from now on) on a Laravel application via an external service. This auth paradigm is usually know as [Single sign-on](https://en.wikipedia.org/wiki/Single_sign-on) (SSO), its purpose is centralizing all auth management into a dedicated system.

[Security Assertion Markup Language](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language) (SAML) auth is one of the several SSO implementations commonly used, based on an open data exchange format.

Some terms:

* IdP (Identity Provider): application providing auth service
* SP (Service Provider): application subscribed to use an IdP's auth service

Laravel 5.2 version has been chosen, since it's the first version implementing a standard middleware group by default (see [Laravel 5.2's release notes](https://laravel.com/docs/5.2/releases)).



### 0. Preliminary operations

1. create the [SSL public certificate and related key](https://en.m.wikipedia.org/wiki/Public-key_cryptography), if not already available, and store them together with IdP's public certificate (e.g. idp.example.com.crt):

    ```bash
    openssl req -newkey rsa:2048 -new -x509 -days 3652 -nodes \
        -out /path/to/certificate_folder/sp.example.com.crt \
        -keyout /path/to/certificate_folder/sp.example.com.pem
    cp /path/to/idp.example.com.crt /path/to/certificate_folder
    ```



### 1. Laravel application installation

1. create Laravel application via [Composer](https://getcomposer.org/):

    ```bash
    composer create-project --prefer-dist laravel/laravel 5.2.* sp.example.com --no-dev
    chmod -R 777 storage/*
    chmod -R 777 bootstrap/cache
    ```

    ***tutorial-scoped assumption*** *: development dependencies are not required*

1. customize file **.env** as required (for the scope of this tutorial, only the following few parameters are required):

        APP_ENV=local
        APP_DEBUG=true
        APP_KEY=base64:jgTzCe6Iv1eYmCM57jmpzGnBeRBHfPmsGI1MXftjCAY=
        APP_URL=http://sp.example.com

        DB_HOST=my.db.host
        DB_DATABASE=my_database
        DB_USERNAME=my_username
        DB_PASSWORD=my_password



### 2. Users table customization

Since Laravel cannot handle authentication without a users table, create it:

1. simplify users table migration (password field is useless, with SAML authentication) by editing file **database/migrations/2014_10_12_000000_create_users_table.php**:

    ```php
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('email')->unique();
            $table->rememberToken();
            $table->timestamps();
        });
    }
    ```
    "email" field is used as unique user identifier

1. since users are managed by the IdP, password reset table migration can be removed:

    ```bash
    rm database/migrations/2014_10_12_100000_create_password_resets_table.php
    ```

1. run migration of customized users table:

    ```bash
    php artisan migrate
    ```



### 3. SAML library installation

1. install Alejandro Cotroneo's [SAML library](https://github.com/aacotroneo/laravel-saml2) via [Composer](https://getcomposer.org/) and publish its configurations to file **config/saml2_settings.php**:

    ```bash
    composer require aacotroneo/laravel-saml2 --update-no-dev
    php artisan vendor:publish
    ```

1. add its [service provider](https://laravel.com/docs/5.2/providers) to file **config/app.php**:

    ```php
    'providers' => [
        ...
        Aacotroneo\Saml2\Saml2ServiceProvider::class,
    ]
    ```
    this operation sets the following list of routes:

    URI            | Name          | Action                                                     | Middleware
    ---------------|---------------|------------------------------------------------------------|-----------
    /              |               | Closure                                                    | web
    saml2/acs      | saml_acs      | Aacotroneo\Saml2\Http\Controllers\Saml2Controller@acs      |
    saml2/login    | saml_login    | Aacotroneo\Saml2\Http\Controllers\Saml2Controller@login    |
    saml2/logout   | saml_logout   | Aacotroneo\Saml2\Http\Controllers\Saml2Controller@logout   |
    saml2/metadata | saml_metadata | Aacotroneo\Saml2\Http\Controllers\Saml2Controller@metadata |
    saml2/sls      | saml_sls      | Aacotroneo\Saml2\Http\Controllers\Saml2Controller@sls      |



### 4. SAML authentication setup inside Laravel

The "web" middleware group above automatically starts session management, functionality strictly required to perform authentication. A slightly different middleware group is required (see below).

1. add the following custom middleware group, to avoid issues related to VerifyCsrfToken middleware, in file **app/Http/Kernel.php** (as suggested [by SAML library's author](https://github.com/aacotroneo/laravel-saml2/issues/7)):

    ```php
    protected $middlewareGroups = [
        ...
        'web_for_saml' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        ],
        ...
    ];
    ```

1. customize SP and IdP metadata in file **config/saml2_settings.php**:

    ```php
    $idp_host = 'http://idp.example.com/simplesaml';
    
    return $settings = array(
        ...
        'routesMiddleware' => ['web_for_saml'],
        ...
        'sp' => array(
            ...
            'simplesaml.nameidattribute' => 'email',
            'x509cert' => file_get_contents('/path/to/certificate_folder/sp.example.com.crt'),
            'privateKey' => file_get_contents('/path/to/certificate_folder/sp.example.com.pem'),
            ...
        ),
        'idp' => array(
            ...
            'x509cert' => file_get_contents('/path/to/certificate_folder/idp.example.com.crt'),
        ),
        ...
    );
    ```
    this operation customizes the list of routes as follows:

    URI            | Name          | Action                                                     | Middleware
    ---------------|---------------|------------------------------------------------------------|-----------
    /              |               | Closure                                                    | web
    saml2/acs      | saml_acs      | Aacotroneo\Saml2\Http\Controllers\Saml2Controller@acs      | web_for_saml
    saml2/login    | saml_login    | Aacotroneo\Saml2\Http\Controllers\Saml2Controller@login    | web_for_saml
    saml2/logout   | saml_logout   | Aacotroneo\Saml2\Http\Controllers\Saml2Controller@logout   | web_for_saml
    saml2/metadata | saml_metadata | Aacotroneo\Saml2\Http\Controllers\Saml2Controller@metadata | web_for_saml
    saml2/sls      | saml_sls      | Aacotroneo\Saml2\Http\Controllers\Saml2Controller@sls      | web_for_saml

1. in order to bind local authentication session to remote authentication session, add SAML login/logout [event listeners](https://laravel.com/docs/5.2/events) into file **app/Providers/EventServiceProvider.php**:

    ```php
    public function boot(DispatcherContract $events)
    {

        parent::boot($events);

        $events->listen(
            'Aacotroneo\Saml2\Events\Saml2LoginEvent',
             function (Saml2LoginEvent $event) {
                 $user = $event->getSaml2User();
                 $laravelUser = User::where('email', $user->getUserId())->first();
                 if (empty($laravelUser)) {
             	       $laravelUser = User::create([
                         'email' => $user->getUserId(),
                     ]);
                 }
                 Auth::login($laravelUser);
             }
        );

        $events->listen(
            'Aacotroneo\Saml2\Events\Saml2LogoutEvent',
             function ($event) {
                 Auth::logout();
                 Session::save();
             }
        );
	
    }
    ```
    N.B.: for the scope of this tutorial, event listeners are hard-coded in file **app/Providers/EventServiceProvider.php**, but the most suitable solution would consist of two classes under directory **app/Listeners**
    
1. customize welcome view, in order to easily test login and logout, by adding login/logout links to file **resources/views/welcome.blade.php**:

    ```php
    <body>
        <div class="container">
            @if (Auth::guest())
                <a href="{{ route('saml2_login') }}">Login</a>
            @else
                <a href="{{ route('saml2_logout') }}">Logout</a>
            @endif
            ...
        </div>
    </body>
    ```



### 5. Connection to IdP finalization

1. export SP metadata in XML format, available at URL *http://sp.example.com/saml2/metadata*, e.g.:

    ```xml
<?xml version="1.0"?>
<md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata" validUntil="2017-03-09T20:35:05Z" cacheDuration="PT604800S" entityID="http://sp.example.com/saml2/metadata">
  <md:SPSSODescriptor AuthnRequestsSigned="false" WantAssertionsSigned="false" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    <md:KeyDescriptor use="signing">
      <ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
        <ds:X509Data>
          <ds:X509Certificate>...</ds:X509Certificate>
        </ds:X509Data>
      </ds:KeyInfo>
    </md:KeyDescriptor>
    <md:SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="http://sp.example.com/saml2/sls"/>
    <md:NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:persistent</md:NameIDFormat>
    <md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://sp.example.com/saml2/acs" index="1"/>
  </md:SPSSODescriptor>
  ...
</md:EntityDescriptor>
    ```

1. import SP metadata to IdP: IdP's usually have easy metadata management interfaces

### References

* https://en.wikipedia.org/wiki/Single_sign-on
* https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language
* https://laravel.com/docs/5.2/releases
* https://github.com/aacotroneo/laravel-saml2
* https://github.com/aacotroneo/laravel-saml2/issues/7
* https://simplesamlphp.org/docs/stable/simplesamlphp-sp
* https://laravel.com/docs/5.2/events
