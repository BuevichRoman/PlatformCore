# Архитектура сервисов предметной области (Domain Services)

Источник: https://chatgpt.com/s/t_6a4e7c48cf488191b2875f4e1414b19b (изначально «ТЗ-023», получен в серии документов GreenMarket 2026-07-08, вынесен как платформенная методология — см. `README.md`)

> Текст оставлен как получен, в исходных примерах упоминается GreenMarket — принципы ниже одинаково применимы к любому продукту на Platform Core (GreenMarket, Taxi, Postamat, Voice Assistant).

**Вводная реплика чата:** после документа о подготовке к FSM Engine не стоит возвращаться к UI — время написать ТЗ для Backend, которое обеспечит эту совместимость.

## Назначение

Настоящий документ определяет требования к сервисам продукта на Platform Core.

Сервисы предметной области являются единственной точкой реализации бизнес-логики. Они отделяют клиентский UI, REST API и Platform Core от конкретного механизма выполнения бизнес-процессов.

## Общая архитектура

```
Client UI
  ↓
Platform Core
  ↓
Action Dispatcher
  ↓
Domain Service
  ↓
Repository
  ↓
Business Events
```

После внедрения FSM Engine изменится только внутренняя часть Domain Service.

## Требование 1. Один сервис — одна предметная область

Не допускается создание универсальных сервисов. Например: `PurchaseService`, `CatalogService`, `SellerService`, `InventoryService`, `BasketService`.

## Требование 2. Сервис принимает Action

Правильно: `PurchaseService.handle(AddProductAction)` или `PurchaseService.execute(action)`.

Неправильно: `repository.update(...)` из UI или Controller.

## Требование 3. Repository не содержит бизнес-логики

Repository отвечает только за: чтение; сохранение; поиск. Repository никогда не принимает решений.

## Требование 4. Domain Service принимает решения

Например: можно ли заменить товар? можно ли завершить покупку? можно ли подтвердить наличие? Ответ всегда вычисляет Domain Service.

## Требование 5. После выполнения Service возвращает не Entity

Возвращается результат операции: `ActionResult` + `BusinessEvents` + `ViewModel`. А не просто изменённая запись БД.

## Требование 6. Запрет Repository → Repository

Один Repository не вызывает другой Repository. Взаимодействие происходит только через Domain Service.

## Требование 7. Service ничего не знает о UI

Service не знает: React; конкретные экраны и панели интерфейса. Он знает только предметную область.

## Требование 8. Service ничего не знает о CRUD/FSM

Сервис не должен содержать кода вида `if (fsmEnabled)` или `if (crudMode)`. Реализация выбирается инфраструктурой.

## Требование 9. Service публикует Business Events

После успешного выполнения Action Service публикует события. Например: `PurchaseIntentChanged`, `PurchaseOptionCalculated`, `SellerSelected` (пример из GreenMarket — в других продуктах названия событий свои).

## Требование 10. Внедрение FSM Engine

После появления FSM Engine допускается изменение только внутреннего механизма работы Domain Service. Интерфейс Service должен остаться неизменным.

## Архитектурный принцип

Сейчас:

```
Client UI → Platform Core → Action Dispatcher → Domain Service → Repository
```

Через месяц станет:

```
Client UI → Platform Core → Action Dispatcher → Domain Service → FSM Engine → Repository
```

Ни клиентский UI, ни Platform Core, ни REST API при этом не изменяются.

---

**Комментарий из чата:** до сих пор система готовилась к появлению FSM Engine на уровне контрактов. Теперь это закреплено на уровне архитектуры сервисов. Разработчик Backend получает чёткое правило: вся бизнес-логика концентрируется в сервисах предметной области, а смена механизма её выполнения (CRUD сегодня, FSM Engine завтра) не должна затрагивать внешние интерфейсы системы. Это позволяет развивать продукт постепенно, без массового рефакторинга при подключении FSM.
