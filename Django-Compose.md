# Stack com Django (Python 3) + PostgreSQL + pgAdmin

Dockerfile para Django

```Dockerfile
# syntax=docker/dockerfile:1
FROM python:3
ENV PYTHONUNBUFFERED=1
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt
COPY . /code/
```

Docker-compose.yml para o Django
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
    # command: python manage.py runserver 0.0.0.0:8000
    command: gunicorn --bind 0.0.0.0:8000 vozsms.wsgi:application
    volumes:
      - .:/code      
    # ports:
    #   - "8000:8000"
    expose:
      - 8000
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

  nginx:
    build: ./nginx
    ports:
      - 8000:80
    volumes:
      - ./staticfiles:/home/web/static
    depends_on:
      - web
```

No ```settings.py```, para configurar o banco de dados:
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'PASSWORD': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
    }
}
```

## Configurando Gunicorn + Nginx
A configuração do gunicorn é relativamente simples, bastando colocá-lo no ```requirements.txt``` e trocar o comando do container web de:
```yml
#docker-compose.yml
# De:
command: python manage.py runserver 0.0.0.0:8000
# Para:    
command: gunicorn --bind 0.0.0.0:8000 vozsms.wsgi:application
```

A configuração do Nginx por sua vez é um pouco mais complexa. Primeiro criar um diretório com o nome "nginx" na raiz, dentro deste um ```Dockerfile``` e um ```nginx.conf```.
```Dockerfile
FROM nginx:1.19.0-alpine

RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d
```
O Dockerfile irá criar a imagem, e copiar o arquivo de configuração para dentro da mesma.
```conf
server {
    listen 80;

    location / {
        proxy_pass http://web:8000;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

    location /static {
        root /home/web/;
    }

}
```
O arquivo de configuração irá apontar para a instância do gunicorn rodando na porta 8000 (configurado no docker-compose.yml) e irá porém direcionar as requisições de "/static" para uma pasta de arquivos estáticos dentro do container do Nginx.
```yml
#docker-compose.yml

nginx:
    build: ./nginx
    ports:
      - 8000:80
    volumes:
      - ./staticfiles:/home/web/static
    depends_on:
      - web
```
O ```volumes``` irá relacionar a pasta de arquivos estáticos da máquina host para dentro do container.
