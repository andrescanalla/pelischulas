## PelisChulis

### Installation 
1. To install this app clone the repo 

2. Install the dependencies
```
composer install
npm install
gulp
```
3. After that, setup db connection in `.env` file
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```
4. Now Run the seed command to setup the dummy data
```
php artisan db:seed
```
5. App is ready you can run `php artisan serve` and run the demo
----
