# CHECKLIST-4.4.1: Smoke-тестирование создания заявки

**Модуль:** Создание заявки (POST /application)  
**Приоритет:** High  
**Тип тестирования:** Функциональное, smoke-тестирование  

## Предусловия
- Микросервис Application запущен и доступен по адресу http://localhost:8080
- БД доступна
- Микросервис Deal доступен

## Шаги воспроизведения

| № | Шаг | Данные | Ожидаемый результат |
|---|-----|--------|---------------------|
| 1 | Выполнить вызов метода POST /application с валидными данными | **URL:** http://localhost:8080/application**Body:**```{  "amount": 100000,  "term": 24,  "firstName": "Ivan",  "lastName": "Ivanov",  "middleName": "Ivanovich",  "email": "smoke01@test.com",  "birthdate": "1990-01-01",  "passportSeries": "1234",  "passportNumber": "123456"}``` | Метод вызван, возвращен HTTP 200 OK и массив из 4 кредитных предложений |
| 1.1 | Убедиться, что данные клиента сохранены в БД | **SQL:**```SELECT first_name, last_name, email FROM client WHERE email = 'smoke01@test.com'``` | Запись найдена, данные совпадают с переданными в запросе |
| 1.2 | Убедиться, что параметры кредита сохранены в таблице credit | **SQL:**```SELECT amount, term FROM credit WHERE application_id = (  SELECT id FROM application   WHERE client_id = (    SELECT id FROM client WHERE email = 'smoke01@test.com'  ))``` | Найдена 1 запись, где amount = 100000, term = 24 |
| 2 | Выполнить вызов метода POST /application с невалидной суммой (amount < 10000) | **URL:** http://localhost:8080/application**Body:**```{  "amount": 9999,  "term": 24,  "firstName": "Ivan",  "lastName": "Ivanov",  "middleName": "Ivanovich",  "email": "smoke02@test.com",  "birthdate": "1990-01-01",  "passportSeries": "1234",  "passportNumber": "123456"}``` | Возвращен HTTP 500 с кодом ошибки ERR_1, запись в БД не создана |

## Фактический результат
Все проверки пройдены успешно

## Примечание
Этот чек-лист покрывает критический путь (Critical Path) создания заявки. Он гарантирует, что:
- Валидные запросы корректно сохраняются в БД (маппинг T_02).
- Невалидные запросы блокируются на уровне валидации, не создавая "мусорных" записей в БД.
