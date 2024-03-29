---
share: true
tags:
 - NET/ASPNETCore
 - NET/DI
---
# Регистрация собственных сервисов в контейнере
Как уже [[ch-10-creating-loosely-coupling-code|упоминалось]], чтобы контроллер `UserControlller` смог использовать сервис — реализацию интерфейса, переданного ему в конструкторе, этот сервис надо зарегистрировать в [[di-container|контейнере зависимостей]]. Также для этого необходимо зарегистрировать все зависимости этой реализации, и т.д.
Процесс регистрации сервисов (конфигурирование контейнера) состоит из ряда утверждений о сервисах, например:
- если сервису требуется `IEmailSender`, используй экземпляр `EmailSender`;
- если сервису требуется `NetworkClient`, используй экземпляр `NetworkClient`;
- если сервису требуется `MessageFactory`, используй экземпляр `MessageFactory`.

Эти утверждения выполняются путём вызова методов `Add*` у `IServiceCollection` в методе `ConfigureServices()`. Каждый метод указывает на три составляющие регистрации:
- *тип сервиса* `TService` — класс или интерфейс, который будет запрошен в качестве зависимости;
- *тип реализации* `TService` или `TImplementation` — класс, объект которого контейнер должен предоставить для разрешения зависимости. Это должен быть конкретный тип, который может совпадать с типом сервиса;
- *жизненный цикл* — **transient**, **singleton** или **scoped**. Определяет, как долго должен использоваться экземпляр сервиса.

Пример:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddControllers();
	services.AddScoped<IEmailSender, EmailSender>();
	services.AddScoped<NetworkClient>();
	services.AddSingleton<MessageFactory>();
}
```
Как видно, здесь не указано, как конкретно создавать реализации. Контейнер в своей работе делает ряд предположений о типах-реализациях, и, чтобы данный способ регистрации работал, типы должны удовлетворять этим предположениям, а именно:
- это должен быть класс конкретного типа;
- у класса должен быть только один “допустимый” конструктор;
- чтобы конструктор был “допустимым”, все аргументы конструктора должны быть зарегистрированы в контейнере, или быть аргументами со значениями по умолчанию.
