# **Домашнее задание к занятию «Микросервисы: принципы»**-***Вуколов Евгений***
 
Вы работаете в крупной компании, которая строит систему на основе микросервисной архитектуры.
Вам как DevOps-специалисту необходимо выдвинуть предложение по организации инфраструктуры для разработки и эксплуатации.
 
## **Задача 1: API Gateway** 
 
Предложите решение для обеспечения реализации API Gateway. Составьте сравнительную таблицу возможностей различных программных решений. На основе таблицы сделайте выбор решения.
 
Решение должно соответствовать следующим требованиям:
- маршрутизация запросов к нужному сервису на основе конфигурации,
- возможность проверки аутентификационной информации в запросах,
- обеспечение терминации HTTPS.
 
Обоснуйте свой выбор.
 
## **Задача 2: Брокер сообщений**
 
Составьте таблицу возможностей различных брокеров сообщений. На основе таблицы сделайте обоснованный выбор решения.
 
Решение должно соответствовать следующим требованиям:
- поддержка кластеризации для обеспечения надёжности,
- хранение сообщений на диске в процессе доставки,
- высокая скорость работы,
- поддержка различных форматов сообщений,
- разделение прав доступа к различным потокам сообщений,
- простота эксплуатации.
 
Обоснуйте свой выбор.
 
## **Задача 3: API Gateway * (необязательная)**
 
### **Есть три сервиса:**
 
**minio**
- хранит загруженные файлы в бакете images,
- S3 протокол,
 
**uploader**
- принимает файл, если картинка сжимает и загружает его в minio,
- POST /v1/upload,
 
**security**
- регистрация пользователя POST /v1/user,
- получение информации о пользователе GET /v1/user,
- логин пользователя POST /v1/token,
- проверка токена GET /v1/token/validation.
 
### **Необходимо воспользоваться любым балансировщиком и сделать API Gateway:**
 
**POST /v1/register**
1. Анонимный доступ.
2. Запрос направляется в сервис security POST /v1/user.
 
**POST /v1/token**
1. Анонимный доступ.
2. Запрос направляется в сервис security POST /v1/token.
 
**GET /v1/user**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/.
2. Запрос направляется в сервис security GET /v1/user.
 
**POST /v1/upload**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/.
2. Запрос направляется в сервис uploader POST /v1/upload.
 
**GET /v1/user/{image}**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/.
2. Запрос направляется в сервис minio GET /images/{image}.
 
### Ожидаемый результат
 
- Результатом выполнения задачи должен быть docker compose файл, запустив который можно локально выполнить следующие команды с успешным результатом.
Предполагается, что для реализации API Gateway будет написан конфиг для NGinx или другого балансировщика нагрузки, который будет запущен как сервис через docker-compose и будет обеспечивать балансировку и проверку аутентификации входящих запросов.
Авторизация
curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/token

 
- Загрузка файла
 
curl -X POST -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I' -H 'Content-Type: octet/stream' --data-binary @yourfilename.jpg http://localhost/upload
 
- Получение файла
curl -X GET http://localhost/images/4e6df220-295e-4231-82bc-45e4b1484430.jpg
 
---
 
#### [Дополнительные материалы: как запускать, как тестировать, как проверить](https://github.com/netology-code/devkub-homeworks/tree/main/11-microservices-02-principles)
 
---
 
### Как оформить ДЗ?
 
Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории

# **Решение**

## **Задача 1**

- Предложение для реализации API Gateway, представлена сравнительная таблица возможностей различных программных решений:



 |    Решение       | Маршрутизация запросов           | Проверка аутентификации        | Терминация HTTPS            |
 |------------------|----------------------------------|--------------------------------|-----------------------------|
 |NGINX             |Да, через конфигурацию            | Поддерживается через модули    | Да, встроенная поддержка    |
 |Amazon API Gateway| Да, на основе конфигурации       | Поддерживается через AWS IAM   | Да, автоматическая поддержка|
 |Kong              |Да, через плагины и конфигурацию  | Поддерживается через плагины   | Да, с помощью плагинов      |
 |KrakenD           | Да, на основе конфигурации       | Поддерживается через плагины   | Да, с помощью плагинов      |


- Выбор решения:

