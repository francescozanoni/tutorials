
create project

    composer create-project --prefer-dist laravel/laravel 5.2.* sp.example.net --no-dev
    cd sp.example.net

customize .env

    APP_ENV=local
    APP_DEBUG=true
    APP_KEY=base64:jgTzCe6Iv1eYmCM57jmpzGnBeRBHfPmsGI1MXftjCAY=
    APP_URL=http://sp.example.net
    DB_CONNECTION=sqlite
    DB_DATABASE=/absolute/path/to/database/database.sqlite

run migration of custom users table

    touch database/database.sqlite
    rm database/migrations/2014_10_12_100000_create_password_resets_table.php

simplify users table (password field is useless, with SAML authentication)

database/migrations/2014_10_12_000000_create_users_table.php
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('email')->unique();
            $table->rememberToken();
            $table->timestamps();
        });
    }

    php artisan migrate

# install SAML dependency
composer require aacotroneo/laravel-saml2 --update-no-dev
php artisan vendor:publish
app/config/app.php:
'providers' => [
    ...
    Aacotroneo\Saml2\Saml2ServiceProvider::class,
]

#openssl req -newkey rsa:2048 -new -x509 -days 3652 -nodes -out sp.example.net.crt -keyout sp.example.net.pem
# cp /path/to/idp.example.net.crt cert

# add SP metadata to IdP

# customize SP and IdP metadata
app/config/saml2_settings.php
$idp_host = 'http://idp.example.net/simplesaml';
'routesMiddleware' => ['web_for_saml'],
'simplesaml.nameidattribute' => 'mail',
'x509cert' => file_get_contents('/path/to/sp.example.net.crt'),
'privateKey' => file_get_contents('/path/to/sp.example.net.pem'),
'x509cert' => file_get_contents('/path/to/idp.example.net.crt'),

# add custom middleware group
# @see https://github.com/aacotroneo/laravel-saml2/issues/7
app/Http/Kernel.php
		 'web_for_saml' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        ],

# SAML2 login/logout event listeners
app/Providers/EventServiceProvider.php
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
        });


        $events->listen(
        	'Aacotroneo\Saml2\Events\Saml2LogoutEvent',
        	function ($event) {
            Auth::logout();
            Session::save();
        });
    }
    
# welcome view customization, to display login/logout links
app/resources/views/welcome.blade.php
@if (Auth::guest())
    <a href="{{ route('saml2_login') }}">Login</a>
@else
    <a href="{{ route('saml2_logout') }}">Logout</a>
@endif