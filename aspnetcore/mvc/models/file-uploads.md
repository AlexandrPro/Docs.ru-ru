---
title: "В ASP.NET Core передачи файлов"
author: ardalis
description: "Инструкции по использованию привязки модели и потоковой передачи для передачи файлов в ASP.NET Core MVC."
keywords: "Привязки, IFormFile, потоковые модели ASP.NET Core, отправки файла"
ms.author: riande
manager: wpickett
ms.date: 07/05/2017
ms.topic: article
ms.assetid: ebc98159-a028-4a94-b06c-43981c79c6be
ms.technology: aspnet
ms.prod: asp.net-core
uid: mvc/models/file-uploads
ms.openlocfilehash: e8608a46d6688df8da6c665a25b6f4db5f480461
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/10/2017
---
# <a name="file-uploads-in-aspnet-core"></a>В ASP.NET Core передачи файлов

По [Стив Смит](https://ardalis.com/)

Действия ASP.NET MVC поддерживает возможность отправки одного или нескольких файлов, используя простую модель привязки для файлов меньшего размера или потоковая передача больших файлов.

[Просмотреть или загрузить пример из GitHub](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/models/file-uploads/sample/FileUploadSample)

## <a name="uploading-small-files-with-model-binding"></a>Передача небольших файлов с использованием привязки модели

Чтобы отправить небольшие файлы, можно использовать составную HTML-форму или создать запрос POST с использованием JavaScript. Ниже приводится пример формы, с помощью Razor, поддерживающей несколько отправленных файлов.

```html
<form method="post" enctype="multipart/form-data" asp-controller="UploadFiles" asp-action="Index">
    <div class="form-group">
        <div class="col-md-10">
            <p>Upload one or more files using this form:</p>
            <input type="file" name="files" multiple />
        </div>
    </div>
    <div class="form-group">
        <div class="col-md-10">
            <input type="submit" value="Upload" />
        </div>
    </div>
</form>
```

Для поддержки передачи файлов, необходимо указать HTML-формы `enctype` из `multipart/form-data`. `files` Входной элемент, показанный выше поддерживает передачу нескольких файлов. Пропустите `multiple` атрибут для этого входного элемента, чтобы разрешить только один файл для отправки. Выше Разметка отображает в браузере как:

![Форму для загрузки файла](file-uploads/_static/upload-form.png)

Отдельные файлы, переданные на сервер можно получить с помощью [привязки модели](xref:mvc/models/model-binding) с помощью [IFormFile](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.iformfile) интерфейса. `IFormFile`имеет следующую структуру:

```csharp
public interface IFormFile
{
    string ContentType { get; }
    string ContentDisposition { get; }
    IHeaderDictionary Headers { get; }
    long Length { get; }
    string Name { get; }
    string FileName { get; }
    Stream OpenReadStream();
    void CopyTo(Stream target);
    Task CopyToAsync(Stream target, CancellationToken cancellationToken = null);
}
```

> [!WARNING]
> Не полагайтесь на или доверяйте `FileName` свойству без проверки. `FileName` Свойства должено использоваться только в целях отображения.

При отправке файлов с использованием привязки модели и `IFormFile` интерфейса, метод действия может принимать один `IFormFile` или `IEnumerable<IFormFile>` (или `List<IFormFile>`) представляя несколько файлов. Следующий пример просматривает один или несколько переданных файлов, сохраняет их в локальной файловой системе и возвращает общее число и размер переданных файлов.

[!INCLUDE [GetTempFileName](../../includes/GetTempFileName.md)]

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Controllers/UploadFilesController.cs?name=snippet1)]

