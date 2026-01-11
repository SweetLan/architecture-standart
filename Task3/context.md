# Контекстная диаграмма банк "Стандарт"

```plantuml
@startuml context-diagram
!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Context.puml

title "Банк Стандарт — Контекстная диаграмма TO-BE"
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

System_Ext(telco, "Телеком-оператор", "Внешний провайдер доставки СМС")

Rel(customerNew, site, "Подаёт заявку / смотрит ставки", "HTTPS/TLS")
Rel(customerExisting, ib, "Подаёт заявку, подтверждает СМС", "HTTPS/TLS")

Rel(site, das, "Создать заявку", "REST (HTTPS/TLS)")
Rel(ib, das, "Создать заявку + OTP verify", "REST (HTTPS/TLS)")
Rel(ib, drs, "Получить продукты/ставки", "REST (HTTPS/TLS)")

Rel(das, cc, "Создать/обновить лид/заявку", "REST/Integration")
Rel(operator, cc, "Работает с заявками", "Web UI")

Rel(das, absAdapter, "Запрос статусов/операций", "Internal API")
Rel(absAdapter, abs, "Ограниченная интеграция с АБС", "Adapter/API")

Rel(das, sms, "Отправка СМС/OTP", "Internal API")
Rel(sms, telco, "Доставка СМС", "External gateway")

Rel(branchStaff, abs, "Идентификация/оформление", "Desktop client")
Rel(backOffice, abs, "Подтверждение условий/ставки (MVP)", "Desktop client")


@enduml
```