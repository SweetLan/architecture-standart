# Диаграмма контейнеров банк "Стандарт" (MVP + ставки для кол-центров)

```plantuml
@startuml container
title "банк "Стандарт" — Диаграмма контейнеров TO-BE 2"

top to bottom direction

!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Container.puml

Person(operator, "Оператор кол-центра банка", "Консультации и работа со ставками")
Person(backOffice, "Бэк-офис (депозиты/кредиты)", "Ведение/обновление ставок в АБС")
System_Ext(partnerCC, "Партнёрский кол-центр", "Внешняя система. Получает ставки файлами")

Container_Boundary(ccb, "Кол-центр (существующий, банк)") {
  Container(ccUi, "CC Web UI", "React", "Интерфейс оператора")
  Container(ccBe, "CC Backend", "Java Spring Boot (microservices)", "API и бизнес-логика кол-центра")
  ContainerDb(ccDb, "CC DB", "PostgreSQL", "Лиды/обращения")
}

Container_Boundary(ns, "Сервисы депозитов (MVP)") {
  Container(drs, "Deposit Rates Service", ".NET", "Витрина ставок + API, кэширование")
  ContainerDb(drsDb, "Rates DB/Cache", "MS SQL / Cache", "Ставки/справочники (витрина)")
  Container(ratePublisher, "Rate File Publisher", ".NET", "Генерация файла ставок партнёру (CSV/XLSX) + checksum/подпись")
}

Container_Boundary(absb, "АБС (существующая)") {
  Container(absClient, "ABS Desktop Client", "Delphi", "Клиент сотрудников")
  Container(absLogic, "ABS Business Logic", "PL/SQL", "Бизнес-логика в БД")
  ContainerDb(absDb, "ABS Database", "Oracle", "Операции/продукты/счета")
  Container(absRates, "ABS Rates Module", "Oracle (tables + PL/SQL)", "Единый источник ставок вместо XLS")
}

Container_Boundary(integ, "Интеграция") {
  Container(absAdapter, "ABS Integration Adapter", "Java", "ACL, лимиты, контролируемый доступ к АБС")
}


' UC-01: Оператор кол-центра банка смотрит актуальные ставки
Rel(operator, ccUi, "Открывает экран ставок", "HTTPS")
Rel(ccUi, ccBe, "Вызовы пользовательского интерфейса", "HTTPS")
Rel(ccBe, drs, "Запрос актуальных ставок", "REST (TLS)")
Rel(drs, drsDb, "Чтение витрины ставок (кэш / БД)", "SQL")

' UC-02: DRS получает ставки из банковского источника (по расписанию)
Rel(drs, absAdapter, "Запуск синхронизации ставок (по расписанию)", "Internal API")
Rel(absAdapter, absRates, "Получение данных ставок", "Контролируемый доступ")
Rel(absRates, absDb, "Чтение/запись ставок", "SQL")
Rel(drs, drsDb, "Обновление витрины ставок", "SQL")

' UC-03: Публикация ставок для партнёрского кол-центра файлами
Rel(ratePublisher, drs, "Получение витрины ставок для файла", "Internal API")
Rel(ratePublisher, partnerCC, "Передача файла ставок (CSV/XLSX)", "Файл (email/manual)")


' UC-04: Партнёрский кол-центр консультирует по ставкам
Rel(partnerCC, partnerCC, "Использование полученного файла ставок", "Внешний процесс")

' UC-05: Бэк-офис ведёт/обновляет ставки в едином месте (АБС)
Rel(backOffice, absClient, "Обновление ставок", "Desktop")
Rel(absClient, absLogic, "Выполнение операций", "Local/DB")
Rel(absLogic, absDb, "Чтение/запись данных", "SQL")

@enduml

```