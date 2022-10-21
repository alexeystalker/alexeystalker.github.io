---
share: true
tags:
 - NET/ASPNETCore/Identity
---
# Создание проекта из шаблона
> [!tip] Совет
> Можно воспользоваться командной строкой:
>  ```bash
>  dotnet new webapp -au Individual -uld
>  ```

Чтобы создать проект в VS 2019:
1. **File > New > Project** или **Create new project** на заставке;
2. Выбираем **ASP.NET Core Web Application**
3. Вводим имя проекта, местоположение и имя решения, нажимаем **Create**;
4. В разделе **Authentication** нажимаем **Change**;
5. Выбираем **Individual User Accounts** и жмём **OK**;
6. Жмем "Создать", чтобы создать приложение. Запустится команда `dotnet restore`.
7. Запускаем приложение, чтобы убедиться, что всё хорошо.

Получаем экран по умолчанию с кнопками **Register** и **Login**:
![[Pasted image 20220605195209.png]]