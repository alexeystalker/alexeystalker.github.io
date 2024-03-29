---
tags: [microservice/pattern,architecture]
share: true
---
# Шаблон “Микросервисная архитектура”
![[Pasted image 20210829152253.png]]
Микросервисная архитектура — это архитектурный стиль, при котором [[ch-2-4-plus-1-view-model-of-architecture|представление реализации]] структурируется в виде набора компонентов — исполняемых или WAR файлов. Компоненты — это [[service-microservice|сервисы]], коннекторы — коммуникационные протоколы. Каждый сервис имеет свою архитектуру [[ch-2-4-plus-1-view-model-of-architecture|логического представления]] (например, [[ch-2-hexagonal-architecture|шестигранную архитектуру]]).
На рисунке — пример микросервисной архитектуры, где сервисы соответствуют бизнес-функциям, коннекторы — REST API или асинхронный обмен сообщениями.

Важной характеристикой микросервисной архитектуры является [[coupling|слабое зацепление]]

## достоинства
+ **Continuos delivery, Continuos deploy** — возможность непрерывных доставки и развертывания
	+ *Обеспечивает уровень тестируемости* — маленькие сервисы проще покрывать автоматическими тестами
	+ *Обеспечивает уровень развертываемости* — каждый сервис можно развернуть независимо от других
	+ *Позволяет сделать команды автономными*
+ **Небольшая кодовая база** каждого сервиса
+ **Независимое масштабирование** по осям X и Z [[scale-cube|куба масштабирования]], кроме того — каждый компонент можно развернуть на оптимальном для компонента оборудовании
+ **Лучшая изоляция неполадок**
+ **Возможность экспериментировать и внедрять новые технологии**

## недостатки
- **Сложно подобрать подходящий набор сервисов** — нет алгоритма разбиения системы на сервисы — каждый раз нужно всё продумывать с нуля. Также, если разделить систему неправильно — можно получить *распределенный монолит* — систему, компонеты которой необходимо деплоить вместе. Такой системе присущи недостатки как [[monolithic-architecture-pattern#Недостатки|монолита]], так и микросервисов
- **Распределенные системы гораздо сложнее** — сложнее реализация взаимодействия, сложнее реализация согласованности данных, сложнее написать тесты, покрывающие несколько сервисов, сложнее администрирование
- **Развертывание фич, охватывающих несколько сервисов, требует тщательной координации нескольких команд**
- **Нетривиальность решения о переходе** — непонятно, в какой момент надо переходить от монолита к микросервисам

## Ссылки
https://microservices.io/patterns/microservices.html
