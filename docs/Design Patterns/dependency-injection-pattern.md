---
aliases:
 - внедрение зависимостей
 - внедрения зависимостей
 - внедрению зависимостей
 - внедрением зависимостей
 - внедрении зависимостей
share: true
tags: [OOP/pattern]
---
# Шаблон “Внедрение зависимостей”
**Внедрение зависимостей (Dependency Injection, DI)** — шаблон, при котором зависимости (то есть классы, от которых зависит код класса) не создаются кодом класса, а передаются в него извне. Часть реализации [[single-responsibility-principle|принципа единой ответственности]].
Также так называют и архитектурный подход, когда приложение проектируется с преимущественным использованием этого шаблона. Обычно для эффективной реализации подхода применяют специальный фреймворк, называемый [[di-container|контейнером зависимостей]]. Задачей контейнера является контроль за жизненным циклом классов, использующих данный шаблон (создание, предоставление по требованию,  уничтожение). Классы, контролируемые контейнером, обычно называются *сервисами*.
## Основные способы внедрения зависимостей
1. *Через конструктор* — зависимости передаются как параметры конструктора.
2. *Через сеттеры свойств* — зависимости передаются через сеттеры соответствующих свойств объекта
3. *Через интерфейс* — интерфейс зависимости предоставляет метод инжектора, который внедрит зависимость в любой переданный ей клиент. Клиенты должны реализовать интерфейс, который предоставляет метод установки (сеттер), который принимает зависимость.

Основным ( и поддерживаемым всеми контейнерами) является первый способ. Остальные способы могут не поддерживаться тем или иным контейнером.
## Жизненный цикл сервиса
*Жизненный цикл* сервиса — это продолжительность существования экземпляра сервиса в контейнере, прежде чем будет создан новый экземпляр.
Основные типы жизненных циклов:
- Объект создается один раз на всё время жизни приложения. Способ реализации шаблона [[singleton-pattern|Одиночка]] [^1];
- Новый объект создается при каждом новом запросе [^2];
- Объект создается один раз в рамках существования специального объекта, называемого *скоуп (scope)* [^3]. Обычно на основе scope реализуются более сложные случаи — per request, per thread и т.п. Зависит от контейнера.

[^1]:Пример из книги Э.Лока [[ch-10-di-lifecycle#Singleton|ASP.NET Core in Action]]
[^2]:Пример из книги Э.Лока [[ch-10-di-lifecycle#Transient|ASP.NET Core in Action]]
[^3]:Пример из книги Э.Лока [[ch-10-di-lifecycle#Scoped|ASP.NET Core in Action]]

## Ссылки
https://en.wikipedia.org/wiki/Dependency_injection
https://ru.wikipedia.org/wiki/%D0%92%D0%BD%D0%B5%D0%B4%D1%80%D0%B5%D0%BD%D0%B8%D0%B5_%D0%B7%D0%B0%D0%B2%D0%B8%D1%81%D0%B8%D0%BC%D0%BE%D1%81%D1%82%D0%B8

