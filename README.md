[![workflow yamdb_final](https://github.com/Nemets87/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg?branch=master)](https://github.com/Nemets87/yamdb_final/actions/workflows/yamdb_workflow.yml)
-
![example workflow](https://myoctocat.com/assets/images/base-octocat.svg)
-
Коты рулят !

# Проект **yamdb_final** 

#### **Django & Docker Power + CI и CD проекта api_yamdb = 16 Sprint**
![docker power](https://img.shields.io/docker/automated/nemets87/infra_sp2)
![django version](https://img.shields.io/badge/Django-2.2-green)

Api собирает отзывы пользователей на различные произведения.

# Как запустить проект чeрез Docker:
Должен быть уставлен Docker https://www.docker.com

Клонируем репозиторий и переходим в него:

```
git@github.com:Nemets87/yamdb_final.git

```
Перейти в папку infra

```
cd infra
```

в папке infra создаем файл .env с следующим содержимом:

```
DB_ENGINE=django.db.backends.postgresql = указываем, что работаем с postgresql
DB_NAME=postgres = имя базы данных
POSTGRES_USER=postgres = логин для подключения к базе данных
POSTGRES_PASSWORD=postgres = пароль для подключения к БД (установите свой)
DB_HOST=db = название сервиса (контейнера)
DB_PORT=5432 = порт для подключения к БД 
```
- проверяем requirements.txt (должен содержать необходимый минимум,важно наличие самих пакетов, а не версии)

```
gunicorn==20.0.4
django==2.2.16
pytest-django==4.4.0
requests==2.26.0
djangorestframework==3.12.4
PyJWT==2.1.0
pytest==6.2.4
pytest-pythonpath==0.7.3
django-filter==2.4.0
djangorestframework-simplejwt==4.8.0
asgiref==3.2.10
psycopg2-binary==2.8.6
pytz==2020.1
sqlparse==0.3.1
python-dotenv==0.21
```
Работать мы будем в linux и все команды начинаются со слова sudo
P.S в Windows sudo не нужно  
```
sudo su root дает возможность избежать слово sudo,но под рутом сидеть опасно 
- создаем новый ВМ с ubuntu 20.04 (НЕ 22.04!!!!)

- заходим на сервер

nemets87
ssh nemets87@158.160.21.17

- ставим докер и композ
sudo apt update
sudo apt install docker.io
sudo apt install docker-compose
sudo systemctl start docker

- зайти в свой проект на компе, проверить докерфайл
```
Собрать проект (в папке с файлом docker-compose.yaml:)
зайти в свой проект на компе, проверить докерфайл
```
FROM python:3.7-slim
WORKDIR /app
COPY requirements.txt .
RUN pip3 install -r requirements.txt --no-cache-dir
COPY . .
CMD ["gunicorn", "api_yamdb.wsgi:application", "--bind", "0:8000"]
```

перед созданием образа лучше удалить предыдущие варианты образа с ДокерХаб. На ДХ - Settings - Delete repository.

```
- создать образ (находиться в папке с докерфайлом)
docker build -t <ваш_логин_докерхаб>/<название_образа_придумать> .

- пушить образ
docker login -u <ваш_логин_докерхаб>
docker push <ваш_логин_докерхаб>/<название_образа_придумать>:v1 


- снова правим докерфайл
```
- проверить воркфлоу, потом скопировать в корень проекта
```
name: workflow yamdb_final

on: [push]

jobs:
  tests:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: 3.7

    - name: Install dependencies
      run: | 
        python -m pip install --upgrade pip 
        pip install flake8 pep8-naming flake8-broken-line flake8-return flake8-isort
        pip install -r api_yamdb/requirements.txt 
        
    - name: Test with flake8 and django tests
      run: |
        python -m flake8
        pytest
        
  build_and_push_to_docker_hub:
      name: Push Docker image to Docker Hub
      runs-on: ubuntu-latest
      needs: tests
      if: github.ref == 'refs/heads/master'
      steps:
        - name: Check out the repo
          uses: actions/checkout@v2
        - name: Set up Docker Buildx
          uses: docker/setup-buildx-action@v1
        - name: Login to Docker
          uses: docker/login-action@v1
          with:
            username: ${{ secrets.DOCKER_USERNAME }}
            password: ${{ secrets.DOCKER_PASSWORD }}
        - name: Push to Docker Hub
          uses: docker/build-push-action@v2
          with:
            context: ./api_yamdb/ # а без этой строчки = будут бессонные ночки !
            push: true
            tags: nemets87/api_yamdb:v1 # можно через секрет = но секрета нет !

  deploy:
      runs-on: ubuntu-latest
      needs: build_and_push_to_docker_hub
      if: github.ref == 'refs/heads/master'
      steps:
      - name: executing remote ssh commands to deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USER }}
          key: ${{ secrets.SSH_KEY }}
          passphrase: ${{ secrets.PASSPHRASE }}
          script: |
            sudo docker-compose stop
            sudo docker-compose rm web
            touch .env
            echo DB_ENGINE=${{ secrets.DB_ENGINE }} >> .env
            echo DB_NAME=${{ secrets.DB_NAME }} >> .env
            echo POSTGRES_USER=${{ secrets.POSTGRES_USER }} >> .env
            echo POSTGRES_PASSWORD=${{ secrets.POSTGRES_PASSWORD }} >> .env
            echo DB_HOST=${{ secrets.DB_HOST }} >> .env
            echo DB_PORT=${{ secrets.DB_PORT }} >> .env
            sudo docker-compose up -d 
```
докер хаб образ

```
https://hub.docker.com/repository/docker/nemets87/api_yamdb/general
```

проект доступен по адресу 

```
http://158.160.21.17/admin/

http://158.160.21.17/redoc/

```
- скопируйте файлы docker-compose.yaml и nginx/default.conf на сервер
```
P- выполнить команды на сервере
sudo docker-compose exec web python manage.py migrate 
(возможно понадобиться сделать makemigrations reviews)
sudo docker-compose exec web python manage.py createsuperuser
sudo docker-compose exec web python manage.py collectstatic --no-input 
sudo docker-compose exec web python manage.py migrate 
```

**Р- проверили сайт по полной, что админка и ридок нормально работает**
# Примеры запросов
## Получение списка всех категорий

```
cd ~
touch docker-compose.yml
nano docker-compose.yml
копировать содержимое файла на локальном компе и вставить в файл на сервере

cd ~
mkdir nginx
cd nginx
touch default.conf
nano default.conf
копировать содержимое файла на локальном компе и вставить в файл на сервере
```
## ВМ не крутим целый год == на диплом ее == на диплом 

```
- если все хорошо, то легкая проверка сайта, что действительно запустилось.
Если не запускается IP/redoc, то скорей всего этот файл не попал на сервер.
Можно добавить его туда вручную:
1. Локально: 
scp redoc.yaml  <имя_на_сервере>@<публичный_IP>:~
(закинуть на сервер)
2. На сервере:
sudo docker cp redoc.yaml <id_container>:/app/static

Если локально не удается скопировать файл на сервер, можно его создать прямо на сервере:
1. sudo nano redoc.yaml
 - скопировать в него данные из локального redoc.yaml
2. Перенести файл в нужную папку:
sudo docker cp redoc.yaml <id_container>:/app/static
```


### **Авторы:**
- [Sergey Fedorov + Power 45 Team ](https://github.com/Nemets87)

### **Создан и проверен на: **
- [Linux Mint 21.1 "Vera" ](https://linuxmint.com/download.php)
