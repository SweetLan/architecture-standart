# Контекстная диаграмма банк "Стандарт" (MVP + ставки для кол-центров)

```plantuml
@startuml context-diagram
!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Context.puml

title "Банк Стандарт — Контекстная диаграмма TO-BE 2"
top to bottom direction

Person(customerNew, "Новый клиент", "Подаёт заявку на депозит через сайт")
Person(customerExisting, "Действующий клиент", "Открывает депозит через интернет-банк")
Person(operator, "Оператор кол-центра", "Обрабатывает заявки и звонит клиенту")
Person(branchStaff, "Сотрудник отделения", "Проводит идентификацию/оформление в АБС")
Person(backOffice, "Менеджер бэк-офиса депозитов", "Подтверждает условия/ставку в АБС (MVP)")

System(site, "Сайт банка", "PHP/React. Подача заявки на депозит (новый клиент)")
System(ib, "Интернет-банк", "ASP.NET MVC монолит. Заявка на депозит (действующий клиент)")
System(das, "Deposit Application Service", ".NET + MS SQL. Приём заявок, статусы, интеграции, OTP/SMS orchestration")
System(drs, "Deposit Rates Service", ".NET + Cache/DB. Каталог депозитов и ставки (общие/персональные)")
System(cc, "Система кол-центра", "React + Java Spring Boot микросервисы + PostgreSQL")
System(absAdapter, "ABS Integration Adapter", "Anti-corruption layer для интеграции с АБС и ограничения нагрузки")
System(abs, "АБС", "Delphi клиент + Oracle + PL/SQL. Учёт, договоры, открытие депозитов")
System(sms, "СМС-шлюз банка", "Интеграция с телеком-оператором для отправки СМС")
System(ratePublisher, "Rate File Publisher", "Формирование файла ставок партнёру")

System(absRates, "ABS Rates Module", "Oracle/PL/SQL (ставки вместо XLS)")
System_Ext(partnerCC, "Партнёрский кол-центр", "Внешняя система. Получает ставки файлами")
System_Ext(telco, "Телеком-оператор", "Внешний провайдер доставки СМС")

Rel(customerNew, site, "Подаёт заявку / смотрит ставки", "HTTPS/TLS")
Rel(customerExisting, ib, "Подаёт заявку, подтверждает СМС", "HTTPS/TLS")

Rel(site, das, "Создать заявку", "REST (HTTPS/TLS)")
Rel(ib, das, "Создать заявку + OTP verify", "REST (HTTPS/TLS)")
Rel(ib, drs, "Получить продукты/ставки", "REST (HTTPS/TLS)")

Rel(das, cc, "Создать/обновить лид/заявку", "REST/Integration")

Rel(das, absAdapter, "Запрос статусов/операций", "Internal API")

Rel(das, sms, "Отправка СМС/OTP", "Internal API")
Rel(sms, telco, "Доставка СМС", "External gateway")

Rel(branchStaff, abs, "Идентификация/оформление", "Desktop client")
Rel(backOffice, abs, "Подтверждение условий/ставки (MVP)", "Desktop client")

' --- UC-01 (новый): оператор кол-центра банка смотрит ставки
Rel(operator, cc, "Запрашивает ставки", "Web UI")
Rel(cc, drs, "Получить актуальные ставки", "REST (TLS)")

' --- UC-02/UC-05: источник ставок в АБС + синхронизация витрины
Rel(drs, absAdapter, "Синхронизация витрины ставок", "Internal API (schedule)")
Rel(absAdapter, absRates, "Чтение ставок", "Контролируемый доступ")
Rel(absRates, abs, "Хранение/ведение ставок", "Внутренний модуль")

' --- UC-03: файл ставок для партнёра
Rel(ratePublisher, drs, "Получить витрину ставок для экспорта", "Internal API")
Rel(ratePublisher, partnerCC, "Передать файл ставок (CSV/XLSX)", "Файл (email/manual)")


@enduml
```