---
share: true
tags:
 - microservice/pattern
 - microservice/testing
---
# Шаблон "Компонентный тест сервиса"
![[Pasted image 20211024124855.png]]
При этом шаблоне тесты проверяют работу сервиса, подменяя заглушками все его зависимости.
Различают [[ch-10-component-tests-writing#Компонентные тесты внутри процесса|внутрипроцессные]] и [[ch-10-component-tests-writing#Компонентное тестирование за пределами процесса|внепроцессные]] компонентные тесты.
Для более простого описания сценариев тестом можно воспользоваться языком Gherkin и фреймворком Specflow для .NET.
## Ссылки
https://microservices.io/patterns/testing/service-component-test.html
 [Specflow](https://specflow.org/).