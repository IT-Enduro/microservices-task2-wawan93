# Домашняя работа #2

![GitHub Classroom Workflow](../../workflows/GitHub%20Classroom%20Workflow/badge.svg?branch=master)

## Authorization

### Формулировка

На базе Домашней работы #1 реализовать OAuth2 token-based авторизацию.

1. Для авторизации использовать OpenID Connect, в роли Identity Provider использовать [Auth0](https://auth0.com).
2. На Ticket Service реализовать метод `GET /api/v1/authorize` получения токена (`username` и `password` пользователя
   передаем в Header `Authorization` как Basic Authorization.
3. При запросе на получение токена Ticket Service берет `username` и `password` из запроса, `Client Id`
   и `Client Secret` из настроек приложения и обращается к Identity Provider для получения токена.
4. Все остальные методы `/api/**` на всех сервисах закрыть token-based авторизацией.
5. Убрать заголовок `X-User-Name` и получать пользователя из JWT-токена.
6. Если авторизация некорректная (отсутствует токен, ошибка валидации JWT токена, закончилось время жизни токена (поле
   exp в payload)), то отдавать 401 ошибку.
7. Проверку токена выполнять в Ticket Service через JWKs.

### Настройка Auth0

1. Регистрируемся на [Auth0](https://auth0.com).
2. Создаем приложение: `Applications` -> `Create Application`: `Native`, заходим в созданное приложение и
   копируем `Client ID` и `Client Secret`.
3. Переходим в `Advanced Settings` -> `Grant Types`: только `Password` (Resource Owner Password Flow).
4. Переходим в `API` -> `Create API`:
    * Name: `Cinema Aggregator Service`;
    * Identifier: `http://ciname-aggregator-<your-name>.ru`;
    * Signing Algorithm: HS256.
5. Настраиваем хранилище паролей: `Settings` -> `Tenant Settings` -> `API Authorization Settings`:
    * Default Audience: `http://ciname-aggregator-<your-name>.ru`;
    * Default Directory: `Username-Password-Authentication`.
6. Создаем тестового пользователя: `User Management` -> `Users` -> `Create User`:
    * Email: `user@mail.ru`;
    * Password: `Qwerty123`;
    * Connection: `Username-Password-Authentication`.

После настройки у вас должен успешно выполняться запрос на проверку получение токена (подставить свои настройки):

```shell
curl --location --request POST 'https://<YOUR ACCOUNT NAME>.eu.auth0.com/oauth/token' \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'grant_type=password' \
  --data-urlencode 'username=user@mail.ru' \
  --data-urlencode 'password=Qwerty123' \
  --data-urlencode 'scope=openid' \
  --data-urlencode 'client_id=<YOU CLIENT ID>' \
  --data-urlencode 'client_secret=<YOUR CLIENT SECRET>'
```

В ответ получаем токен:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InZsSWctTG4yR0dOeEE3dGNjSi1aRyJ9.eyJpc3MiOiJodHRwczovL21pY3Jvc2VydmljZXMteWVyZXZhbi5ldS5hdXRoMC5jb20vIiwic3ViIjoiYXV0aDB8NjJmZGM2ZDNhNDEzNjBiYWJkYTQ4NjA5IiwiYXVkIjpbImh0dHBzOi8vbWljcm9zZXJ2aWNlcy15ZXJldmFuLmV1LmF1dGgwLmNvbS9hcGkvdjIvIiwiaHR0cHM6Ly9taWNyb3NlcnZpY2VzLXllcmV2YW4uZXUuYXV0aDAuY29tL3VzZXJpbmZvIl0sImlhdCI6MTY2MDc5OTQ3OCwiZXhwIjoxNjYwODg1ODc4LCJhenAiOiJadHJHZWg3YmVZRE1QZTJpWElsamlZam5wc3RORTg0YiIsInNjb3BlIjoib3BlbmlkIHByb2ZpbGUgZW1haWwgYWRkcmVzcyBwaG9uZSByZWFkOmN1cnJlbnRfdXNlciB1cGRhdGU6Y3VycmVudF91c2VyX21ldGFkYXRhIGRlbGV0ZTpjdXJyZW50X3VzZXJfbWV0YWRhdGEgY3JlYXRlOmN1cnJlbnRfdXNlcl9tZXRhZGF0YSBjcmVhdGU6Y3VycmVudF91c2VyX2RldmljZV9jcmVkZW50aWFscyBkZWxldGU6Y3VycmVudF91c2VyX2RldmljZV9jcmVkZW50aWFscyB1cGRhdGU6Y3VycmVudF91c2VyX2lkZW50aXRpZXMiLCJndHkiOiJwYXNzd29yZCJ9.uXwescu7TwwXnDVVW0-OlPRHaVF0Ahk0ilm3GfYfEEMCuTxz_kPURopbfQtViEeQNB1_UxiCgw7pco_8HmMkaK4f7rbGSBTAmpA6mA5kz-lIb1tpBpNSZlUYjSMZQfvMYK5p7N3rv52OA335v29t3ExOqMDmRHbvFUqTBE6T3uxcxBDqbS67uHhXolmevm95hbZoRTxdFcs7ROkZI0rmEx7bK-ZeIiJFcXBmXtcIu817akUNMPavM4Qt4B7xFnryKiTka0T9K7Hp9NVxS14GsuU0zsWJDsjqq_HzxYA7jViI2WYXuwC_R8w6InYNoiXmZN-UJZK_-MUzLgli_Sawdg",
  "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InZsSWctTG4yR0dOeEE3dGNjSi1aRyJ9.eyJuaWNrbmFtZSI6InVzZXIiLCJuYW1lIjoidXNlckBtYWlsLnJ1IiwicGljdHVyZSI6Imh0dHBzOi8vcy5ncmF2YXRhci5jb20vYXZhdGFyLzQ3OTAyMzA3Y2Y4ZWQ2Y2ZiMWViZDZiZmVhMzFkY2QwP3M9NDgwJnI9cGcmZD1odHRwcyUzQSUyRiUyRmNkbi5hdXRoMC5jb20lMkZhdmF0YXJzJTJGdXMucG5nIiwidXBkYXRlZF9hdCI6IjIwMjItMDgtMThUMDU6MTE6MTguMzkyWiIsImVtYWlsIjoidXNlckBtYWlsLnJ1IiwiZW1haWxfdmVyaWZpZWQiOmZhbHNlLCJpc3MiOiJodHRwczovL21pY3Jvc2VydmljZXMteWVyZXZhbi5ldS5hdXRoMC5jb20vIiwic3ViIjoiYXV0aDB8NjJmZGM2ZDNhNDEzNjBiYWJkYTQ4NjA5IiwiYXVkIjoiWnRyR2VoN2JlWURNUGUyaVhJbGppWWpucHN0TkU4NGIiLCJpYXQiOjE2NjA3OTk0NzgsImV4cCI6MTY2MDgzNTQ3OH0.p1hdBNUuXjEwPErV-lgiRit_Ud2kVGIuz-AcEZjzgRZKbyKYh1zckK1Z0ZrLOhpSH63UCBX6KLtrx9imJP9QkamWlkK3yxOI4qX7zRstTAEY_doSXukxm-U4jaLwl_Yc2B39FZMCMz0A22TGjVL9L1upzyNdngzhAihlcliteE1XNu9gWMeQV5-GdOWqI1XL7QLJZbMXeSlr7uF5hTNqy2Gpilr4BAJM0Hn_XZefD1ZefK1yXRWDjpyO6Sm74d6UEI1aFu_eKWDAWVdsnthirkhP61Lustytlt8rrxMszyvDXSglIkn1GoA9mKRI5MUVUEloAM4KogUlu_YcClEO-Q",
  "scope": "openid profile email address phone read:current_user update:current_user_metadata delete:current_user_metadata create:current_user_metadata create:current_user_device_credentials delete:current_user_device_credentials update:current_user_identities",
  "expires_in": 86400,
  "token_type": "Bearer"
}
```

### Пояснения

1. Для получения метаданных для OpenID Connect можно
   использовать [Well-Known URI](https://auth0.com/docs/security/tokens/json-web-tokens/locate-json-web-key-sets):
   https://<your-account-name>.eu.auth0.com/.well-known/openid-configuration.
2. Из Well-Known метаданных можно получить Issuer URI и JWKs URI.
3. Для реализации OAuth2 можно использовать сторонние библиотеки.

### Прием задания

1. При получении задания у вас создается _копия_ этого репозитория для вашего пользователя.
2. После того как все тесты успешно завершатся, в Github Classroom на Dashboard будет отмечено успешное выполнение
   тестов.

### Ссылки

1. [Resource Owner Password Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/resource-owner-password-flow)
2. [Locate JSON Web Key Sets](https://auth0.com/docs/security/tokens/json-web-tokens/locate-json-web-key-sets)
3. [Introduction to JSON Web Tokens](https://jwt.io/introduction)
