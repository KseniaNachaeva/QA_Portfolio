# TC-4.4.3-POS-15: Проверка комбинации корректировок (несколько факторов одновременно)

**Модуль:** Скоринг заявки (PUT /application/registration/{applicationId})  
**Приоритет:** High  
**Тип тестирования:** Функциональное, сложная бизнес-логика  

## Предусловия
- Микросервис Application запущен и доступен по адресу http://localhost:8080
- БД доступна
- Микросервисы Deal, Conveyor и Dossier доступны
- Kafka доступна
- Выполнен вызов метода POST /application (4.4.1) с валидными данными
- Получен HTTP 200 OK, сохранены applicationId и список из 4 кредитных предложений
- Выбрано предложение №4 (isInsuranceEnabled=true, isSalaryClient=true) и выполнен вызов POST /application/apply
- Получен HTTP 200 OK, статус заявки изменен на APPROVED

## Шаги воспроизведения

| № | Шаг | Данные | Ожидаемый результат |
|---|-----|--------|---------------------|
| 1 | Выполнить вызов PUT /application/registration/{applicationId} | **Body:**```{  "gender": "FEMALE",  "maritalStatus": "MARRIED",  "dependentAmount": 2,  "passportIssueDate": "2010-01-01",  "passportIssueBranch": "Department 15",  "employment": {    "employmentStatus": "EMPLOYED",    "employerINN": "123456789012",    "salary": 150000,    "position": "TOP_MANAGER",    "workExperienceTotal": 120,    "workExperienceCurrent": 48  },  "account": "11223344556677889914"}```**Параметры клиента:**- Женщина 44 года- В браке (MARRIED)- 2 иждивенца- Топ-менеджер (TOP_MANAGER) | Метод вызван |
| 2.1 | Проверить логи МС credit-conveyor | — | Запись о применении всех корректировок:• Базовая ставка TT = 10%• Женщина 35–60 лет (44 года): −3% → 7%• MARRIED: −3% → 4%• Иждивенцы > 1 (2 шт): +1% → 5%• TOP_MANAGER: −4% → 1% |
| 3 | Выполнить запрос к БД | **SQL:**```SELECT rate FROM credit WHERE id = <credit_id>``` | **rate = 1.0**(10% − 3% − 3% + 1% − 4% = 1%) |
| 3.1 | Выполнить запрос к БД | **SQL:**```SELECT status FROM application WHERE id = <applicationId>``` | status = 'CC_APPROVED' |
| 3.2 | Выполнить запрос к БД | **SQL:**```SELECT credit_status FROM credit WHERE id = <credit_id>``` | credit_status = 'CALCULATED' |
| 4 | Убедиться, что вернулся ответ | — | HTTP статус 200, тело ответа пустое |

## Фактический результат
Тест пройден успешно

## Примечание
Этот тест проверяет **комбинацию нескольких корректировок** процентной ставки одновременно:

| Фактор | Корректировка | Промежуточный результат |
|--------|---------------|-------------------------|
| Базовая ставка (TT) | — | 10% |
| Женщина 35–60 лет | −3% | 7% |
| MARRIED | −3% | 4% |
| Иждивенцы > 1 | +1% | 5% |
| TOP_MANAGER | −4% | **1%** |

Тест подтверждает корректную работу **сложной бизнес-логики** с множественными факторами.