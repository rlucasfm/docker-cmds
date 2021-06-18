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
        image: php:7.4-fpm-alpine
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
        # links:
        #     - php
        depends_on:
            - db     
            - php        

```
