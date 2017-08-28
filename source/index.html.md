---
title: Verbatoria API

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Взаимодействие с Backend осуществляется по HTTPS протоколу, используя REST API,
формат данных - JSON.

## Выполнение запросов, требующих авторизации

При выполнении запросов API, которые требуют авторизации, необходимо передавать
в HTTP заголовке с именем *access_token* токен возвращенный сервером после
авторизации.


# Авторизация Нейрометриста

> Пример кода

```shell
curl "http://verbatoria.ru/api/v1/session.json" --request POST --data-binary
'{"phone":"+79031234567", "password":"Str0nGpa$s"}'
```

> Ответ

```json
{
  "access_token": "7827nd88ju98dj29084d28j9048",
  "expires_at": "2017-10-17T23:10:01.770+04:00"
}
```
> TODO: добавить возврат состояния сессии - активен ли партнер и достаточно
> средств на балансе

### HTTP Request

`POST http://verbatoria.ru/api/v1/session.json`

### Параметры

Parameter | Required | Description
--------- | ------- | -----------
phone | true | Телефон пользователя
password | true | Пароль пользователя

<aside>
При успешной авторизации код ответа 201, при неуспешной 422
</aside>

# Получение информации об авторизованном Нейрометристе

<aside>
Данный метод требует валидной сессии. Необходимо передать  *access_token* в
заголовке HTTP запроса.
</aside>

> Пример кода

```shell
curl "http://verbatoria.ru/api/v1/verbatolog/current.json" 
--header "access_token: 7827nd88ju98dj29084d28j9048"
```

> Ответ

```json
{
  "first_name": "Иван",
  "last_name": "Иванов",
  "middle_name": "Иванович",
  "phone": "+79031234567",
  "email": "ivan@ivanov.ru"
}
```

### HTTP Request

`GET http://verbatoria.ru/api/v1/verbatolog/current.json`

# Получение списка запланированных сеансов для текущего нейрометриста

<aside>
Данный метод требует валидной сессии. Необходимо передать  *access_token* в
заголовке HTTP запроса.
</aside>

> Пример кода

```shell
curl "http://verbatoria.ru/api/v1/verbatolog/current/events.json"
--header "access_token: 7827nd88ju98dj29084d28j9048"
```

> Ответ

```json
[
  {
    "id": 1,
    "start_at": "2017-05-01T12:00:00.000+04:00",
    "end_at": "2017-05-01T13:00:00.000+04:00",
    "child": {
      "id": 1,
      "name": "Василий Пупкин",
      "birth_day": "2007-01-01"
    }
  }
]
```

### HTTP Request

`GET http://verbatoria.ru/api/v1/verbatolog/current/events.json`

# Старт сессии нейрометрии

<aside>
Данный метод требует валидной сессии. Необходимо передать  *access_token* в
заголовке HTTP запроса.
</aside>

> Пример кода

```shell
curl "http://verbatoria.ru/api/v1/reports.json" --request POST
--data-binary '{"event_id":10}'
--header "access_token: 7827nd88ju98dj29084d28j9048"
```

> Ответ

При успешной обработке заброса возвращается HTTP статус 201. В случае, если
переданный event_id принадлежит другому нейрометристу, будет возвращен HTTP
статус 404.

При успешной обработке, возвращается объект report, id которого будет нужен для
записи результатов измерений.

```json
{
  "id": 1,
  "child_id": 10,
  "verbatolog_id": 11,
  "location_id": 2,
  "created_at": "2017-05-01T12:00:00.000+04:00",
  "updated_at": "2017-05-01T12:00:00.000+04:00"
}
```

### HTTP Request

`POST http://verbatoria.ru/api/v1/reports.json`

# Добавление результатов измерения в открытую сессию нейрометрии

<aside>
Данный метод требует валидной сессии. Необходимо передать  *access_token* в
заголовке HTTP запроса.
</aside>

> Пример кода

```shell
curl "http://verbatoria.ru/api/v1/reports/5/measurement.json" --request POST
--data-binary '{"measurement":{"action_id":0,"attention":-1,"bci_id":null,"block":null,
"created_at":null,"delta":16777095,"device_id":null,"event_id":null,"high_alpha":16777154,
"high_beta":16777158,"logoped_mode_id":null,"low_alpha":4395,"low_beta":16777200,
"low_gamma":16777158,"mediation":0,"mid_gamma":6694,"mistake":null,"report_id":0,
"reserve_blank1":null,"reserve_blank2":null,"theta":16777140,"updated_at":null,"word":null}}'
--header "access_token: 7827nd88ju98dj29084d28j9048"
```

> Ответ

При успешной обработке заброса возвращается HTTP статус 201. В случае, если
переданный event_id принадлежит другому нейрометристу, будет возвращен HTTP
статус 404.


