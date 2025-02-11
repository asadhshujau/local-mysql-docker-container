# Guide

1. Create a docker network called **database**
```bash
docker network create database
```
2. If not already made, make 2 directories named **mysql5** and **mysql8** in this project's root folder
```bash
mkdir mysql5
mkdir mysql8
```
3. Start the container
```bash
docker compose up -d
```
4. If all goes well you'll have 2 running containers now. You'll get a similar output when you run **docker ps**
```bash
docker ps

CONTAINER ID   IMAGE               COMMAND                  CREATED             STATUS             PORTS                                                                      NAMES
2d827c39ea9a   mysql:8.4           "docker-entrypoint.s…"   About an hour ago   Up About an hour   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp                       mysqlserverviii
369e2f755ddf   mysql:5.7           "docker-entrypoint.s…"   About an hour ago   Up About an hour   33060/tcp, 0.0.0.0:33060->3306/tcp, [::]:33060->3306/tcp                   mysqlserverv
```
5. Using something like mysql-workbench or dbeaver, you can connect to
    - 5.1. **mysql 8.4** from your **localhost** on port **3306**
    - 5.2. **mysql 5.7** from your **localhost** on port **33060**
6. To use it in another project add the network "**database**" to your service and add it to the bottom of your **docker-compose.yml** file
```yml
services:
    laravel.test:
        build:
            context: ./vendor/laravel/sail/runtimes/8.2
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        image: sail-8.2/app
        ports:
            - '${APP_PORT:-80}:80'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
            IGNITION_LOCAL_SITES_PATH: '${PWD}'
        volumes:
            - '.:/var/www/html'
        networks:
            - sail
            - database      # add it here
networks:
    sail:
        driver: bridge
    database:           # add the network as external network here
        external: true
```
7. Now to connect to the database add it to .env. For example in a laravel project
```bash
# .env

...

DB_CONNECTION=mysql

# use service name mysqlserverviii for mysql 8
DB_HOST=mysqlserverviii
# or use service name mysqlserverv for mysql 5
DB_HOST=mysqlserverv

DB_PORT=3306
DB_DATABASE=lms
DB_USERNAME=root
DB_PASSWORD=secret

...

```
8. Start your project and for example migrate your database. Here's an example with Laravel Sail
```bash
-> sail up -d

[+] Running 1/0
 ✔ Container lms-laravel.test-1  Running

-> sail artisan migrate

   WARN  The database 'lms' does not exist on the 'mysql' connection.

 ┌ Would you like to create it? ────────────────────────────────┐
 │ Yes                                                          │
 └──────────────────────────────────────────────────────────────┘

    INFO  Preparing database.

  Creating migration table ............................................................................................................... 14ms DONE

   INFO  Running migrations.

  2014_10_12_000000_create_users_table ................................................................................................... 86ms DONE
```

# Import and Export Database

Dump database from docker MySQL container
```
docker exec -i container_name mysqldump -uUSERNAME -p DBNAME > where/to/dump.sql
```

Import database to docker MySQL container
```
docker exec -i container_name mysql -uUSERNAME -pPASSWORD DBNAME < db.sql
```
