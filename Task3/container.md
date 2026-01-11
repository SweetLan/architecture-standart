# Диаграмма контейнеров банк "Стандарт"

```plantuml
@startuml container
title "банк "Стандарт" — Диаграмма контейнеров TO-BE"

top to bottom direction

!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Container.puml

Person(customerNew, "Новый клиент", "Подаёт заявку на депозит на сайте")
Person(customerExisting, "Действующий клиент", "Подаёт заявку на депозит в интернет-банке")
Person(operator, "Оператор кол-центра", "Обрабатывает заявки")
Person(branchStaff, "Сотрудник отделения", "Идентификация клиента")
Person(backOffice, "Менеджер бэк-офиса депозитов", "Обработка заявки в АБС (MVP)")

System_Ext(telco, "Телеком-оператор", "Доставка СМС")

Container_Boundary(ch, "Каналы обслуживания") {
   Container(site, "Сайт", "PHP + React", "Подача заявки, отображение депозитов/ставок (публичный канал)")
  Container(ibWeb, "Интернет-банк Web UI", "ASP.NET MVC", "UI интернет-банка")
  Container(ibBackend, "Интернет-банк Backend", ".NET Framework (монолит) + MS SQL", "Бэкенд интернет-банка (существующий)")
  ContainerDb(ibDb, "IB Database", "MS SQL", "БД интернет-банка (существующая)")
}

Container_Boundary(ns, "Новые сервисы (MVP депозитов)") {
  Container(apiGw, "API Gateway ", "Nginx", "Единая точка входа")
  Container(das, "Deposit Application Service", ".NET + MS SQL", "Заявки/статусы, интеграции, SMS")
  ContainerDb(dasDb, "Deposit Applications DB", "MS SQL", "Хранение заявок и статусов (MVP)")
  Container(drs, "Deposit Rates Service", ".NET", "Каталог депозитов и ставки (общие/персональные), кэширование")
  ContainerDb(drsDb, "Rates DB/Cache", "MS SQL / Cache", "Ставки/справочники")
  Container(notify, "Notification", ".NET", "Шаблоны уведомлений, формирование сообщений")
}

Container_Boundary(ccb, "Кол-центр (существующий)") {
  Container(ccUi, "CC Web UI", "React", "Интерфейс оператора")
  Container(ccBe, "CC Backend", "Java Spring Boot (microservices)", "API и бизнес-логика кол-центра")
  ContainerDb(ccDb, "CC DB", "PostgreSQL", "Данные кол-центра/лиды/обращения")
}

Container_Boundary(absb, "АБС (существующая)") {
  Container(absClient, "ABS Desktop Client", "Delphi", "Клиент для сотрудников")
  Container(absLogic, "ABS Business Logic", "PL/SQL", "Основная бизнес-логика в БД")
  ContainerDb(absDb, "ABS Database", "Oracle", "Учёт операций, договоры, продукты")
  Container(absRates, "ABS Rates Module", "Oracle (tables + PL/SQL)", "Ведение ставок вместо XLS (доступ депозиты/кредиты по ролям)")
}

Container_Boundary(integ, "Интеграция") {
  Container(absAdapter, "ABS Integration Adapter", "Java", "ACL, лимиты")
}

Container_Boundary(smsb, "SMS") {
  Container(sms, "СМС-шлюз банка", "Integration service", "Отправка СМС/OTP через телеком")
}

' Интернет-банк
Rel(ibWeb, ibBackend, "HTTP-запросы UI", "HTTPS")
Rel(ibBackend, ibDb, "read/write", "SQL")

' UC-01: заявка с сайта + список депозитов/ставок
Rel(customerNew, site, "Подаёт заявку / смотрит депозиты", "HTTPS/TLS")
Rel(site, apiGw, "Запрос на создание заявки", "HTTPS/TLS")
Rel(apiGw, das, "Передача запрос на создание заявки", "HTTP(S)")
Rel(das, dasDb, "Сохранить заявку/статус", "SQL")

Rel(site, apiGw, "Получение списка депозитов и ставок", "HTTPS/TLS")
Rel(apiGw, drs, "Маршрутизация / передача запроса", "HTTP(S)")
Rel(drs, drsDb, "Чтение данных (кэш / БД)", "SQL")

' UC-02: обработка кол-центром, особые условия, фиксация статуса/заметок
Rel(das, ccBe, "Создание и обновление лида по заявке", "REST")
Rel(operator, ccUi, "Работа с лидами/заявками", "HTTPS")
Rel(ccUi, ccBe, "HTTP-запросы UI", "HTTPS")
Rel(ccBe, ccDb, "read/write", "SQL")

Rel(ccBe, apiGw, "Обновление статуса и комментариев заявки", "HTTPS/TLS")
Rel(apiGw, das, "Маршрутизация запроса на обновление заявки", "HTTP(S)")
Rel(das, dasDb, "Обновление статуса и комментариев заявки", "SQL")

' UC-03: идентификация в отделении (в АБС)
Rel(branchStaff, absClient, "Идентификация клиента", "Desktop")
Rel(absClient, absLogic, "Выполнение бизнес-операций", "Local/DB")
Rel(absLogic, absDb, "read/write", "SQL")

' UC-04: просмотр депозитов/ставок в интернет-банке (через DRS)
Rel(customerExisting, ibWeb, "Просмотр депозитов и ставок", "HTTPS/TLS")
Rel(ibBackend, apiGw, "Запрос списка депозитов и ставок", "HTTPS/TLS")
Rel(apiGw, drs, "Маршрутизация запроса на получение ставок", "HTTP(S)")
Rel(drs, drsDb, "Чтение данных (кэш / БД)", "SQL")

' Наполнение витрины ставок из АБС (чтобы не обращаться к АБС на каждый запрос клиента)
Rel(drs, absAdapter, "Синхронизация ставок и продуктового каталога (по расписанию)", "REST")
Rel(absAdapter, absRates, "Получение данных по ставкам и продуктам", "Adapter/API")
Rel(absRates, absDb, "read/write", "SQL")

' UC-05: подача заявки в интернет-банке и подтверждение по СМС
Rel(ibBackend, apiGw, "Инициация подтверждения по СМС", "REST HTTPS/TLS")
Rel(apiGw, das, "Маршрутизация запроса на отправку СМС", "HTTP(S)")
Rel(das, notify, "Формирование текста СМС", "Internal call")
Rel(das, sms, "Отправка СМС с кодом подтверждения", "Internal API")
Rel(sms, telco, "Доставка СМС", "Внешний шлюз"))

Rel(ibBackend, apiGw, "Подтверждение СМС-кода", "HTTPS/TLS")
Rel(apiGw, das, "Маршрутизация запроса на проверку СМС-кода", "HTTP(S)")

Rel(ibBackend, apiGw, "Создание подтверждённой заявки на депозит", "HTTPS/TLS")
Rel(apiGw, das, "Маршрутизация запроса на создание заявки", "HTTP(S)")
Rel(das, dasDb, "Сохранение заявки и статуса", "SQL")

' UC-06: обработка заявки бэк-офисом в АБС + СМС о подтверждении ставки
Rel(backOffice, absClient, "Подтверждение условий и ставки по депозиту", "Desktop")
Rel(das, absAdapter, "Запрос статуса подтверждения ставки", "Internal API")
Rel(absAdapter, absDb, "Контролируемый доступ к данным АБС", "Adapter/API")
Rel(das, notify, "Формирование СМС о подтверждении ставки", "Internal call")
Rel(das, sms, "Отправка СМС о подтверждении ставки", "Internal API")

' UC-07: открытие депозита в АБС + СМС об открытии
Rel(backOffice, absClient, "Открытие депозита (MVP через бэк-офис)", "Desktop")
Rel(das, absAdapter, "Запрос статуса открытия депозита", "Internal API")
Rel(absAdapter, absDb, "Контролируемый доступ к данным АБС", "Adapter/API")
Rel(das, notify, "Формирование СМС об открытии депозита", "Internal call")
Rel(das, sms, "Отправка СМС об открытии депозита", "Internal API")

' Архитектурное ограничение: не ходить из ИБ напрямую в АБС
Rel_D(ibBackend, absDb, "Запрещено: прямой доступ к АБС", "Причина: нагрузка на Oracle/риски доступности")



@enduml

```