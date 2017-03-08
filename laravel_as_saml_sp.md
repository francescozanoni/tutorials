#Laravel as SAML service provider

Some terms:

* IdP (Identity Provider): application providing authentication/authorization service
* SP (Service Provider): application that uses an IdP's authentication/authorization service

### 1. Laravel application installation

1. create project (for the scope of this tutorial, development dependencies are not required):

    ```bash
    composer create-project --prefer-dist laravel/laravel 5.2.* sp.example.net --no-dev
    ```

1. customize file *.env* as required (for the scope of this tutorial, only the following few parameters are required):

        APP_ENV=local
        APP_DEBUG=true
        APP_KEY=base64:jgTzCe6Iv1eYmCM57jmpzGnBeRBHfPmsGI1MXftjCAY=
        APP_URL=http://sp.example.net

        DB_HOST=my.db.host
        DB_DATABASE=my_database
        DB_USERNAME=my_username
        DB_PASSWORD=my_password

### 2. Users table customization

Since Laravel cannot handle authentication without a users table, create it:

1. simplify users table migration (password field is useless, with SAML authentication) by editing file *database/migrations/2014_10_12_000000_create_users_table.php*:

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

1. since users are managed by the IdP, password reset table migration can be removed:

    ```bash
    rm database/migrations/2014_10_12_100000_create_password_resets_table.php
    ```

1. run migration of customized users table:

    ```bash
    php artisan migrate
    ```

### 3. SAML library installation

1. install [SAML library](https://github.com/aacotroneo/laravel-saml2):

    ```bash
    composer require aacotroneo/laravel-saml2 --update-no-dev
    php artisan vendor:publish
    ```

1. add its service provider to file *app/config/app.php*:

    ```php
    'providers' => [
        ...
        Aacotroneo\Saml2\Saml2ServiceProvider::class,
    ]
    ```

1. create the SSL public certificate and related key, if not already available, and store them together with IdP's public certificate (e.g. idp.example.net.crt):

    ```bash
    openssl req -newkey rsa:2048 -new -x509 -days 3652 -nodes \
        -out /path/to/certificate_folder/sp.example.net.crt \
        -keyout /path/to/certificate_folder/sp.example.net.pem
    cp /path/to/idp.example.net.crt /path/to/certificate_folder
    ```

### 3. SAML authentication setup inside Laravel

1. add custom middleware group, to avoid issues related to VerifyCsrfToken middleware, in file *app/Http/Kernel.php* (as suggested [by SAML library's author](https://github.com/aacotroneo/laravel-saml2/issues/7)):

    ```php
    'web_for_saml' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
    ],
    ```

1. customize SP and IdP metadata in file *app/config/saml2_settings.php*:

    ```php
    $idp_host = 'http://idp.example.net/simplesaml';
    'routesMiddleware' => ['web_for_saml'],
    'simplesaml.nameidattribute' => 'mail',
    'x509cert' => file_get_contents('/path/to/certificate_folder/sp.example.net.crt'),
    'privateKey' => file_get_contents('/path/to/certificate_folder/sp.example.net.pem'),
    'x509cert' => file_get_contents('/path/to/certificate_folder/idp.example.net.crt'),
    ```

1. in order to bind local authentication session to remote authentication session, add SAML login/logout event listeners into file *app/Providers/EventServiceProvider.php*:

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
    N.B.: for the scope of this tutorial, event listeners are hard-coded in file *app/Providers/EventServiceProvider.php*, but the most suitable solution would consist of two classes under directory *app/Listeners*
    
1. customize welcome view, to display login/logout links, in file *app/resources/views/welcome.blade.php*:

    ```php
    @if (Auth::guest())
        <a href="{{ route('saml2_login') }}">Login</a>
    @else
        <a href="{{ route('saml2_logout') }}">Logout</a>
    @endif
    ```

### 4. Connection to IdP finalization

1. add SP metadata to IdP; SP metadata in XML format are available at URL *http://sp.example.net/saml2/metadata*:

    ```xml
<?xml version="1.0"?>
<md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata" validUntil="2017-03-09T20:35:05Z" cacheDuration="PT604800S" entityID="http://sp.example.net/saml2/metadata">
  <md:SPSSODescriptor AuthnRequestsSigned="false" WantAssertionsSigned="false" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    <md:KeyDescriptor use="signing">
      <ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
        <ds:X509Data>
          <ds:X509Certificate>MII[...]</ds:X509Certificate>
        </ds:X509Data>
      </ds:KeyInfo>
    </md:KeyDescriptor>
    <md:SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="http://sp.example.net/saml2/sls"/>
    <md:NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:persistent</md:NameIDFormat>
    <md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://sp.example.net/saml2/acs" index="1"/>
  </md:SPSSODescriptor>
  <md:Organization>
    <md:OrganizationName xml:lang="en-US">Name</md:OrganizationName>
    <md:OrganizationDisplayName xml:lang="en-US">Display Name</md:OrganizationDisplayName>
    <md:OrganizationURL xml:lang="en-US">http://url</md:OrganizationURL>
  </md:Organization>
  <md:ContactPerson contactType="technical">
    <md:GivenName>name</md:GivenName>
    <md:EmailAddress>no@reply.com</md:EmailAddress>
  </md:ContactPerson>
  <md:ContactPerson contactType="support">
    <md:GivenName>Support</md:GivenName>
    <md:EmailAddress>no@reply.com</md:EmailAddress>
  </md:ContactPerson>
</md:EntityDescriptor>

    ```

### References

* https://github.com/aacotroneo/laravel-saml2
* https://github.com/aacotroneo/laravel-saml2/issues/7
* https://simplesamlphp.org/docs/stable/simplesamlphp-sp