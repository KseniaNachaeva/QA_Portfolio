# TC-SEC-01: Получение ошибки при SQL-инъекции в поле firstName

**Модуль:** Создание заявки (POST /application)  
**Приоритет:** High  
**Тип тестирования:** Безопасность (Security Testing)  

## Предусловия
- Микросервис Application запущен и доступен по адресу http://localhost:8080
- БД доступна
- Микросервис Deal доступен

## Шаги воспроизведения

| № | Шаг | Данные | Ожидаемый результат |
|---|-----|--------|---------------------|
| 1 | Выполнить вызов метода POST /application МС Application | **URL:** http://localhost:8080/application**Body:**```{  "amount": 50000,  "term": 12,  "firstName": "Ivan'; DROP TABLE client; --",  "lastName": "Ivanov",  "middleName": "Ivanovich",  "email": "sec01@test.com",  "birthdate": "1990-01-01",  "passportSeries": "1234",  "passportNumber": "123456"}```**SQL-инъекция:** `Ivan'; DROP TABLE client; --` | Метод вызван |
| 1.1 | Проверить наличие сообщения о вызове МС Deal в логах МС Application | — | **Сообщение о вызове МС Deal ОТСУТСТВУЕТ в логах** |
| 2а | Убедиться, что данные клиента не сохранены в БД | **SQL:**```SELECT * FROM client WHERE email = 'sec01@test.com'``` | **Запись НЕ найдена** |
| 2а.1 | Убедиться, что параметры кредита не сохранены в таблице credit | **SQL:**```SELECT * FROM credit WHERE application_id = (  SELECT id FROM application   WHERE client_id = (    SELECT id FROM client     WHERE email = 'sec01@test.com'  ))``` | **Запись НЕ найдена** |
| 3б | Убедиться, что вернулся ответ с ошибкой и кодом 500 | — | **HTTP статус 500**, тело ответа содержит сообщение об ошибке ERR_1 |
| 4 | Проверить наличие информации об ошибке в логах | — | В логах МС Application присутствует stack trace, указывающий на **недопустимые символы в поле firstName** |
| 5 | Проверить, что таблица client не была удалена | **SQL:**```SELECT COUNT(*) FROM client``` | Таблица client существует, количество записей не изменилось |

## Фактический результат
Тест пройден успешно

## Примечание
Этот тест проверяет **защиту от SQL-инъекций**.

Атакующий пытается внедрить SQL-код:
```sql
Ivan'; DROP TABLE client; --