Файлы загружаются с помощью `IFormFile` метод помещаются в буфер в памяти или на диске, на веб-сервере перед обработкой. Внутри метода действия `IFormFile` содержимое доступно в виде потока. В дополнение к локальной файловой системе, файлах может передаваться в [хранилища больших двоичных объектов Azure](https://azure.microsoft.com/documentation/articles/vs-storage-aspnet5-getting-started-blobs/) или [Entity Framework](https://docs.microsoft.com/ef/core/index).

Для хранения двоичного файла данных в базе данных с использованием Entity Framework, определите свойство типа `byte[]` на сущность:

```csharp
public class ApplicationUser : IdentityUser
{
    public byte[] AvatarImage { get; set; }
}
```

Укажите свойства viewmodel типа `IFormFile`:

```csharp
public class RegisterViewModel
{
    // other properties omitted

    public IFormFile AvatarImage { get; set; }
}
```

> [!NOTE]
> `IFormFile`можно использовать напрямую как параметр метода действия или свойства viewmodel, как показано выше.

Копировать `IFormFile` в поток и сохраните его на массив байтов:

```csharp
// POST: /Account/Register
[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Register(RegisterViewModel model)
{
    ViewData["ReturnUrl"] = returnUrl;
    if  (ModelState.IsValid)
    {
        var user = new ApplicationUser {
          UserName = model.Email,
          Email = model.Email
        };
        using (var memoryStream = new MemoryStream())
        {
            await model.AvatarImage.CopyToAsync(memoryStream);
            user.AvatarImage = memoryStream.ToArray();
        }
    // additional logic omitted
    
    // Don't rely on or trust the model.AvatarImage.FileName property 
    // without validation.
}
```

> [!NOTE]
> Будьте внимательны при хранения двоичных данных в реляционных базах данных, как это может отрицательно повлиять на производительность.

## <a name="uploading-large-files-with-streaming"></a>Передача больших файлов с помощью потоковой передачи

Если размер или частоты передачи файлов вызывает проблемы ресурсов для приложения, рассмотрите возможность потоковой передачи передачу файла, а не используя буфер в полном объеме как показанный выше подход привязки модели. При использовании `IFormFile` и привязки модели — это гораздо проще решение, потоковой передачи требуется ряд действий для реализации должным образом.

> [!NOTE]
> Любой файл одного буферизации, которых превышает 64 КБ будет перемещен из оперативной памяти во временный файл на диске на сервере. Ресурсы (диск ОЗУ), используемые передачи файлов зависят от числа и размера передачи файлов. Потоковая передача не столько производительности, о масштабе. При попытке буфер слишком много передачи веб-узла произойдет сбой в случае нехватки памяти или места на диске.

В следующем примере показано использование JavaScript/Angular можно передавать на действия контроллера. Токен сложные файл создается с помощью пользовательского атрибута фильтра и передаются в HTTP-заголовки, а не в теле запроса. Так как метод действия обрабатывает отправляемые вами данные непосредственно, другой фильтр отключил привязки модели. В действии, содержимого формы считываются с использованием `MultipartReader`, который читает каждый отдельный `MultipartSection`, обработка файла или сохранение содержимого соответствующим образом. После чтения всех разделов, действие выполняет собственную привязки модели.

Начальное действие загружает форму и сохраняет токен сложные в файле cookie (через `GenerateAntiforgeryTokenCookieForAjax` атрибут):

```csharp
[HttpGet]
[GenerateAntiforgeryTokenCookieForAjax]
public IActionResult Index()
{
    return View();
}
```

Атрибут использует встроенные ASP.NET Core [Antiforgery](xref:security/anti-request-forgery) поддержки, чтобы задать файл cookie с маркера запроса:

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Filters/GenerateAntiforgeryTokenCookieForAjaxAttribute.cs?name=snippet1)]

Angular автоматически передает сложные токенs в заголовок запроса с именем `X-XSRF-TOKEN`. Приложение ASP.NET Core MVC настраивается для ссылки на этот заголовок в конфигурации в *файла Startup.cs*:

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Startup.cs?name=snippet1)]

`DisableFormValueModelBinding` Атрибут, показанный ниже, используется для отключения привязки модели для `Upload` метода действия.

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Filters/DisableFormValueModelBindingAttribute.cs?name=snippet1)]

После отключения привязки модели, `Upload` метод действия не принимают параметры. Он работает непосредственно с `Request` свойство `ControllerBase`. Объект `MultipartReader` используется для чтения в каждом разделе. Файл сохраняется с именем файла, GUID и хранения ключей и значений данных в `KeyValueAccumulator`. Как только все разделы были считаны, содержимое `KeyValueAccumulator` используются для привязки данных формы для типа модели.

Полный `Upload` метода показан ниже:

[!INCLUDE [GetTempFileName](../../includes/GetTempFileName.md)]

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Controllers/StreamingController.cs?name=snippet1)]

## <a name="troubleshooting"></a>Устранение неполадок

Ниже приведены некоторые наиболее часто возникающие при работе с передача файлов и их возможные решения.

### <a name="unexpected-not-found-error-with-iis"></a>Непредвиденная ошибка не найден в IIS

Следующая ошибка указывает на попытку отправки файла превышает сервера настроенного `maxAllowedContentLength`:

```
HTTP 404.13 - Not Found
The request filtering module is configured to deny a request that exceeds the request content length.
```

Значение по умолчанию — `30000000`, который составляет примерно 28.6 МБ. Значение можно настроить, изменив *web.config*:

```xml
<system.webServer>
  <security>
    <requestFiltering>
      <!-- This will handle requests up to 50MB -->
      <requestLimits maxAllowedContentLength="52428800" />
    </requestFiltering>
  </security>
</system.webServer>
```

Этот параметр применяется только к службам IIS. Поведение не происходит по умолчанию при размещении на Kestrel. Дополнительные сведения см. в разделе [ограничения запросов \<requestLimits\>](https://docs.microsoft.com/iis/configuration/system.webServer/security/requestFiltering/requestLimits/).

### <a name="null-reference-exception-with-iformfile"></a>Пустая ссылка на исключение IFormFile

Если ваш контроллер принимает передать файлы с помощью `IFormFile` , но вы можете найти значение всегда равно null, убедитесь, что формы HTML: необходимо указать `enctype` значение `multipart/form-data`. Если этот атрибут не задан для `<form>` элемент, не происходит отправка файла и любую границу `IFormFile` аргументы будут иметь значение null.
