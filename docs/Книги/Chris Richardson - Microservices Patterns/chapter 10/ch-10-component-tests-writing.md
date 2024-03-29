---
share: true
tags:
 - testing
 - microservice/testing
---
# Разработка компонентных тестов
[[service-component-test-pattern|Шаблон "компонентный тест сервиса"]]. 
Компонентные тесты близки к приемочным тестам (acceptance tests), поэтому сперва определим приемочные тесты.
## Определение приёмочных тестов
Приемочные тесты относятся к бизнес-аспектам программного компонента. Они описывают предпочтительное поведение, которое наблюдают клиенты компонента, игнорируя внутреннюю реализацию. Формируются на основе сценариев, например, история `Place Order`
```
Как у потребителя сервиса Order, у меня должна быть возможность разместить заказ.
```
разворачивается в сценарий
```
Дано: действительный потребитель
Дано: действительная банковская карта
Дано: ресторан принимает заказы
Когда я заказываю виндалу из курицы в Ajanta
Тогда заказ должен быть принят
И должно быть опубликовано событие Order Authorized
```
Этот сценарий описывает желаемое поведение сервиса Order с точки зрения его API.
Каждый сценарий определяет приемочный тест. Разделы `Дано` соответствуют подготовительному этапу теста, раздел `Когда` соотносится с этапом выполнения, а `Тогда` и `И` — с проверкой.
Таким образом, тест будет делать следующее:
1. Создает заказ, обращаясь к ручке `POST /orders`
2. Проверяет состояние заказа, обращаясь к ручке `GET /orders/{orderId}`
3. Подписывается на подходящий канал сообщений, чтобы проверить, опубликовал ли сервис `Order` событие `OrderAuthorized`.

Далее в книге предлагаются примеры на языке Gherkin и среде выполнения Cucumber. Для .NET есть проект [Specflow](https://specflow.org/).
## Проектирование компонентных тестов
Представим, что мы пишем компонентный тест для сервиса Order. Мы определяем сценарий на Gherkin и выполняем его с помощью фреймворка. Однако перед выполнением сценария компонентыый тест должен запустить сервис Order и подготовить его зависимости, сконфигурировав заглушки.
Существует несколько решений, которые позволяют выбирать между реализмом и скоростью/простотой.
### Компонентные тесты внутри процесса
Одно из решений состоит в написании *внутрипроцессных компонентных тестов*, которые подменяют зависимости сервиса заглушками и макетами. Слабая сторона здесь в том, что мы не тестируем развёртываемый сервис.
### Компонентное тестирование за пределами процесса
*Внепроцессный компонентный тест* задействует настоящую инфраструктуру, включая БД и брокер сообщений, подменяя заглушками те зависимости, которые являются сервисами приложения. Ключевое преимущество — более широкое покрытие тестов, посокольку тестируемые компоненты значительно меньше отличаются от кода, который будет развертываться. Недостаток в том, что они сложнее в написании, выполняются медленнее.
