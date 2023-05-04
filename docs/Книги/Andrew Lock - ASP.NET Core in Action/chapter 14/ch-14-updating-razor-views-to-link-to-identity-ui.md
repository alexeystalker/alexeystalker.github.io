---
share: true
tags:
 - NET/ASPNETCore/Identity
---
# Обновление представлений Razor для связи с пользовательским интерфейсом Identity
Добавим виджет Login в макет, а также убедимся, что страницы Identity используют тот же макет, что и остальная часть приложения.
Сначала создадим файл **Areas/Identity/Pages/\_ViewStart.cshtml** и добавим туда следующее:
```razor
@{ Layout = "/Pages/Shared/_Layout.cshtml"; }
```
Таким образом, для страниц Identiy используется тот же макет по умолчанию, что и в остальном приложении.
Далее добавим **\_LoginPartial.cshtml** в **Pages/Shared** и определим виджет `Login`.
```razor
@using Microsoft.AspNetCore.Identity
@using RecipeApplication.Data;
@inject SignInManager<ApplicationUser> SignInManager
@inject UserManager<ApplicationUser> UserManager

<ul class="navbar-nav">
@if (SignInManager.IsSignedIn(User))
{
    <li class="nav-item">
        <a  class="nav-link text-dark" asp-area="Identity" asp-page="/Account/Manage/Index" title="Manage">Hello @User.Identity.Name!</a>
    </li>
    <li class="nav-item">
        <form class="form-inline" asp-area="Identity" asp-page="/Account/Logout" asp-route-returnUrl="@Url.Page("/", new { area = "" })" method="post" >
            <button  type="submit" class="nav-link btn btn-link text-dark">Logout</button>
        </form>
    </li>
}
else
{
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="Identity" asp-page="/Account/Register">Register</a>
    </li>
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="Identity" asp-page="/Account/Login">Login</a>
    </li>
}
</ul>
```

Фактически, это файл **\_LoginPartial.cshtml** из [[ch-14-creating-a-project-from-template|шаблона по умолчанию]], но с использованием `ApplicationUser`.

Теперь можно разместить в основном шаблоне
```html
<partial name="_LoginPartial" />
```
