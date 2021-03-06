---
title: "Привязка модели"
author: rick-anderson
description: 
keywords: ASP.NET Core
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: b355a48e-a15c-4d58-b69c-899763613a97
ms.technology: aspnet
ms.prod: asp.net-core
uid: mvc/models/model-binding
ms.openlocfilehash: 40aa105dcf06b269025d0c44e5cd7bffef271e9d
ms.sourcegitcommit: fe880bf4ed1c8116071c0e47c0babf3623b7f44a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/21/2017
---
# <a name="model-binding"></a>Привязка модели

По [Рэйчел Аппель](https://github.com/rachelappel)

## <a name="introduction-to-model-binding"></a>Общие сведения о привязки модели

Привязка модели в ASP.NET Core MVC сопоставляет данные из HTTP-запросы с параметрами метода действия. Параметры могут быть простые типы, такие как строки, целые числа или значений с плавающей запятой, или они могут быть сложные типы. Это прекрасная возможность MVC, так как сопоставление входящих данных аналога часто повторяющихся сценарий, независимо от размера и сложности данных. MVC решает эту проблему, Воспринимая привязки, поэтому разработчикам не нужно сохранять перезаписи другая версия этого же кода в каждом приложении. Запись текста в тип преобразователя кода сложным и подвержены ошибкам.

## <a name="how-model-binding-works"></a>Как работает привязка модели

Когда MVC получает запрос HTTP, он направляет его в метод определенного действия контроллера. Он определяет, какой метод действия, необходимо запускать на основании возможности маршрутизации данных, а затем привязывает значения из HTTP-запроса с параметрами метода действия. Например рассмотрим следующий URL-адрес:

`http://contoso.com/movies/edit/2`

Поскольку шаблон маршрута выглядит следующим образом, `{controller=Home}/{action=Index}/{id?}`, `movies/edit/2` направляет `Movies` контроллера и его `Edit` метода действия. Он также принимает необязательный параметр с именем `id`. Код для метода действия должен выглядеть примерно следующим образом:

```csharp
public IActionResult Edit(int? id)
   ```

Примечание: Строк в URL-адрес маршрута не учитывается регистр.

MVC пытается привязать данные запроса с параметрами действия по имени. MVC будет искать значения для каждого параметра, используя имя параметра и имена его открытые устанавливаемые свойства. В приведенном выше примере параметр единственным действием называется `id`, которое MVC привязывает значение с тем же именем в значения маршрута. Помимо значений маршрута MVC будет привязывать данные из различных частей запроса, и она делает это в заданном порядке. Ниже приведен список источников данных в порядке, в котором их просматривает привязки модели.

1. `Form values`: Это значений формы, перейдите в HTTP-запроса, с помощью метода POST. (включая запросы POST jQuery).

2. `Route values`: Набор значений маршрута, предоставляемые [маршрутизации](../../fundamentals/routing.md)

3. `Query strings`: Часть URI, строка запроса.

<!-- DocFX BUG
The link works but generates an error when building with DocFX
@fundamentals/routing
[Routing](xref:fundamentals/routing)
-->

Примечание: Формы, значения, данные о маршруте и запросов строки хранятся как пары "имя значение".

Так как привязки модели и ответы для ключа с именем `id` и нет ничего с именем `id` значений формы, он перемещен в поиске разделу значения маршрута. В нашем примере это совпадение. Привязки, а значение преобразуется в целочисленное значение 2. Тот же запрос, с помощью редактирования (идентификатор строки) будет преобразовать в строку «2».

До сих в примере простых типов. В MVC простые типы являются любого .NET примитивного типа или типа с преобразователь строкового типа. Если параметр метода действия были класса, такие как `Movie` тип, который содержит простые и сложные типы, как свойства, будет привязка модели MVC по-прежнему обрабатываются он хорошо. Она использует отражение и рекурсии для обхода свойств сложных типов, поиск совпадения. Привязка модели ищет шаблон *parameter_name.property_name* для привязки значений к свойствам. Если он не находит совпадающие значения этой формы, будет пытаться осуществить привязку с помощью имени свойства. Для этих типов, таких как `Collection` типов привязки модели ищет совпадения с *parameter_name [index]* или просто *[index]*. Модель рассматривает привязки `Dictionary` аналогичным образом, типы, запрашивая *parameter_name [key]* или просто *[key]*, при условии, что ключи являются простыми типами. Ключи, которые поддерживаются совпадать с именами полей HTML и вспомогательных функций тегов, созданных для того же типа модели. Это позволяет значения кругового пути, чтобы поля формы быть заполнено с входными данными пользователя для их удобства, например, при связанных данных из Создание или изменение не прошел проверку.

Для привязки к класса должен иметь открытый конструктор по умолчанию и элемент для привязки должны быть открытыми свойствами для записи. Когда происходит привязка модели класс только создать экземпляр с помощью открытый конструктор по умолчанию, можно задать свойства.

При привязке параметра привязки модели прекращает поиск значений с таким именем, и он переходит к привязать следующего параметра. В противном случае поведение привязки модели по умолчанию задает параметры к значениям по умолчанию, в зависимости от их типа:

* `T[]`: За исключением элемента массивы типа `byte[]`, привязка задает параметры типа `T[]` для `Array.Empty<T>()`. Массивы типа `byte[]` присваиваются `null`.

* Ссылочные типы: Привязки создает экземпляр класса с помощью конструктора по умолчанию без задания свойств. Однако модели привязки наборов `string` параметры `null`.

* Типы, допускающие значение NULL: Типы, допускающие значение NULL, присваивается `null`. В приведенном выше примере модель привязки наборов `id` для `null` , так как он имеет тип `int?`.

* Типы значений: Запретом типы значений типа `T` присваиваются `default(T)`. Например, привязка модели параметру `int id` значение 0. Можно с помощью проверки модели или типы, допускающие значение NULL, не полагаясь на значения по умолчанию.

Если привязка не выполняется, MVC не вызывают ошибку. Каждое действие, который принимающее вводимые пользователем данные следует проверять `ModelState.IsValid` свойство.

Примечание: Каждая запись на контроллере `ModelState` свойство `ModelStateEntry` содержащий `Errors` свойство. Редко бывает необходимо самостоятельно запроса этой коллекции. Взамен рекомендуется использовать `ModelState.IsValid` .

Кроме того существуют некоторые особые типы данных, MVC необходимо учитывать при выполнении привязки модели:

* `IFormFile`, `IEnumerable<IFormFile>`: Один или несколько переданные файлы, которые являются частью HTTP-запроса.

* `CancellationToken`: Используется для отмены действия в асинхронных контроллеров.

Эти типы можно привязать к параметрам действия или к свойствам типа класса.

После привязки модели [проверки](validation.md) происходит. Привязка модели по умолчанию отлично подходит для большинства сценариев разработки. Он также является расширяемым, если у вас есть потребности можно настроить встроенное поведение.

## <a name="customize-model-binding-behavior-with-attributes"></a>Настройка поведения привязки модели с атрибутами

MVC содержит несколько атрибутов, которые можно использовать для направления поведения привязки модели по умолчанию для другого источника. Например, можно указать будет ли привязка обязательна для свойства или вообще его не случится с помощью `[BindRequired]` или `[BindNever]` атрибуты. Кроме того можно переопределить источник данных по умолчанию и укажите источник данных связывателя модели. Ниже приведен список атрибутов привязки модели.

* `[BindRequired]`: Этот атрибут добавляет ошибку модели состояние, если невозможно выполнить привязку.

* `[BindNever]`: Указывает связывателя модели, никогда не связано с этим параметром.

* `[FromHeader]`, `[FromQuery]`, `[FromRoute]`, `[FromForm]`: Используются для указания точного привязки источника, необходимо применить.

* `[FromServices]`: Этот атрибут содержит [внедрения зависимостей](../../fundamentals/dependency-injection.md) для привязки параметров из служб.

* `[FromBody]`: Используется настроенный модули форматирования для привязки данных в тексте запроса. Модуль форматирования выбирается в зависимости от типа содержимого запроса.

* `[ModelBinder]`: Используется для переопределения связыватель модели по умолчанию, источник привязки и имя.

Атрибуты имеют очень полезных инструментов, когда необходимо переопределить поведение по умолчанию привязки модели.

## <a name="binding-formatted-data-from-the-request-body"></a>Привязка форматированные данные из текста запроса

Запрос данные могут поступать в различных форматах, включая JSON, XML и многие другие. При использовании атрибута [FromBody] для указания, что нужно привязать параметр к данным в тексте запроса, MVC использует настроенный набор модулей форматирования для обработки данных запрос на основе типа содержимого. По умолчанию MVC включает `JsonInputFormatter` класса для обработки данных JSON, но можно добавить дополнительные модули форматирования для обработки XML и других пользовательских форматов.

> [!NOTE]
> Может существовать не более одного параметра каждого действия с `[FromBody]`. Времени выполнения ASP.NET Core MVC делегирует ответственность за чтения в поток запроса модуль форматирования. Когда для параметра прочитать поток запроса, это обычно невозможно прочитать поток запроса для привязки других `[FromBody]` параметров.

> [!NOTE]
> `JsonInputFormatter` Форматирования по умолчанию и основанное на [Json.NET](https://www.newtonsoft.com/json).

ASP.NET выбирает входной модули форматирования на основе [Content-Type](https://www.w3.org/Protocols/rfc1341/4_Content-Type.html) заголовок и тип параметра, при отсутствии атрибута, примененного к нему задание, в противном случае. Если вы хотите использовать XML или другой формат необходимо настроить его в *файла Startup.cs* файл, но возможно, сначала нужно получить ссылку на `Microsoft.AspNetCore.Mvc.Formatters.Xml` с помощью NuGet. Код запуска должен выглядеть примерно следующим образом:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddXmlSerializerFormatters();
   }
```

Код в *файла Startup.cs* файл содержит `ConfigureServices` метод с `services` аргумента можно использовать для построения служб для приложения ASP.NET. В этом примере мы добавляем модуль форматирования XML как служба, которая будет предоставлять MVC для этого приложения. `options` Аргумент, переданный в `AddMvc` метод позволяет добавлять и управлять фильтры, форматирования и другие системные параметры из MVC при запуске приложения. Затем примените `Consumes` контроллера классы или методы действий для работы с необходимый формат.

### <a name="custom-model-binding"></a>Привязка пользовательских модели

Привязка модели можно расширить, добавив собственные пользовательские связыватели. Дополнительные сведения о [пользовательскую модель привязки](../advanced/custom-model-binding.md).
