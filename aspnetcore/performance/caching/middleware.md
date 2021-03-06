---
title: "Кэширование по промежуточного слоя в ASP.NET Core ответов"
author: guardrex
description: "Дополнительные сведения о настройке и использовании по промежуточного слоя, кэширование ответа в приложениях ASP.NET Core."
ms.author: riande
manager: wpickett
ms.date: 08/22/2017
ms.topic: article
ms.prod: asp.net-core
uid: performance/caching/middleware
ms.openlocfilehash: f3312d0c333b47169c71891eea79f03be0abcfa3
ms.sourcegitcommit: 216dfac27542f10a79274a9ce60dc449e888ed20
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/29/2017
---
# <a name="response-caching-middleware-in-aspnet-core"></a>Кэширование по промежуточного слоя в ASP.NET Core ответов

По [Latham Люк](https://github.com/guardrex) и [Джон Luo](https://github.com/JunTaoLuo)

[Просмотреть или скачать образец кода](https://github.com/aspnet/Docs/tree/master/aspnetcore/performance/caching/middleware/samples) ([как скачивать](xref:tutorials/index#how-to-download-a-sample))

Этот документ содержит сведения о способах настройки по промежуточного слоя ответа кэширование в приложениях ASP.NET Core. По промежуточного слоя определяет, когда ответов может быть кэширован, ответы на магазины и служит ответы из кэша. Основные сведения о кэшировании HTTP и `ResponseCache` см. в разделе [кэширование ответов](response.md).

## <a name="package"></a>Пакет
Чтобы включить по промежуточного слоя в проекте, добавьте ссылку на [ `Microsoft.AspNetCore.ResponseCaching` ](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/) пакета или использовать [ `Microsoft.AspNetCore.All` ](https://www.nuget.org/packages/Microsoft.AspNetCore.All/) пакета.

## <a name="configuration"></a>Конфигурация
В `ConfigureServices`, добавление в коллекцию службы по промежуточного слоя.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

[!code-csharp[Main](middleware/samples/2.x/Program.cs?name=snippet1&highlight=4)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

[!code-csharp[Main](middleware/samples/1.x/Startup.cs?name=snippet1&highlight=3)]

---

Настройка приложения для использования по промежуточного слоя с `UseResponseCaching` метод расширения, который добавляет по промежуточного слоя в конвейер обработки запросов. Добавляет в пример приложения [ `Cache-Control` ](https://tools.ietf.org/html/rfc7234#section-5.2) заголовок в ответ, который кэширует кэшируемых ответов на срок до 10 секунд. Образец отправляет [ `Vary` ](https://tools.ietf.org/html/rfc7231#section-7.1.4) заголовок для настройки по промежуточного слоя для обслуживания только если кэшированный ответ [ `Accept-Encoding` ](https://tools.ietf.org/html/rfc7231#section-5.3.4) заголовок последующих запросов таковыми исходного запроса.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

[!code-csharp[Main](middleware/samples/2.x/Program.cs?name=snippet1&highlight=8)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

[!code-csharp[Main](middleware/samples/1.x/Startup.cs?name=snippet2&highlight=3)]

---

По промежуточного слоя кэширования ответа только кэширует ответы сервера 200 (ОК). Все другие ответы, включая [страницы ошибок](xref:fundamentals/error-handling), учитываются по промежуточного слоя.

> [!WARNING]
> Ответы с содержимым для авторизованных клиентов должен быть помечен как не может быть кэширован, чтобы предотвратить по промежуточного слоя в хранении и обслуживании этих ответов. В разделе [условия для кэширования](#conditions-for-caching) сведения о том, как по промежуточного слоя определяет, является ли ответ может быть кэширован.

## <a name="options"></a>Параметры
По промежуточного слоя предоставляет три варианта управлении кэшем ответа.

| Параметр                | Значение по умолчанию |
| --------------------- | ------------- |
| UseCaseSensitivePaths | Определяет, если ответы кэшируются на пути с учетом регистра.</p><p>Значение по умолчанию — `false`. |
| MaximumBodySize       | Максимальный размер кэшируемого текст ответа в байтах.</p>Значение по умолчанию — `64 * 1024 * 1024` (64 МБ). |
| Значение SizeLimit             | Предельный размер для по промежуточного слоя кэша ответа в байтах. Значение по умолчанию — `100 * 1024 * 1024` (100 МБ). |

В следующем примере настраивается по промежуточного слоя для кэширования ответов меньше или равна 1 024 байт, с использованием путей, с учетом регистра, хранение ответы `/page1` и `/Page1` отдельно.

```csharp
services.AddResponseCaching(options =>
{
    options.UseCaseSensitivePaths = true;
    options.MaximumBodySize = 1024;
});
```

## <a name="varybyquerykeys"></a>VaryByQueryKeys
При использовании MVC, `ResponseCache` атрибут задает параметры, необходимые для установки соответствующие заголовки для кэширования ответа. Единственным параметром `ResponseCache` атрибут, строго требует по промежуточного слоя — `VaryByQueryKeys`, которой не соответствует фактическое заголовка HTTP. Дополнительные сведения см. в разделе [ResponseCache атрибут](response.md#responsecache-attribute).

Если не используется MVC, можно варьировать кэширование ответов с `VaryByQueryKeys` компонентов. Используйте `ResponseCachingFeature` непосредственно из `IFeatureCollection` из `HttpContext`:

```csharp
var responseCachingFeature = context.HttpContext.Features.Get<IResponseCachingFeature>();
if (responseCachingFeature != null)
{
    responseCachingFeature.VaryByQueryKeys = new[] { "MyKey" };
}
```

## <a name="http-headers-used-by-response-caching-middleware"></a>Заголовки HTTP, используемые по промежуточного слоя кэширования ответа
Кэширование по промежуточного слоя ответов настраивается с помощью заголовков HTTP. Ниже перечислены соответствующие заголовки с заметками на их влияние на кэширование.

| Header | Подробные сведения |
| ------ | ------- |
| Авторизация | Ответ не кэшируется. Если заголовок существует. |
| Cache-Control | По промежуточного слоя рассматривает только кэширование ответов, отмеченные `public` директива кэша. Можно управлять кэшированием со следующими параметрами:<ul><li>max-age</li><li>Устаревшая max &#8224;</li><li>Min нуля</li><li>должен revalidate</li><li>Нет-cache</li><li>нет хранилища</li><li>только если кэширования</li><li>private</li><li>public</li><li>s-maxage</li><li>Proxy-revalidate &#8225;</li></ul>&#8224; Если указано без ограничений `max-stale`, по промежуточного слоя не выполняет никаких действий.<br>&#8225; `proxy-revalidate` действует так же, как `must-revalidate`.<br><br>Дополнительные сведения см. в разделе [RFC 7231: запрос директивы управления кэшем](https://tools.ietf.org/html/rfc7234#section-5.2.1). |
| Директивы pragma | Объект `Pragma: no-cache` заголовка в запросе дает тот же эффект, что `Cache-Control: no-cache`. Этот заголовок переопределяется соответствующей директивы в `Cache-Control` заголовка, если он имеется. Для обеспечения обратной совместимости с HTTP/1.0 во внимание. |
| Set-Cookie | Ответ не кэшируется. Если заголовок существует. |
| Различаются | `Vary` Заголовок используется другой заголовок для изменения кэшированного ответа. Например, можно кэшировать ответы при кодировании, включая `Vary: Accept-Encoding` заголовок, который кэширует ответы для запросов с заголовками `Accept-Encoding: gzip` и `Accept-Encoding: text/plain` отдельно. Значение заголовка ответа `*` никогда не хранится. |
| Срок действия истекает | Как устаревшие средством этот заголовок ответа не хранятся или получить, если не переопределено другим `Cache-Control` заголовки. |
| If-None-Match | Полный ответ подано из кэша, если значение не `*` и `ETag` ответа не соответствует ни одному из предлагаемых значений. В противном случае выдается в отклике 304 (не изменено). |
| If-Modified-Since | Если `If-None-Match` заголовок отсутствует, полный ответ подано из кэша, если дата кэшированного ответа новее, чем указанное значение. В противном случае выдается в отклике 304 (не изменено). |
| Дата | При выполнении из кэша, `Date` задать заголовок по промежуточного слоя, если он не указан исходный ответа. |
| Content-Length | При выполнении из кэша, `Content-Length` задать заголовок по промежуточного слоя, если он не указан исходный ответа. |
| Срок действия | `Age` Заголовок, отправляемый в ответ исходного игнорируется. По промежуточного слоя вычисляет новое значение, при выполнении кэшированного ответа. |

## <a name="caching-respects-request-cache-control-directives"></a>Кэширование соблюдает директивы управления кэшем запроса

По промежуточного слоя соблюдает правилами [кэширования HTTP 1.1 спецификации](https://tools.ietf.org/html/rfc7234#section-5.2). Правила требуют кэша соблюдать является допустимым для `Cache-Control` заголовок, отправленных клиентом. В спецификации, клиент может выполнять запросы с `no-cache` значение заголовка и force сервера, чтобы создать новый ответ для каждого запроса. В настоящее время нет не контроль разработчика над этот режим кэширования при использовании по промежуточного слоя, так как по промежуточного слоя соответствует спецификации официальный кэширования.

[Будущих улучшений по промежуточного слоя](https://github.com/aspnet/ResponseCaching/issues/96) разрешает настройки по промежуточного слоя сценариях кэширования, где запрос `Cache-Control` заголовок игнорироваться при принятии решения о обслуживать кэшированный ответ. При получении больший контроль над поведение кэширования изучение других функций кэширования ASP.NET Core. См. указанные ниже разделы.

* [Кэширование в памяти](xref:performance/caching/memory)
* [Работа с распределенного кэша](xref:performance/caching/distributed)
* [Кэшировать вспомогательный тег в ядре ASP.NET MVC](xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper)
* [Вспомогательный тег распределенного кэша](xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper)

## <a name="troubleshooting"></a>Устранение неполадок
Если режим кэширования не должным образом, проверка ответов может быть кэширован и может быть предоставлен из кэша с помощью проверки запроса заголовки входящих и исходящих заголовки ответа. Включение [входа](xref:fundamentals/logging/index) может помочь при отладке. Журналы по промежуточного слоя, кэширование поведение и при получении ответа из кэша.

При тестирование и устранение неполадок поведение кэширования, браузер может задать заголовки запроса, влияющих на кэширование нежелательно способами. Например, браузер может задать `Cache-Control` заголовка `no-cache` при обновлении страницы. Следующие средства можно явно задать заголовки запроса и являются предпочтительными для тестирования кэширования:

* [Fiddler](http://www.telerik.com/fiddler)
* [Firebug](http://getfirebug.com/)
* [Postman](https://www.getpostman.com/)

### <a name="conditions-for-caching"></a>Условия для кэширования
* Запрос должен привести ответ 200 (ОК) с сервера.
* Метод запроса должно быть GET или HEAD.
* Терминалов по промежуточного слоя, например по промежуточного слоя статических файлов не должен обработать ответ перед по промежуточного слоя кэширования ответа.
* `Authorization` Заголовок не должен присутствовать.
* `Cache-Control`Параметры заголовка должен быть допустимым, а ответ должен быть помечен `public` и не помечен `private`.
* `Pragma: no-cache` Заголовка значение не должно быть Если `Cache-Control` заголовок отсутствует, как `Cache-Control` переопределяет заголовок `Pragma` заголовка, если они есть.
* `Set-Cookie` Заголовок не должен присутствовать.
* `Vary`Параметры заголовка должно быть допустимым и не равно `*`.
* `Content-Length` Значение заголовка (если задать) должно соответствовать размеру текста ответа.
* [IHttpSendFileFeature](/aspnet/core/api/microsoft.aspnetcore.http.features.ihttpsendfilefeature) не используется.
* Ответ не должно быть устаревшей в соответствии с `Expires` заголовок и `max-age` и `s-maxage` кэшировать директивы.
* Буферизация прошла успешно, и размер ответа меньше, чем настроенное или по умолчанию `SizeLimit`.
* Требуется ответ может быть кэширован в соответствии с [RFC 7234](https://tools.ietf.org/html/rfc7234) спецификации. Например `no-store` директива не должен существовать в поля заголовка запроса или ответа. В разделе *раздел 3: хранение ответы в кэше* из [RFC 7234](https://tools.ietf.org/html/rfc7234) подробные сведения.

> [!NOTE]
> Сложные системы, для создания токенов безопасности для предотвращения подделки межсайтовых запросов (CSRF) атак наборы `Cache-Control` и `Pragma` заголовки `no-cache` , чтобы ответы не кэшируются.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Запуск приложения](xref:fundamentals/startup)
* [ПО промежуточного слоя](xref:fundamentals/middleware)
* [Кэширование в памяти](xref:performance/caching/memory)
* [Работа с распределенного кэша](xref:performance/caching/distributed)
* [Обнаруживать изменения с маркерами изменения](xref:fundamentals/primitives/change-tokens)
* [Кэширование ответов](xref:performance/caching/response)
* [Вспомогательный тег кэша](xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper)
* [Вспомогательный тег распределенного кэша](xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper)
