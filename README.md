---
# api_yamdb with docker
## Описание
Проект **YaMDb** собирает отзывы пользователей на различные произведения. Проект можно запустить из контейнера благодоря Docker compose.
## Технологии
![Django-app workflow](https://github.com/Grizzzley/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)
Django
Postgresql
Nginx
Gunicorn
Docker Compose

Авторы проекта YaMDb:
- [Виктория Мальцева (тимлид, разработка Categories, Genres, Titles)](https://github.com/Amica24)
- [Антон Милов (разработка Review, Comments)](https://github.com/TonyKauk)
- [Михаил Васильев (разработка Auth и Users)](https://github.com/Grizzzley)
---


### Запуск из контейнера на ОС Linux
Все операции производятся с помощью терминала Linux
Клонируем репозиторий:
```
git clone git@github.com:Grizzzley/infra_sp2.git
```
Для корректной работы необходимо создать файл ```.env``` (env - переменные окружения) в папке infra_sp2/infra.
```
cd infra_sp2/infra
touch .env
nano .env
```
Пример содержимого файла ```.env```:
```python
DB_ENGINE=django.db.backends.postgresql
DB_NAME=postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
DB_HOST=db
DB_PORT=5432
SECRET_KEY=<your_key>
```
Что бы сохранить файл в редакторе nano Ctrl+O и вернуться в терминал Ctrl+X
Далее запускаем docker-compose:
```
sudo docker-compose up -d
``` 
-d запуск в фоновом режиме
Будут созданы необходимые для работы контейнеры (database, web, nginx)
Далее нужно применить миграции, создать суперпользователя и собрать статику:
```
sudo docker-compose exec web python manage.py makemigrations
sudo docker-compose exec web python manage.py migrate
sudo docker-compose exec web python manage.py createsuperuser
sudo docker-compose exec web python manage.py collectstatic --no-input
```
Проект будет доступен по адресу ```http://localhost/```

---
### Работа с API
### Алгоритм регистрации пользователей

    1. Пользователь отправляет POST-запрос на добавление нового пользователя с параметрами `email` и `username` на эндпоинт `/api/v1/auth/signup/`.
    2. **YaMDB** отправляет письмо с кодом подтверждения (`confirmation_code`) на адрес  `email`.
    3. Пользователь отправляет POST-запрос с параметрами `username` и `confirmation_code` на эндпоинт `/api/v1/auth/token/`, в ответе на запрос ему приходит `token` (JWT-токен).
    4. При желании пользователь отправляет PATCH-запрос на эндпоинт `/api/v1/users/me/` и заполняет поля в своём профайле (описание полей — в документации).


## Paths

Запросы к API начинаются с `/api/v1/`

Документация так-же доступна по адрессу:
http://127.0.0.1:8000/redoc/


**/auth/signup/**
* Получить код подтверждения на переданный `email`.
* Права доступа: Доступно без токена.
    
    *Пример POST-запроса:* 

   >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"email": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"username": "string"  
    }


**/auth/token/**
* Получение JWT-токена в обмен на username и confirmation code.
* Права доступа: Доступно без токена
    
    *Пример POST-запроса:*  

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"username": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"confirmation_code": "string"  
    }

    *Пример ответа:*  
    
    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"token": "string",  
    }

**/categories/**
  
GET-запрос:
* Получить список всех категорий
* Права доступа: Доступно без токена.

    *Пример ответа:*

    >[ 
    &nbsp;&nbsp;&nbsp;&nbsp;{  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"count": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"next": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"previous": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"results": []  
    &nbsp;&nbsp;&nbsp;&nbsp;}  
    ]
    
POST-запрос:
* Создать категорию
* Права доступа: Администратор.

    *Пример запроса:*  
    
    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"slug": "string"  
    }

    *Пример ответа:*  
    
    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"slug": "string"  
    }

**/categories/{slug}/**

DELETE-запрос:
* Удалить категорию
* Права доступа: Администратор.


**/genres/**

GET-запрос:
* Получить список всех жанров
* Права доступа: Доступно без токена.   

    >[  
    &nbsp;&nbsp;&nbsp;&nbsp;{  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"count": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"next": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"previous": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"results": []  
    &nbsp;&nbsp;&nbsp;&nbsp;}  
    ]

POST-запрос:
* Добавить жанр
* Права доступа: Администратор.

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"slug": "string"  
    }  
    
    *Пример ответа:* 
    
    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"slug": "string"  
    }


**/genres/{slug}/**

DELETE-запрос:
* Удалить жанр
* Права доступа: Администратор.


**/titles/**


GET-запрос:
* Получить список всех объектов
* Права доступа: Доступно без токена. 

    >[  
    &nbsp;&nbsp;&nbsp;&nbsp;{  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"count": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"next": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"previous": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"results": []  
    &nbsp;&nbsp;&nbsp;&nbsp;}  
    ]

POST-запрос:
* Добавить новое произведение
* Права доступа: Администратор.

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"year": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"description": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"genre": [  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"string"  
    &nbsp;&nbsp;&nbsp;&nbsp;],  
    &nbsp;&nbsp;&nbsp;&nbsp;"category": "string"  
    }

    *Пример ответа:*  

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"id": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"year": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"rating": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"description": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"genre": [  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{}  
    &nbsp;&nbsp;&nbsp;&nbsp;],  
    &nbsp;&nbsp;&nbsp;&nbsp;"category": {  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"slug": "string"  
    &nbsp;&nbsp;&nbsp;&nbsp;}  
    }


**/titles/{titles_id}/**

GET-запрос:
* Информация о произведении
* Права доступа: Доступно без токена. 

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"id": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"year": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"rating": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"description": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"genre": [  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{}  
    &nbsp;&nbsp;&nbsp;&nbsp;],  
    &nbsp;&nbsp;&nbsp;&nbsp;"category": {  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"slug": "string"  
    &nbsp;&nbsp;&nbsp;&nbsp;}  
    }

PATCH-запрос:
* Обновить информацию о произведении
* Права доступа: Администратор.

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"year": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"description": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"genre": [  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"string"  
    &nbsp;&nbsp;&nbsp;&nbsp;],  
    &nbsp;&nbsp;&nbsp;&nbsp;"category": "string"  
    }  
    
    *Пример ответа:* 
    
    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"id": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"year": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"rating": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"description": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"genre": [  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{}  
    &nbsp;&nbsp;&nbsp;&nbsp;],  
    &nbsp;&nbsp;&nbsp;&nbsp;"category": {  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"slug": "string"  
    &nbsp;&nbsp;&nbsp;&nbsp;}  
    }

DELETE-запрос:
* Удалить произведение
* Права доступа: Администратор.


**/titles/{title_id}/reviews/**

GET-запрос:
* Получить список всех отзывов
* Права доступа: Доступно без токена. 

    >[  
    &nbsp;&nbsp;&nbsp;&nbsp;{  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"count": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"next": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"previous": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"results": []  
    &nbsp;&nbsp;&nbsp;&nbsp;}  
    ]

POST-запрос:
* Добавить новый отзыв
* Права доступа: Аутентифицированные пользователи.

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"text": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"score": 1  
    }  
    
    *Пример ответа:*  
    
    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"id": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"text": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"author": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"score": 1,  
    &nbsp;&nbsp;&nbsp;&nbsp;"pub_date": "2019-08-24T14:15:22Z"  
    }


**/titles/{title_id}/reviews/{review_id}/**

GET-запрос:
* Получить отзыв по id для указанного произведения
* Права доступа: Доступно без токена. 

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"id": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"text": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"author": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"score": 1,  
    &nbsp;&nbsp;&nbsp;&nbsp;"pub_date": "2019-08-24T14:15:22Z"  
    }

PATCH-запрос:
* Частично обновить отзыв по id
* Права доступа: Автор отзыва, модератор или администратор.

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"text": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"score": 1  
    }  
    
    *Пример ответа:* 
    
    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"id": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"text": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"author": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"score": 1,  
    &nbsp;&nbsp;&nbsp;&nbsp;"pub_date": "2019-08-24T14:15:22Z"  
    }

DELETE-запрос:
* Удалить отзыв по id
* Права доступа: Автор отзыва, модератор или администратор.


**/titles/{title_id}/reviews/{review_id}/comments/**

GET-запрос:
* Получить список всех комментариев к отзыву по id
* Права доступа: Доступно без токена. 

    >[  
    &nbsp;&nbsp;&nbsp;&nbsp;{  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"count": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"next": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"previous": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"results": []  
    &nbsp;&nbsp;&nbsp;&nbsp;}  
    ]

POST-запрос:
* Добавить новый комментарий для отзыва
* Права доступа: Аутентифицированные пользователи.

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"text": "string"  
    }  
    
    *Пример ответа:*  
    
    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"id": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"text": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"author": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"pub_date": "2019-08-24T14:15:22Z"  
    }


**/titles/{title_id}/reviews/{review_id}/comments/{comment_id}/**

GET-запрос:
* Получить комментарий для отзыва по id
* Права доступа: Доступно без токена. 

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"id": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"text": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"author": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"pub_date": "2019-08-24T14:15:22Z"  
    }


PATCH-запрос:
* Частично обновить комментарий к отзыву по id
* Права доступа: Автор отзыва, модератор или администратор.

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"text": "string"  
    }  
    
    *Пример ответа:*  
    
    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"id": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;"text": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"author": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"pub_date": "2019-08-24T14:15:22Z"  
    }

DELETE-запрос:
* Удалить комментарий к отзыву по id
* Права доступа: Автор отзыва, модератор или администратор.


**/users/**

GET-запрос:
* Получить список всех пользователей
* Права доступа: Администратор. 

    >[  
    &nbsp;&nbsp;&nbsp;&nbsp;{  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"count": 0,  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"next": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"previous": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"results": []  
    &nbsp;&nbsp;&nbsp;&nbsp;}  
    ]

POST-запрос:
* Добавить нового пользователя
* Права доступа: Администратор. 

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"username": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"email": "user@example.com",  
    &nbsp;&nbsp;&nbsp;&nbsp;"first_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"last_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"bio": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"role": "user"  
    }  

    *Пример ответа:*  
    
    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"username": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"email": "user@example.com",  
    &nbsp;&nbsp;&nbsp;&nbsp;"first_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"last_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"bio": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"role": "user"  
    }


**/users/{username}/**

GET-запрос:
* Получить пользователя по username
* Права доступа: Администратор. 

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"username": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"email": "user@example.com",  
    &nbsp;&nbsp;&nbsp;&nbsp;"first_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"last_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"bio": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"role": "user"  
    }

PATCH-запрос:
* Изменить данные пользователя по username
* Права доступа: Администратор. 

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"username": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"email": "user@example.com",  
    &nbsp;&nbsp;&nbsp;&nbsp;"first_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"last_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"bio": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"role": "user"  
    }  

    *Пример ответа:*  

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"username": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"email": "user@example.com",  
    &nbsp;&nbsp;&nbsp;&nbsp;"first_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"last_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"bio": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"role": "user"  
    }

DELETE-запрос:
* Удалить пользователя по username
* Права доступа: Администратор. 


**/users/me/**

GET-запрос:
* Получить данные своей учетной записи
* Права доступа: Любой авторизованный пользователь. 


PATCH-запрос:
* Изменить данные своей учетной записи
* Права доступа: Любой авторизованный пользователь. 

    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"username": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"email": "user@example.com",  
    &nbsp;&nbsp;&nbsp;&nbsp;"first_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"last_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"bio": "string"  
    }  
    
    *Пример ответа:*  
    
    >{  
    &nbsp;&nbsp;&nbsp;&nbsp;"username": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"email": "user@example.com",  
    &nbsp;&nbsp;&nbsp;&nbsp;"first_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"last_name": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"bio": "string",  
    &nbsp;&nbsp;&nbsp;&nbsp;"role": "user"  
    }
    