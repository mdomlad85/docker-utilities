# Laravel Util

## Start a server

To start a laravel server run the command below. 
Build flag is added to check if there is any change 
in docker files to make a new build. 
If not it will be very fast as it is cached.

``docker-compose up -d --build server``

## Utils

### Composer

``docker-compose run --rm composer [any composer command]``

#### Laravel

If you want to install laravel all over again 
delete everything in src folder, update versions 
(php, whatever you want to use) and issue composer command:

``docker-compose run --rm composer create-project --prefer-dist laravel/laravel .``

### Artisan

``docker-compose run --rm artisan migrate``
