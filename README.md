# Containerized PHP application for Laravel Development

## What is this?

This an example to how using Docker as development environment for Laravel/Lumen.

## What is included

This docker-compose contains the basic pieces to build an applications with PHP.
 - PHP7
 - Nginx or Caddy
 - Postgres
 - Redis

## How to use

### 1. Get the files and spin up containers

```bash
# Get shipping-docker files
git clone https://github.com/petronetto/laravel-docker.git
cd laravel-docker

# Start the app, run containers in the background
# This will download and build the images the first time you run this
docker-compose up -d
```

At this point, we've created containers and have them up and running. However, we didn't create a Laravel application to serve yet. We waited because we wanted a PHP image to get created so we can re-use it and run `composer` commands.

### 2. Create a new Laravel application

> **Note:** You need delete the files in path `app` before run the command bellow.

From directory `laravel-docker` the following command to create the Laravel app:
```bash
docker run -it --rm \
           -v $(pwd)/app:/app \
           -w /app \
           petronetto/php7-alpine \
           composer create-project laravel/laravel .
```

Now, install Redis driver:
```sh
docker run -it --rm \
           -v $(pwd)/app:/app \
           -w /app \
           petronetto/php7-alpine \
           composer require predis/predis
```

> In Linux, maybe you need run the follow ommand to set directory ownership to you `sudo chown -R $USER:$USER .`

Edit the `app/.env` file to have correct settings for our containers. Adjust the following as necessary:

```
DB_CONNECTION=pgsql
DB_HOST=database
DB_PORT=5432
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret

BROADCAST_DRIVER=log
CACHE_DRIVER=redis
SESSION_DRIVER=redis
SESSION_LIFETIME=120
QUEUE_DRIVER=redis

REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379
```

> If you already have an application, you can move it to the `app` directory here. Else, you can adjust the shared volume file paths within the `docker-compose.yml` file.
> 
> If you edit the `docker-compose.yml` file, run `docker-compose down; docker-compose up -d` to suck in the new Volume settings.

**NOTE**: If you're not running Docker Mac/Windows (which run Docker in a small virtualized layer), you may need to set permissions on the shared directories that Laravel needs to write to. The following will let Laravel write the storage and bootstrap directories:

```bash
# From directory laravel-docker
sudo chmod -R o+rw app/bootstrap app/storage
```

### 3. (Optionally) Add Auth Scaffolding:

If you'd like, we can add Laravel's Auth scaffolding as well. To do that, we need to run some Artisan commands:

```bash
# Scaffold authentication views/routes
docker exec -it php-fpm php artisan make:auth

# Run migrations for auth scaffolding
docker exec -it php-fpm php artisan migrate
```

## Remote debug

The container is configured to run the file that bootstrap applications in folder `/app/public`, because I use Laravel/Lumen by default, so, if you don't use a framework that bootstrap the application in this folder, you must put your source files there.
To the debug works, you must:

1) Set your local IP address in `XDEBUG_HOST` environment variable in `docker-compose.yml`.

2) Configure your editor/IDE to map the local source and the remote source.

### VS Code
Install [PHP Debug Extension](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug).

Put this config in your `launch.json`:

>**NOTE** this example will map the local folder `app`, to `/app` on the container. Remenber that container is configured to boostrap in `/app/public`, as the example provided in this repo.

```json
//launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "XDebug",
            "type": "php",
            "request": "launch",
            "port": 9000,
            "pathMappings": {
                "/app": "${workspaceRoot}"
            },
            "ignore": ["**/vendor/**/*.php"]
        }
    ]
}
```

Check the Sublime and PHPStorm configs [here](https://github.com/petronetto/php7-alpine#remote-debug)


Enjoy!
