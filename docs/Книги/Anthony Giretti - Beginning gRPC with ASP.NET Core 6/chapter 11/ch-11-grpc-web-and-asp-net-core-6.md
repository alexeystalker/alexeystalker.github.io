---
share: true
tags:
 - gRPC/gRPC-web
 - NET/ASPNETCore
---
# gRPC-web и ASP.NET Core 6
Реализация gRPC-web будет на удивление простой, если вы уже знакомы с gRPC для ASP.NET Core 6. Некоторые изменения будут необходимы в том, и только в том случае, если вашим клиентом будет браузер. Помните, браузеры *не поддерживают* [[grpc-client-streaming-call|клиентские]] и [[grpc-duplex-streaming-call|двунаправленные потоки]], поэтому вам необходимо переделать `proto`-файлы так, чтобы они использовали только [[grpc-server-streaming-call|серверные потоки]] и [[grpc-unary-call|унарные вызовы]].
Но в чём смысл использовать gRPC-web, если ваш клиент не является браузером? В экосистеме .NET cуществует некоторое количество клиентских приложений, которые не поддерживают [[http-2|HTTP/2]], и поэтому использовать HTTP/1 альтернативу gRPC (то есть gRPC-web) необходимо для следующих типов клиентов:
- .NET Core 2;
- .NET Framework;
- Blazor WebAssembly;
- Mono;
- Xamarin.IOS и Android;
- UWP;
- Unity.

Помните, что gRPC-web может быть естественной альтернативой gRPC в случае, когда хостинг (например, Microsoft Azure) не полностью поддерживает HTTP/2.
Любое приложение, работающее под .NET Core 3 и выше, может использовать как gRPC, так и gRPC-web.
> [!Important] Важно!
> Если ваш клиент не является HTML/JavaScript приложением, выполняющимся в браузере, либо десктопным приложением, основанном на ElectronJS, вы можете переконфигурировать приложение CountryService из главы 9 так, чтобы оно стало работать в режиме gRPC-web.

Сперва добавим к проекту `CountryService.gRPC` NuGet-пакет Grpc.AspNetCore.Web, например через Package Manager Console:
```powershell
Install-Package Grpc.AspNetCore.Web
```
Далее, добавим соответствующую мидлварю
```csharp
app.UseGrpcWeb(new GrpcWebOptions { DefaultEnabled = true });
```
Настройка `DefaultEnabled = true` включает gRPC-web по умолчанию для любого настроенного gRPC-сервиса. Если же нет желания включать его по умолчанию, можно указать сервисы, для которых следует включить gRPC-web,  явно:
```csharp
app.UseGrpcWeb();
app.MapGrpcService<CountryGrpcService>().EnableGrpcWeb();
```
Собственно, это всё. Мы настроили gRPC-web для небраузерных клиентов.