# Stack de Laravel + PHP 8 + NGINX + PostgreSQL + pgAdmin

Docker-compose para containerizar uma aplicação laravel

```yml
version: "3.9"

services:
    db:    
        image: postgres
        volumes:
        - ./data/db:/var/lib/postgresql/data
        environment:
        - POSTGRES_DB=grupovoz
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=root
        ports:
        - '5433:5432'

    php:        
        build: ./php-docker        
        volumes: 
           - ./webapp:/webapp

    pgadmin:
        image: dpage/pgadmin4
        environment:
            PGADMIN_DEFAULT_EMAIL: "admin@admin.com"
            PGADMIN_DEFAULT_PASSWORD: "admin"
        ports:
            - "16543:80"
        depends_on:
            - db

    nginx:
        image: nginx:latest
        ports:
            - "8080:80"
        volumes:
            - ./webapp:/webapp
            - ./site.conf:/etc/nginx/conf.d/default.conf
        depends_on:
            - db     
            - php           
```

## Dockerfile para o PHP
```Dockerfile
FROM php:8.0.2-fpm-alpine

RUN mkdir -p /webapp/storage

RUN chown -R $(whoami):www-data /webapp/storage && chmod -R ug+w /webapp/storage
RUN apk --no-cache add postgresql-dev
RUN docker-php-ext-install pdo pdo_pgsql
```

## Erro de permissão de escrita
Caso esteja dando o erro de permissão de escrita, basta rodar os comandos (ou no CLI ou na montagem da imagem), dentro da pasta do laravel, na imagem do PHP:

```sh
chown -R $(whoami):www-data storage
sudo chmod -R ug+w storage
php artisan cache:clear
composer dump-autoload
```

## Instalação das extensões para o PostgreSQL
Rodar os comandos na imagem do PHP (CLI ou montagem da imagem):
```sh
-ex \
  && apk --no-cache add \
    postgresql-dev

docker-php-ext-install pdo pdo_pgsql
```

## CLI com o container em execução
Para abrir uma "interface CLI" com o container já aberto, por exemplo, para rodar a instalação das extensões no container do PHP:
```
sh
docker exec -it docker-release_php_1 sh
```