### HTTP Request

`POST http://verbatoria.ru/api/v1/reports/:report_id:/measurement.json`

# Закрытие сессии нейрометрии

После добавления всех результатов измерения, сессия нейрометрии должна быть
закрыто.
*Важно* - закрыть можно сессию только в том случае, если добавлены не пустые
данные о ребенке (имя и дата рождения)

> Пример кода

```shell
curl "http://verbatoria.ru/api/v1/reports/5/finalize.json" --request POST
--header "access_token: 7827nd88ju98dj29084d28j9048"
```
> Ответ

При успешной обработке заброса возвращается HTTP статус 201. В случае, если
данные по ребенку не были заполнены, вернется статус 400.

# Работа с клиентами

## Добавление

<aside>
Данный метод требует валидной сессии. Необходимо передать  *access_token* в
заголовке HTTP запроса.
</aside>

> Пример кода

```shell
curl "http://verbatoria.ru/api/v1/clients.json" --request POST
--data-binary '{"name":"Ivan","email":"email@example.com","phone":"+79031112233"}'
--header "access_token: 7827nd88ju98dj29084d28j9048"
```
> Ответ

При успешной обработке заброса возвращается HTTP статус 201. В случае, если
не удалось добавить клиента HTTP статус 400

### HTTP Request

`POST http://verbatoria.ru/api/v1/clients.json`

### Параметры

Parameter | Required | Description
--------- | ------- | -----------
name | true | Имя клиента
email | true | Email клиента
phone | false | Телефон клиента

## Изменение

<aside>
Данный метод требует валидной сессии. Необходимо передать  *access_token* в
заголовке HTTP запроса.
</aside>

> Пример кода

```shell
curl "http://verbatoria.ru/api/v1/clients/10.json" --request PUT
--data-binary '{"client": {"name":"Ivan","email":"email@example.com","phone":"+79031112233"}}'
--header "access_token: 7827nd88ju98dj29084d28j9048"
```
> Ответ

При успешной обработке заброса возвращается HTTP статус 200. В случае, если
не удалось изменить клиента HTTP статус 400

### HTTP Request

`PUT http://verbatoria.ru/api/v1/clients/:client_id:.json`

### Параметры

Parameter | Required | Description
--------- | ------- | -----------
name | true | Имя клиента
email | true | Email клиента
phone | false | Телефон клиента

## Просмотр

<aside>
Данный метод требует валидной сессии. Необходимо передать  *access_token* в
заголовке HTTP запроса.
</aside>

> Пример кода

```shell
curl "http://verbatoria.ru/api/v1/clients/10.json"
--header "access_token: 7827nd88ju98dj29084d28j9048"
```
# Работа с данными о детях

## Добавление

<aside>
Данный метод требует валидной сессии. Необходимо передать  *access_token* в
заголовке HTTP запроса.
</aside>

> Пример кода

```shell
curl "http://verbatoria.ru/api/v1/clients/10/children.json" --request POST
--data-binary '{"child":{"name":"Ivan","birth_day":"2010-01-01"}}'
--header "access_token: 7827nd88ju98dj29084d28j9048"
```
> Ответ

При успешной обработке заброса возвращается HTTP статус 201. В случае, если
не удалось добавить данные HTTP статус 400

### HTTP Request

`POST http://verbatoria.ru/api/v1/clients/:client_id:/children.json`

### Параметры

Parameter | Required | Description
--------- | ------- | -----------
name | true | Имя ребенка
birth_day | true | Дата рождения ребенка

## Изменение

<aside>
Данный метод требует валидной сессии. Необходимо передать  *access_token* в
заголовке HTTP запроса.
</aside>

> Пример кода

```shell
curl "http://verbatoria.ru/api/v1/clients/10/children/50.json" --request PUT
--data-binary '{"child":{"name":"Ivan","birth_day":"2010-01-01"}}'
--header "access_token: 7827nd88ju98dj29084d28j9048"
```
> Ответ

При успешной обработке заброса возвращается HTTP статус 200. В случае, если
не удалось изменить данные HTTP статус 400

### HTTP Request

`PUT http://verbatoria.ru/api/v1/clients/:client_id:/:child_id:.json`

### Параметры

Parameter | Required | Description
--------- | ------- | -----------
name | true | Имя ребенка
birth_day | true | Дата рождения ребенка

## Просмотр

<aside>
Данный метод требует валидной сессии. Необходимо передать  *access_token* в
заголовке HTTP запроса.
</aside>

> Пример кода

```shell
curl "http://verbatoria.ru/api/v1/clients/10/children/50.json"
--header "access_token: 7827nd88ju98dj29084d28j9048"
```
