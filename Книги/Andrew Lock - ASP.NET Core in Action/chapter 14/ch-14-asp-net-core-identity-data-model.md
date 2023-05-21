---
share: true
tags:
 - NET/ASPNETCore/Identity
---
# Модель данных ASP.NET Core Identity
По умолчанию Identity использует EF Core для хранения данных пользователей. Он предоставляет класс `IdentityDbContext`, который может выступать базовым для `ApplicationDbContext` приложения. `IdentityUser` используется в нем в качестве сущности пользователя.
Также в шаблоне представлены начальные [[ch-12-managing-changes-with-migrations|миграции]] для БД. Обновим БД командой
```sh
dotnet ef database update
```
Получим таблицы:
![[Pasted image 20220606173407.png]]
- *\_EFMigrationsHistory* — стандартная таблица миграции EF Core;
- *AspNetUsers* — сюда сериализуется `IdentityUser`;
- *AspNetUserClaims* — утверждения пользователя. Связь с пользователем — "многие к одному";
- *AspNetUserLogins* и *AspNetUserTokens* — для выполнения входа в учётную запись с использованием сторонних сервисов;
- *AspNetUserRoles*, *AspNetRoles* и *AspNetRoleClaims* — для совместимости со старой моделью полномочий на основе ролей.

Поля таблицы AspNetUsers:
![[Pasted image 20220606174530.png]]
Все дополнительные свойства пользователя хранятся в виде утверждений в таблице AspNetUserClaims. Это позволяет добавлять произвольную дополнительную информацию без изменения схемы БД.

Важна разница между `IdentityUser`, хранимой в таблице AspNetUsers и объектом `ClaimsPrincipal`, фактически являющимся свойством `HttpContext.User`. `ClaimsPrincipal` — это `IdentityUser`, который сочетается с дополнительными утверждениями из AspNetUserClaims. Именно `ClaimsPrincipal` используется для аутентификации и сериализуется в cookie-файл.