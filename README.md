[![workflow yamdb_final](https://github.com/Nemets87/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg?branch=master)](https://github.com/Nemets87/yamdb_final/actions/workflows/yamdb_workflow.yml)
-
![example workflow](https://myoctocat.com/assets/images/base-octocat.svg)
-
Коты рулят !

# Проект **YaMDb** 

#### **Django & Docker Power**
![docker power](https://img.shields.io/docker/automated/nemets87/infra_sp2)
![django version](https://img.shields.io/badge/Django-2.2-green)

Api собирает отзывы пользователей на различные произведения.

# Как запустить проект чeрез Docker:
Должен быть уставлен Docker https://www.docker.com

Клонируем репозиторий и переходим в него:

```
git clone https://github.com/Nemets87/infra_sp2

```
Перейти в папку infra

```
cd infra
```

Создать файл ".env"

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
asgiref==3.2.10
Django==2.2.16
django-filter==2.4.0
djangorestframework==3.12.4
djangorestframework-simplejwt==4.8.0
gunicorn==20.0.4
psycopg2-binary==2.8.6
PyJWT==2.1.0
pytz==2020.1
sqlparse==0.3.1
```
Работать мы будем в linux и все команды начинаются со слова sudo
P.S в Windows sudo не нужно  
```
sudo su root дает возможность избежать слово sudo,но под рутом сидеть опасно 
```
Собрать проект (в папке с файлом docker-compose.yaml:)
```
sudo docker-compose up -d --build
```

Cделать миграции, создать суперпользователя и собрать статику 

```
sudo docker-compose exec web python manage.py makemigrations reviews
sudo docker-compose exec web python manage.py migrate 
sudo docker-compose exec web python manage.py createsuperuser 
sudo docker-compose exec web python manage.py collectstatic --no-input 
```
Создаем дамп базы данных (нет их сейчас,но будут):
```
sudo docker-compose exec web python manage.py dumpdata > dumpPostrgeSQL.json
```
докер хаб образ

```
https://hub.docker.com/repository/docker/nemets87/api_yamdb/general
```

проект доступен по адресу 

```
http://localhost/api/v1/

```
# Регистрация нового пользователя
```
POST http://localhost/api/v1/auth/signup/

{
  "email": "string",
  "username": "string"
}
```

**Редактирование, удаление и создание категорий жанров и произведений доступно только Администратору**
# Примеры запросов
## Получение списка всех категорий

```
GET http://localhost/api/v1/categories/
```
## Добавление новой категории

```
POST http://localhost/api/v1/categories/

{
  "name": "string",
  "slug": "string"
}
```


### **Авторы:**
- [Sergey Fedorov + Power 45 Team ](https://github.com/Nemets87)

### **Создан и проверен на: **
- [Linux Mint 21.1 "Vera" ](https://linuxmint.com/download.php)
