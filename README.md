# Containerized PHP application for Laravel Development

## What is this?

This is an containerized PHP application for Laravel Development, the PHP container is based on a very lightweight container ([phusion/baseimage-docker](http://phusion.github.io/baseimage-docker/)), that is a minimal Ubuntu base image modified for Docker-friendliness. _Baseimage-docker **only consumes 6 MB RAM** and is much powerful than Busybox or Alpine._

## What is included

This docker-compose contains the basic pieces to build an applications with PHP.
 - PHP7 with Nginx
 - Debug container
 - MySQL 5.7
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

Access: http://localhos:8080 or http://localhos:8080/test.html

> There's a `.env` file where is placed some configs, like container names, 
> MySQL ports and etc...Change it as you needed. 

At this point, we've created containers and have them up and running. However, we didn't create a Laravel application to serve yet. We waited because we wanted a PHP image to get created so we can re-use it and run `composer` commands.

### 2. Create a new Laravel application

> **Note:** You need delete the files in path `src` before run the command bellow.

```bash
# From directory "laravel-docker"
# Create a Laravel application
docker run -it --rm \
           -v $(pwd)/src:/var/www/src \
           -w /var/www/src \
           petronetto/php-nginx \
           composer create-project laravel/laravel .

docker run -it --rm \
           -v $(pwd)/src:/var/www/src \
           -w /var/www/src \
           petronetto/php-nginx \
           composer require predis/predis

# Restart required to ensure
# app files shares correctly
docker-compose restart

# In Linux, maybe you need run the follow
# command to set directory ownership to you
sudo chown -R $USER:$USER . 
```

Edit the `src/.env` file to have correct settings for our containers. Adjust the following as necessary:

```
DB_CONNECTION=mysql
DB_HOST=docker-database
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret

BROADCAST_DRIVER=log
CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_DRIVER=sync

REDIS_HOST=docker-redis
REDIS_PASSWORD=null
REDIS_PORT=6379
```

> If you already have an application, you can move it to the `src` directory here. Else, you can adjust the shared volume file paths within the `docker-compose.yml` file.
> 
> If you edit the `docker-compose.yml` file, run `docker-compose down; docker-compose up -d` to suck in the new Volume settings.

**NOTE**: If you're not running Docker Mac/Windows (which run Docker in a small virtualized layer), you may need to set permissions on the shared directories that Laravel needs to write to. The following will let Laravel write the storage and bootstrap directories:

```bash
# From directory laravel-docker
sudo chmod -R o+rw src/bootstrap src/storage
```

### 3. (Optionally) Add Auth Scaffolding:

If you'd like, we can add Laravel's Auth scaffolding as well. To do that, we need to run some Artisan commands:

```bash
# Scaffold authentication views/routes
docker exec -it docker-php php artisan make:auth

# Run migrations for auth scaffolding
docker exec -it docker-php php artisan migrate
```

## Running the Debug container

The Debug container is configured to run the file that bootstrap applications in folder `/var/www/html/public`, because I use Laravel by default, so, if you don't use a framework that bootstrap the application in this folder, you must put your source files there.
To the debug works, you must:

1) Set your IP address in `DEBUG_HOST` variable in`.env` file. Optionaly, you can change the debug port using `DEBUG_PORT`, by default will run in port `8989`.

Sample configuration:
```
# .env file
DEBUG_HOST=192.168.25.2 # your local ip address
DEBUG_PORT=8989 # the port to run the debug eg.: http://localhost:8989
```

2) Configure your editor/IDE tor map where is the local source and where is the remote source.

To debug in VS Code, just put this config in your lauch.json:
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "XDebug",
            "type": "php",
            "request": "launch",
            "port": 9000,
            // Path to your source in container
            "serverSourceRoot": "/var/www/html",
            // Path to your local source
            "localSourceRoot": "${workspaceRoot}/src"
        }
    ]
}
```

Check the Sublime and PHPStorm configs [here](https://github.com/Petronetto/php-debug)


Enjoy!
