# CHECKLIST-4.4.3: Smoke-тестирование скоринга заявки

**Модуль:** Скоринг заявки (PUT /application/registration/{applicationId})  
**Приоритет:** High  
**Тип тестирования:** Функциональное, smoke-тестирование  

## Предусловия
- Микросервис Application запущен и доступен по адресу http://localhost:8080
- БД доступна, микросервисы Deal, Conveyor и Dossier доступны, Kafka запущена
- Успешно выполнены CHECKLIST-4.4.1 и CHECKLIST-4.4.2, статус заявки = 'APPROVED'

## Шаги воспроизведения

| № | Шаг | Данные | Ожидаемый результат |
|---|-----|--------|---------------------|
| 1 | Выполнить вызов метода PUT /application/registration/{applicationId} с валидными данными | **URL:** http://localhost:8080/application/registration/<applicationId>**Body:**```{  "gender": "MALE",  "maritalStatus": "SINGLE",  "dependentAmount": 0,  "passportIssueDate": "2010-01-01",  "passportIssueBranch": "Department 1",  "employment": {    "employmentStatus": "EMPLOYED",    "employerINN": "123456789012",    "salary": 50000,    "position": "WORKER",    "workExperienceTotal": 36,    "workExperienceCurrent": 12  },  "account": "11223344556677889900"}``` | Метод вызван, возвращен HTTP 200 OK, тело ответа пустое |
| 1.1 | Убедиться, что статус заявки изменился на CC_APPROVED | **SQL:**```SELECT status FROM application WHERE id = <applicationId>``` | status = 'CC_APPROVED' |
| 1.2 | Убедиться, что данные клиента обновлены в БД | **SQL:**```SELECT passport_info->>'issue_branch' as branch, account FROM client WHERE id = (SELECT client_id FROM application WHERE id = <applicationId>)``` | branch = 'Department 1', account = '11223344556677889900' |
| 1.3 | Убедиться, что кредит рассчитан и сохранен | **SQL:**```SELECT credit_status, rate, psk FROM credit WHERE id = (SELECT credit_id FROM application WHERE id = <applicationId>)``` | credit_status = 'CALCULATED', rate = 15.0, psk > 0 |
| 2 | Выполнить вызов скоринга с отсутствующим обязательным полем (account) | **URL:** http://localhost:8080/application/registration/<applicationId>**Body:**```{  "gender": "MALE",  "maritalStatus": "SINGLE",  "dependentAmount": 0,  "passportIssueDate": "2010-01-01",  "passportIssueBranch": "Department 1",  "employment": {    "employmentStatus": "EMPLOYED",    "employerINN": "123456789012",    "salary": 50000,    "position": "WORKER",    "workExperienceTotal": 36,    "workExperienceCurrent": 12  }}``` | Возвращен HTTP 500 с кодом ошибки ERR_1, статус заявки не изменился |

## Фактический результат
Все проверки пройдены успешно

## Примечание
Этот чек-лист валидирует финальный этап скоринга. Он гарантирует, что:
- Обогащенные данные клиента корректно сохраняются (маппинг T_02).
- Финальный расчет ставки (rate) и полной стоимости кредита (psk) выполняется успешно (маппинг T_06).
- Отсутствие критических полей (например, номера счета) блокирует процесс и не портит данные в БД.