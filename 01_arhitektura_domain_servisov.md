# Архитектура сервисов предметной области (Domain Services)

Источник: https://chatgpt.com/s/t_6a4e7c48cf488191b2875f4e1414b19b (изначально «ТЗ-023», получен в серии документов GreenMarket 2026-07-08, вынесен как платформенная методология — см. `README.md`)

> Текст оставлен как получен, в исходных примерах упоминается GreenMarket — принципы ниже одинаково применимы к любому продукту на Platform Core (GreenMarket, Taxi, Postamat, Voice Assistant).

> Настоящий документ определяет архитектурные требования к сервисам предметной области Platform Core и не зависит от конкретного механизма реализации бизнес-процессов.

## Назначение

Настоящий документ определяет требования к сервисам предметной области продукта на Platform Core.

Сервисы предметной области являются единственной точкой реализации бизнес-логики. Они отделяют Client UI, Platform Core и REST API от конкретного механизма исполнения бизнес-процессов.

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
Business Process Engine
```

Business Process Engine — абстракция реализации бизнес-процесса. В зависимости от продукта она может быть реализована различными способами: Repository + CRUD; встроенный FSM Engine; внешний FSM Engine; Workflow/BPM Engine; иной специализированный механизм исполнения. Platform Core и Domain Service не зависят от способа реализации Business Process Engine.

## Требование 1. Один сервис — одна предметная область

Не допускается создание универсальных сервисов. Например: `PurchaseService`, `CatalogService`, `SellerService`, `InventoryService`, `BasketService`.

## Требование 2. Сервис принимает Action

Правильно: `PurchaseService.handle(AddProductAction)` или `PurchaseService.execute(action)`.

Неправильно: `repository.update(...)` из UI или Controller.

## Требование 3. Repository не содержит бизнес-логики

Repository отвечает только за: чтение; сохранение; поиск. Repository никогда не принимает бизнес-решений.

## Требование 4. Domain Service принимает решения

Например: можно ли заменить товар; можно ли завершить покупку; можно ли подтвердить наличие. Ответ всегда вычисляет Domain Service либо делегирует вычисление Business Process Engine.

## Требование 5. После выполнения Service возвращает не Entity

Возвращается результат операции: `ActionResult`; `BusinessEvents`; `ViewModel`. А не просто изменённая запись БД.

## Требование 6. Запрет Repository → Repository

Один Repository не вызывает другой Repository. Взаимодействие происходит только через Domain Service.

## Требование 7. Service ничего не знает о UI

Domain Service не знает: React; экранов; панелей; голосового интерфейса; карт; других каналов взаимодействия. Service знает только предметную область.

## Требование 8. Service ничего не знает о реализации бизнес-процесса

Domain Service не должен содержать код вида:

```
if (crudMode)
if (fsmEnabled)
if (remoteFsm)
```

или любую другую логику выбора механизма исполнения. Реализация бизнес-процесса выбирается инфраструктурой. Domain Service работает исключительно через контракт Business Process Engine.

Для Domain Service не имеет значения: используется CRUD; используется встроенный FSM; используется внешний FSM; используется Workflow Engine; используется другой механизм исполнения. Все перечисленные варианты являются различными реализациями одного архитектурного контракта.

## Требование 9. Service публикует Business Events

После успешного выполнения Action Service публикует Business Events. Например: `PurchaseIntentChanged`, `PurchaseOptionCalculated`, `SellerSelected` (пример из GreenMarket — в других продуктах названия событий свои).

## Требование 10. Замена реализации Business Process Engine

Допускается изменение только внутренней реализации Business Process Engine. Контракт Domain Service при этом должен оставаться неизменным.

Например, сегодня:

```
Domain Service
    ↓
Repository
```

После внедрения встроенного FSM:

```
Domain Service
    ↓
FSM Engine
    ↓
Repository
```

После вынесения FSM в отдельный сервис:

```
Domain Service
    ↓
Business API
    ↓
FSM Engine
```

Во всех случаях интерфейс Domain Service остаётся неизменным.

## Архитектурный принцип

Platform Core зависит только от контракта Domain Service.

Domain Service зависит только от контракта Business Process Engine.

Business Process Engine может быть реализован: локально; как отдельный сервис; любым другим способом.

Физическое расположение реализации бизнес-процесса не является частью архитектурного контракта и не должно влиять на: Client UI; Platform Core; Action Dispatcher; REST API.

Смена механизма исполнения бизнес-процесса (CRUD, локальный FSM, удалённый FSM, Workflow Engine или иной механизм) не требует изменения публичных интерфейсов системы.
