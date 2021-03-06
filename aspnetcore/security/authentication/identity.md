---
title: "Общие сведения об учетных данных ASP.NET Core"
author: rick-anderson
description: "Использовать удостоверение с приложением ASP.NET Core"
keywords: "ASP.NET Core, удостоверение, авторизации, безопасность"
ms.author: riande
manager: wpickett
ms.date: 12/15/2017
ms.topic: article
ms.assetid: cf119f21-1a2b-49a2-b052-547ccb66ee83
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/authentication/identity
ms.openlocfilehash: 7daf0267a6dc659afbd188ce87e35ca40816a31d
ms.sourcegitcommit: 198fb0488e961048bfa376cf58cb853ef1d1cb91
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/14/2017
---
# <a name="introduction-to-identity-on-aspnet-core"></a>Общие сведения об учетных данных ASP.NET Core

По [Pranav Rastogi](https://github.com/rustd), [Рик Андерсон](https://twitter.com/RickAndMSFT), [Tom Dykstra](https://github.com/tdykstra), Джон Гэллоуэй [Reitan Эрик](https://github.com/Erikre), и [Стив Смит](https://ardalis.com/)

Удостоверение ASP.NET Core является система членства, в котором можно добавить функциональные возможности входа в приложение. Пользователи могут создавать учетную запись и имя входа с именем пользователя и пароль или их можно использовать поставщик внешней учетной записи, например Facebook, Google, учетной записи Майкрософт, Twitter или другим пользователям.

Вы можете настроить ASP.NET Identity Core использование базы данных SQL Server для хранения имен пользователей, пароли и данные профиля. Кроме того можно использовать собственные постоянное хранилище, например, табличное хранилище Azure. Этот документ содержит инструкции по Visual Studio и с помощью CLI.

## <a name="overview-of-identity"></a>Общие сведения об идентификации

В этом разделе будет использование ASP.NET Core Identity Добавление функциональности для регистрации, вход и выход пользователя. Более подробные инструкции по созданию приложений с помощью ASP.NET Core Identity см. в разделе Дальнейшие действия в конце этой статьи.

1.  Создайте проект веб-приложения ASP.NET Core с отдельными учетными записями пользователей.

    # <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)
    В Visual Studio, выберите **файл** -> **New** -> **проекта**. Выберите **веб-приложение ASP.NET** из **новый проект** диалоговое окно. При выборе ASP.NET Core **Web Application(Model-View-Controller)** для ASP.NET Core 2.x с **индивидуальные учетные записи** в качестве метода проверки подлинности.

    Примечание: Необходимо выбрать **индивидуальные учетные записи**.
 
    ![Диалоговое окно создания нового проекта](identity/_static/01-mvc_2.png)
    
    # <a name="net-core-clitabnetcore-cli"></a>[Интерфейс командной строки .NET Core](#tab/netcore-cli)
    Если используется .NET Core CLI, создайте новый проект с помощью ``dotnet new mvc --auth Individual``. Эта команда создает новый проект с тем же кодом шаблона удостоверений, создаваемых в Visual Studio.
 
    Созданный проект содержит `Microsoft.AspNetCore.Identity.EntityFrameworkCore` пакет, который сохраняет данные удостоверений и схемы SQL Server с помощью [Entity Framework Core](https://docs.microsoft.com/ef/).
    
    ---
 
2.  Настройка службы удостоверений и добавление по промежуточного слоя в `Startup`.

    Службы удостоверений добавляются к приложению в `ConfigureServices` метод `Startup` класса:

    # <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)
    
    [!code-csharp[Main](identity/sample/src/ASPNETv2-IdentityDemo/Startup.cs?name=snippet_configureservices&highlight=7-9,11-28,30-39)]
    
    Эти службы доступны для приложения с помощью [внедрения зависимостей](xref:fundamentals/dependency-injection).
    
    Путем вызова для приложения включена удостоверения `UseAuthentication` в `Configure` метод. `UseAuthentication`Добавление проверки подлинности [по промежуточного слоя](xref:fundamentals/middleware) к конвейеру запросов.
    
    [!code-csharp[Main](identity/sample/src/ASPNETv2-IdentityDemo/Startup.cs?name=snippet_configure&highlight=17)]
    
    # <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)
    
    [!code-csharp[Main](identity/sample/src/ASPNET-IdentityDemo/Startup.cs?name=snippet_configureservices&highlight=7-9,13-34)]
    
    Эти службы доступны для приложения с помощью [внедрения зависимостей](xref:fundamentals/dependency-injection).
    
    Путем вызова для приложения включена удостоверения `UseIdentity` в `Configure` метод. `UseIdentity`Добавляет файл cookie проверки подлинности с помощью [по промежуточного слоя](xref:fundamentals/middleware) к конвейеру запросов.
        
    [!code-csharp[Main](identity/sample/src/ASPNET-IdentityDemo/Startup.cs?name=snippet_configure&highlight=21)]
    
    ---
     
    Дополнительные сведения о время загрузки приложения см. в разделе [запуска приложения](xref:fundamentals/startup).

3.  Создайте пользователя.

    Запустите приложение и щелкните на **зарегистрировать** ссылку.

    Если при первом выполнении этого действия может потребоваться для выполнения миграции. Приложение предложит **применить миграции**:
    
    ![Применить миграции веб-страницы](identity/_static/apply-migrations.png)
    
    Кроме того можно проверить с помощью ASP.NET Core Identity вместе с приложением без постоянных баз данных с помощью базы данных в памяти. Чтобы использовать базу данных в памяти, добавьте ``Microsoft.EntityFrameworkCore.InMemory`` пакета приложения и изменить вызов приложения ``AddDbContext`` в ``ConfigureServices`` следующим образом:

    ```csharp
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseInMemoryDatabase(Guid.NewGuid().ToString()));
    ```
    
    Когда пользователь щелкает **зарегистрировать** ссылку, ``Register`` при вызове действия на ``AccountController``. ``Register`` Действие создает пользователя путем вызова `CreateAsync` на `_userManager` объекта (для ``AccountController`` путем внедрения зависимостей):
 
    [!code-csharp[Main](identity/sample/src/ASPNET-IdentityDemo/Controllers/AccountController.cs?name=snippet_register&highlight=11)]

    Если пользователь успешно создан, пользователь вошел вызове ``_signInManager.SignInAsync``.

    **Примечание:** разделе [подтверждение учетной записи](xref:security/authentication/accconfirm#prevent-login-at-registration) для меры по предотвращению немедленно входа при регистрации.
 
4.  Войти.
 
    Пользователи могут войти, щелкнув **входа** ссылок в верхней части сайта, или может быть переход на страницу входа, если они пытаются получить доступ к части сайта, требующей авторизации. Когда пользователь отправляет форму на странице входа ``AccountController`` ``Login`` вызова действия.

    [!code-csharp[Main](identity/sample/src/ASPNET-IdentityDemo/Controllers/AccountController.cs?name=snippet_login&highlight=13-14)]
 
    ``Login`` Вызывает действие ``PasswordSignInAsync`` на ``_signInManager`` объекта (для ``AccountController`` путем внедрения зависимостей).
 
    Базовый ``Controller`` предоставляемых классами ``User`` свойство, которое можно открыть из методов контроллера. Например, можно перечислить `User.Claims` и принимать решения об авторизации. Дополнительные сведения см. в разделе [авторизации](xref:security/authorization/index).
 
5.  Выйдите из системы.
 
    Щелкнув **Выход** связывать вызовы `LogOut` действие.
 
    [!code-csharp[Main](identity/sample/src/ASPNET-IdentityDemo/Controllers/AccountController.cs?name=snippet_logout&highlight=7)]
 
    Предыдущий код выше вызовы `_signInManager.SignOutAsync` метод. `SignOutAsync` Метод очищает утверждения пользователей, хранящихся в файле cookie.
 
6.  Конфигурация.

    Удостоверение имеет некоторые виды поведения по умолчанию, которые можно переопределить в классе при запуске приложения. Необходимо настроить ``IdentityOptions`` при использовании поведения по умолчанию.

    # <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)
    
    [!code-csharp[Main](identity/sample/src/ASPNETv2-IdentityDemo/Startup.cs?name=snippet_configureservices&highlight=7-9,11-28,30-39)]
    
    # <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)
    
    [!code-csharp[Main](identity/sample/src/ASPNET-IdentityDemo/Startup.cs?name=snippet_configureservices&highlight=13-34)]

    ---
    
    Дополнительные сведения о настройке удостоверений см. в разделе [Настройка удостоверений](xref:security/authentication/identity-configuration).
    
    Можно также настроить тип данных первичный ключ. в разделе [тип данных первичные ключи Настройка удостоверения](xref:security/authentication/identity-primary-key-configuration).
 
7.  Просмотр базы данных.

    Если приложение использует базу данных SQL Server (по умолчанию в Windows и для пользователей Visual Studio), можно просматривать базы данных с приложением, создаваемым. Можно использовать **SQL Server Management Studio**. Кроме того, в Visual Studio, выберите **представление** -> **обозреватель объектов SQL Server**. Подключиться к **(localdb) \MSSQLLocalDB**. База данных с именем, соответствующим  **aspnet - <*имя проекта*>-<*Дата строка*> ** отображается.

    ![Контекстные меню AspNetUsers таблицу базы данных](identity/_static/04-db.png)
    
    Разверните базу данных и его **таблиц**, щелкните правой кнопкой мыши **dbo. AspNetUsers** таблицы и выберите **данные представления**.

## <a name="identity-components"></a>Компоненты идентификаторов

Основная ссылка сборки для системы удостоверений `Microsoft.AspNetCore.Identity`. Данный пакет содержит базовый набор интерфейсов для ASP.NET Core Identity и включается по `Microsoft.AspNetCore.Identity.EntityFrameworkCore`.

Эти зависимости необходимы для использования в приложениях ASP.NET Core системы удостоверений.

* `Microsoft.AspNetCore.Identity.EntityFrameworkCore`-Типы, необходимые для использования с Entity Framework Core удостоверений.

* `Microsoft.EntityFrameworkCore.SqlServer`-Entity Framework Core — технология доступа к данных, рекомендуемые корпорации Майкрософт для реляционных баз данных, как SQL Server. Для тестирования, можно использовать `Microsoft.EntityFrameworkCore.InMemory`.

* `Microsoft.AspNetCore.Authentication.Cookies`-Промежуточное по, которая позволяет приложению использовать проверку подлинности на основе файлов cookie.

## <a name="migrating-to-aspnet-core-identity"></a>Переход к удостоверению ASP.NET Core

Дополнительные сведения и инструкции по миграции существующего личности магазине см. в разделе [миграции проверку подлинности и удостоверение](xref:migration/identity).

## <a name="next-steps"></a>Дальнейшие действия

* [Миграция на другой метод проверки подлинности и другие удостоверения](xref:migration/identity)
* [Подтверждение учетной записи и восстановление пароля](xref:security/authentication/accconfirm)
* [Двухфакторная проверка подлинности с помощью SMS](xref:security/authentication/2fa)
* [Включение проверки подлинности с помощью Facebook, Google и других внешних поставщиков](xref:security/authentication/social/index)
