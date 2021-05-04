
# NPMP  Nginx Php Mysql Phpmyadmin
███████████████████████████████████████████████████████████████
█─▄▄▄▄██▀▄─██▄─█─▄█▄─▄███▄─█─▄████▀▄─██▄─█─▄█─▄▄─█▄─██─▄█▄─▄─▀█
█▄▄▄▄─██─▀─███─▄▀███─██▀██▄─▄█████─▀─███▄─▄██─██─██─██─███─▄─▀█
▀▄▄▄▄▄▀▄▄▀▄▄▀▄▄▀▄▄▀▄▄▄▄▄▀▀▄▄▄▀▀▀▀▄▄▀▄▄▀▀▄▄▄▀▀▄▄▄▄▀▀▄▄▄▄▀▀▄▄▄▄▀▀

Docker running Nginx, PHP-FPM, Composer, MySQL and PHPMyAdmin.
## Install prerequisites

To run the docker commands without using **sudo** you must add the **docker** group to **your-user**:

```
sudo usermod -aG docker your-user
```

For now, this project has been mainly created for Unix `(Linux/MacOS)`. Perhaps it could work on Windows.

All requisites should be available for your distribution. The most important are :

* [Git](https://git-scm.com/downloads)
* [Docker](https://docs.docker.com/engine/installation/)
* [Docker Compose](https://docs.docker.com/compose/install/)

Check if `docker-compose` is already installed by entering the following command : 

```sh
which docker-compose
```

Check Docker Compose compatibility :

* [Compose file version 3 reference](https://docs.docker.com/compose/compose-file/)

The following is optional but makes life more enjoyable :

```sh
which make
```

On Ubuntu and Debian these are available in the meta-package build-essential. On other distributions, you may need to install the GNU C++ compiler separately.

```sh
sudo apt install build-essential
```

### Images to use

* [Nginx](https://hub.docker.com/_/nginx/)
* [MySQL](https://hub.docker.com/_/mysql/)
* [PHP-FPM](https://hub.docker.com/_/php)
* [Composer](https://hub.docker.com/_/composer/)
* [PHPMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)
* [Generate Certificate](https://hub.docker.com/r/jacoelho/generate-certificate/)

You should be careful when installing third party web servers such as MySQL or Nginx.

This project use the following ports :

| Server     | Port |
|------------|------|
| MySQL      | 3306 |
| PHPMyAdmin | 8080 |
| Nginx      |   80 |
| Nginx SSL  |  443 |

## Clone the project

To install [Git](http://git-scm.com/book/en/v2/Getting-Started-Installing-Git), download it and install following the instructions :

```sh
git clone https://github.com/saklyayoub/docker-npmp.git
```

Go to the project directory :

```sh
cd docker-nginx-php-mysql
```

### Project tree

```sh
├── docker
│   ├── Dockerfile-7.0FPM
│   ├── Dockerfile-7.4FPM
│   └── Dockerfile-latest
├── docker-compose.yml
├── etc
│   ├── nginx
│   │   ├── default.template.conf
│   │   ├── nginx.conf
│   │   ├── nginx.http.conf
│   │   └── nginx.https.conf
│   ├── php
│   │   └── php.ini
│   └── ssl
├── README.md
└── web
    ├── mysql
    |   ├── backups
    |   |   └── sample.sql
    |   └── init
    |       └── sample.sql
    ├── app
    │   ├── composer.json
    │   ├── composer.json.dist
    │   ├── phpunit.xml.dist
    │   ├── src
    │   │   └── Foo.php
    │   └── test
    │       ├── bootstrap.php
    │       └── FooTest.php
    └── public
        └── index.php
```
## Configure Nginx With SSL Certificates

You can change the host name by editing the `.env` file.

If you modify the host name, do not forget to add it to the `/etc/hosts` file.

1. Generate SSL certificates

    ```sh
    source .env && docker run --rm -v $(pwd)/etc/ssl:/certificates -e "SERVER=$NGINX_HOST" jacoelho/generate-certificate
    ```

2. Configure Nginx

    Do not modify the `etc/nginx/default.conf` file, it is overwritten by  `etc/nginx/default.template.conf`

## Run the application

1. Copying the composer configuration file : 

    ```sh
    cp web/app/composer.json.dist web/app/composer.json
    ```

2. Start the application :

    ```sh
    docker-compose up -d
    ```

    **Please wait this might take a several minutes...**

    ```sh
    docker-compose logs -f # Follow log output
    ```

3. Stop and clear services

    ```sh
    docker-compose down -v
    ```
    
## Use Docker commands

### Installing package with composer

```sh
docker run --rm -v $(pwd)/web/app:/app composer require symfony/yaml
```

### Updating PHP dependencies with composer

```sh
docker run --rm -v $(pwd)/web/app:/app composer update
```

### Generating PHP API documentation

```sh
docker run --rm -v $(pwd):/data phpdoc/phpdoc -i=vendor/ -d /data/web/app/src -t /data/web/app/doc
```

### Testing PHP application with PHPUnit

```sh
docker exec $(docker-compose ps -q php) php ./app/vendor/bin/phpunit --colors=always --configuration ./public
```

### Fixing standard code with [PSR2](http://www.php-fig.org/psr/psr-2/)

```sh
docker exec $(docker-compose ps -q php) php ./app/vendor/bin/phpcbf -v --standard=PSR2 ./public
```

### Checking the standard code with [PSR2](http://www.php-fig.org/psr/psr-2/)

```sh
docker exec $(docker-compose ps -q php) php ./app/vendor/bin/phpcs -v --standard=PSR2 ./public
```

### Analyzing source code with [PHP Mess Detector](https://phpmd.org/)

```sh
docker exec $(docker-compose ps -q php) php ./app/vendor/bin/phpmd ./public text cleancode,codesize,controversial,design,naming,unusedcode
```

### Checking installed PHP extensions

```sh
docker-compose exec php php -m
```

### Handling database

#### MySQL shell access

```sh
docker exec -it mysql bash
```

and

```sh
mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD"
```

#### Creating a backup of all databases

```sh
mkdir -p data/db/dumps
```

```sh
source .env && docker exec $(docker-compose ps -q mysql) mysqldump --all-databases -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" > "/web/mysql/backups/db.sql"
```

#### Restoring a backup of all databases

```sh
source .env && docker exec -i $(docker-compose ps -q mysql) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" < "/web/mysql/backups/db.sql"
```

#### Creating a backup of single database

**`Notice:`** Replace "YOUR_DB_NAME" by your custom name.

```sh
source .env && docker exec $(docker-compose ps -q mysql) mysqldump -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" --databases YOUR_DB_NAME > "/web/mysql/backups/YOUR_DB_NAME_dump.sql"
```

#### Restoring a backup of single database

```sh
source .env && docker exec -i $(docker-compose ps -q mysql) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" < "/web/mysql/backups/YOUR_DB_NAME_dump.sql"
```
