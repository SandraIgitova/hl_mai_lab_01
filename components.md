# Компонентная архитектура
<!-- Состав и взаимосвязи компонентов системы между собой и внешними системами с указанием протоколов, ключевые технологии, используемые для реализации компонентов.
Диаграмма контейнеров C4 и текстовое описание. 
-->
## Компонентная диаграмма

```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

AddElementTag("microService", $shape=EightSidedShape(), $bgColor="CornflowerBlue", $fontColor="white", $legendText="microservice")
AddElementTag("storage", $shape=RoundedBoxShape(), $bgColor="lightSkyBlue", $fontColor="white")

Person(creator, "Чат owner")
Person(member, "Участник чата")
Person(user, "Пользователь")

System_Ext(web_messanger, "Веб-мессенджер", "HTML, CSS, JavaScript, React", "Веб-интерфейс")

System_Boundary(conference_site, "Мессенджер") {
   Container(auth_service, "Сервис авторизаций", "C++", "Сервис авторизаций пользователей", $tags = "microService")   
   Container(message_service, "Сервис сообщений", "C++", "Сервис управления публичными чатами и личными сообщениями", $tags = "microservice")
   Container(user_service, "Сервис управления пользователями", "C++", "Сервис поиска пользователей", $tags = "microservice")
   ContainerDb(db, "База данных", "PostgreSQL", "Хранение данных о чатах, сообщениях, и пользователях", $tags = "storage")   
}

Rel(creator, web_messanger, "Добавление и удаление участников в чата, удаление чата")
Rel(member, web_messanger, "Просмотр, публикация, редактирование сообщений. Выход из чата")
Rel(user, web_messanger, "Создание чатов, отправка и редактирование личных сообщений")
Rel(web_messanger, auth_service, "Авторизация", "localhost/auth")
Rel(auth_service, db, "INSERT/SELECT/UPDATE", "SQL")

Rel(web_messanger, message_service, "Работа c публичными чатами и личными сообщениями", "localhost/message")
Rel(message_service, db, "INSERT/SELECT/UPDATE", "SQL")
Rel(web_messanger, user_service, "Поиск людей, регистрация новых пользователей", "localhost/user")
Rel(user_service, db, "INSERT/SELECT/UPDATE", "SQL")


@enduml
```
## Список компонентов  

### Сервис авторизаций
**API**:
- Авторизация
    - Запрос
      ```http
      POST /auth HTTP/1.1
      Host: localhost
      Content-Type: application/json

      {
        "login": "mySuperLogin",
        "password": "qwerty123"
      }
      ```
    - Ответ
      ```http
      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "token":"auth-token",
        "user_id":"id"
      }
      ```


### Сервис управления пользователями
**API**:
-	Создание пользователя
    - Запрос
      ```http
      POST /user HTTP/1.1
      Host: localhost
      Content-Type: application/json

      {
          "login": "mySuperLogin",
          "password": "qwerty123",
          "firstname": "Bob",
          "secondname": "Rudolf",
          "email": "bob@rudolf.go"
      } 
      ```
    - Ответ
      ```http
      HTTP/1.1 200 OK
      ```
-	Поиск пользователя по логину или по маске: имени, фамилии, почты
    - Запрос
      ```http
      GET /user/search?login=login&mask-first-name=firstname&mask-second-name=secondname&mask-email=email HTTP/1.1
      Host: localhost
      Authorization: <auth-scheme> <authorization-parameters>
      ```
    - Ответ
      ```http
      HTTP/1.1 200 OK
      Content-Type: application/json

      [
        {
          "login": "mySuperLogin",
          "firstname": "Bob",
          "secondname": "Rudolf",
          "email": "bob@rudolf.go"
        },
      ]
      ```
### Сервис сообщений
**API**:
- Создание чата
  - Запрос
    ```http
    POST /message/chat HTTP/1.1
    Host: localhost
    Authorization: <auth-scheme> <authorization-parameters>
    Content-Type: application/json

    {
      "chat_name": "chat_name",
      "users": ["user_id_1", "user_id_2"]
    }
    ```
  - Ответ
    ```http
    HTTP/1.1 200 OK
    Content-Type: application/json

    {
        "chat_id": "chat_id"
    }
    ```
- Удаление чата
  - Запрос
    ```http
    DELETE /message/chat HTTP/1.1
    Host: localhost
    Authorization: <auth-scheme> <authorization-parameters>
    Content-Type: application/json

    {
        "chat_id": "chat_id"
    }
    ```
  - Ответ
    ```http
    HTTP/1.1 200 OK
    ```
- Добавление пользователей в чат
  - Запрос
    ```http
    POST /message/chat/users&chat_id=chat_id HTTP/1.1
    Host: localhost
    Authorization: <auth-scheme> <authorization-parameters>
    Content-Type: application/json

    {
      "users": ["user_id_1", "user_id_2"]
    }
    ```
  - Ответ
    ```http
    HTTP/1.1 200 OK
    ```
- Удаление пользователя из чата
  - Запрос
    ```http
    DELETE /message/chat/users&chat_id=chat_id HTTP/1.1
    Host: localhost
    Authorization: <auth-scheme> <authorization-parameters>
    Content-Type: application/json

    {
      "users": ["user_id_1", "user_id_2"]
    }
    ```
  - Ответ
    ```http
    HTTP/1.1 200 OK
    ```
- Создание сообщений в чате
  - Запрос
    ```http
    POST /message HTTP/1.1
    Host: localhost
    Authorization: <auth-scheme> <authorization-parameters>
    Content-Type: application/json

    {
      "chat_id": "chat_id",
      "message": "message"
    }
    ```
  - Ответ
    ```http
    HTTP/1.1 200 OK
    ```
- Создать личное сообщение
  - Запрос
    ```http
    POST /message/personal HTTP/1.1
    Host: localhost
    Authorization: <auth-scheme> <authorization-parameters>
    Content-Type: application/json

    {
      "user_id": "user_id",
      "message": "message"
    }
    ```
  - Ответ
    ```http
    HTTP/1.1 200 OK
    ```
- Редактировать сообщение
  - Запрос
    ```http
    PUT /message?message_id=id HTTP/1.1
    Host: localhost
    Authorization: <auth-scheme> <authorization-parameters>
    Content-Type: application/json

    {
      "message": "message"
    }
    ```
  - Ответ
    ```http
    HTTP/1.1 200 OK
    ```
- Удалить сообщение
  - Запрос
    ```http
    DELETE /message HTTP/1.1
    Host: localhost
    Authorization: <auth-scheme> <authorization-parameters>
    Content-Type: application/json

    {
      "message_id": "id"
    }
    ```
  - Ответ
    ```http
    HTTP/1.1 200 OK
    ```