На основе таблицы я выберу использование NGINX в качестве API Gateway. Вот почему:

1. Маршрутизация запросов: NGINX позволяет гибко настраивать маршрутизацию через конфигурационные файлы.

2. Проверка аутентификации: NGINX поддерживает проверку аутентификации через дополнительные модули, такие как модуль для JWT или Basic Auth.

3. Терминация HTTPS: NGINX имеет встроенную поддержку терминации HTTPS, что упрощает настройку безопасности.

- Обоснование выбора:

1. Гибкость конфигурации: NGINX позволяет легко настраивать маршруты и правила обработки запросов через конфигурационные файлы.

2. Широкая поддержка протоколов: NGINX поддерживает различные протоколы, включая HTTP/1.1, HTTP/2 и WebSocket.

3. Высокая производительность: NGINX известен своей высокой производительностью и способностью обрабатывать большое количество запросов.

4. Открытый исходный код: NGINX имеет открытый исходный код, что позволяет разработчикам вносить изменения и расширять его функциональность.

5. Широкое сообщество: NGINX имеет большое и активное сообщество пользователей и разработчиков, что облегчает поиск решений для возникающих проблем.


## **Задача 2**

|Брокер	    |Кластеризация	|Хранение на диске	|Скорость работы	 |Поддержка форматов	 |Разделение прав доступа	 |Простота эксплуатации       |
|-----------|-------------------|-----------------------|------------------------|-----------------------|-------------------------------|----------------------------|
|Kafka      |Да, кластеры с     |Да, с ограниченным     |Высокая (до 1 млн       |Поддерживает различные |Поддерживается через ACL       |Средняя, требует настройки  |
|           | репликацией       |временем хранения      |сообщений в секунду)    |форматы                |                               |                            |
|RabbitMQ   |Да, кластеры с     |Да, постоянные и       |Средняя (до 50 тысяч    |Поддерживает различные |Поддерживается через плагины   |Простая, гибкая конфигурация|
|           |репликацией        |временные сообщения    |сообщений в секунду)    |форматы                |                               |                            |
|AWS SQS    |Да, автоматическая |Да, с возможностью     |Средняя (до 300 операций|Поддерживает различные |Поддерживается через IAM       |Простая, интегрируется с AWS|
|           |кластеризация      |хранения в DLQ         |в секунду для FIFO)     |форматы                |                               |                            |


- Выбор решения:

 На основе таблицы я выберу использование Apache Kafka в качестве брокера сообщений:

1. Кластеризация: Kafka поддерживает кластеризацию с репликацией, что обеспечивает надёжность и отказоустойчивость.

2. Хранение на диске: Kafka хранит сообщения на диске, что позволяет обеспечить их доставку даже в случае сбоя.

3. Скорость работы: Kafka имеет высокую скорость работы, способную обрабатывать более 1 миллиона сообщений в секунду, что делает его подходящим для больших потоков данных.

4. Поддержка форматов: Kafka поддерживает различные форматы сообщений, что обеспечивает гибкость в использовании.

5. Разделение прав доступа: Kafka позволяет разделить права доступа к различным потокам сообщений через ACL (Access Control Lists).

6. Простота эксплуатации: Хотя Kafka требует некоторой настройки, его мощные возможности и масштабируемость делают его популярным выбором для крупных проектов.

- Обоснование выбора:

1. Высокая производительность: Kafka предназначен для обработки больших потоков данных, что делает его идеальным для приложений с высокой нагрузкой.

2. Отказоустойчивость: Механизмы репликации и кластеризации Kafka обеспечивают высокую надёжность и способность восстанавливаться после сбоев.

3. Гибкость: Поддержка различных форматов сообщений и возможность настройки прав доступа позволяют использовать Kafka в широком спектре приложений.

4. Масштабируемость: Kafka легко масштабируется, что позволяет ему эффективно работать в крупных распределённых системах.

Если простота эксплуатации является приоритетом, и не требуется обработка миллионов сообщений в секунду, 
RabbitMQ также может быть хорошим выбором благодаря своей гибкой маршрутизации и простоте конфигурации. 
Но для проектов с очень высокими требованиями к производительности и масштабируемости Kafka является более подходящим решением
























