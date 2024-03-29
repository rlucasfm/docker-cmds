# Comandos DOCKER e Docker-compose - Anotações

## Criar um Dockerfile
Cria um dockerfile com a imagem do python 3 e instala as dependências em requirements.txt com um requirements.txt do tipo:
```txt
Django>=3.0,<4.0
psycopg2-binary>=2.8
```

```dockerfile
# syntax=docker/dockerfile:1 
FROM python:3 
ENV PYTHONUNBUFFERED=1
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt
COPY . /code
```

## Criar um docker-compose.yml
Descreve os serviços que serão usados e suas imagens e configurações. Aqui configuramos o "db" com uma imagem do Postgres, e o serviço web para executar o comando do Django.
```yml
version: "3.9"
   
services:
  db:
    image: postgres
    volumes:
      - ./data/db:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    ports:
      - '5433:5432'

  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    depends_on:
      - db

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: "richardlucasfm@gmail.com"
      PGADMIN_DEFAULT_PASSWORD: "admin"
    ports:
      - "16543:80"
    depends_on:
      - db
```

## Executando comandos Django através do Docker
Iremos executar o comando de criar um projeto django a partir do docker.
```bash
docker-compose run web django-admin startproject composeexample .
```

## Executando uma CLI dentro do container
Para abrir um CLI dentro do container e executar comandos dentro do ambiente.
Primeiramente, encontrar o ID do container desejado.
````bash
docker ps
```
```bash
docker exec -it CONTAINER_ID /bin/bash
```

## Executar o docker-composer
Aqui iremos executar os serviços descritos no .yml

```bash
docker-compose up
```
