---
title: "Ограничение времени существования полезных данных, защищенных"
author: rick-anderson
description: "В этом документе объясняется, как ограничить время существования защищенных полезных данных с помощью интерфейсов API защиты данных ASP.NET Core."
keywords: "ASP.NET Core, защиты данных, время жизни полезных данных"
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: 000175e2-10fc-43dd-bfc2-51e004b97b44
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/consumer-apis/limited-lifetime-payloads
ms.openlocfilehash: 3fe2e97db118a92cf6fa003b9ce8a9dedf253136
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/10/2017
---
# <a name="limiting-the-lifetime-of-protected-payloads"></a>Ограничение времени существования полезных данных, защищенных

Существуют сценарии, где разработчик приложения хочет создать защищенный полезные данные, срок действия истекает через установленный период времени. Например защищенные полезных данных может представлять маркер сброса пароля, которая должна только действовать в течение одного часа. Возможна наверняка разработчикам создавать свои собственные формат полезных данных, который содержит внедренные срок действия, и дополнительно разработчикам может потребоваться это сделать, но для большинства разработчиков управление этих сроков действия может увеличиваться утомительным.

Чтобы облегчить эту задачу для наших аудитория разработчиков, пакет [Microsoft.AspNetCore.DataProtection.Extensions](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Extensions/) содержит API служебной программы для создания полезных данных, которые автоматически истекает после заданного периода времени. Эти API-интерфейсы зависания пределами `ITimeLimitedDataProtector` типа.

## <a name="api-usage"></a>Использование API

`ITimeLimitedDataProtector` Интерфейс является основной интерфейс для защиты и снятие защиты полезных данных ограниченной по времени и автоматическим истечением срока действия. Для создания экземпляра `ITimeLimitedDataProtector`, вам потребуется сначала экземпляр обычной [IDataProtector](overview.md) создан с определенной цели. Один раз `IDataProtector` экземпляр доступен, вызовите `IDataProtector.ToTimeLimitedDataProtector` метод расширения для получения предохранитель с возможностями встроенных истечение срока действия.

`ITimeLimitedDataProtector`предоставляет следующие методы API контактную зону и расширения:

* CreateProtector (строка назначение): ITimeLimitedDataProtector - этот API, похожие на существующие `IDataProtectionProvider.CreateProtector` в том, что он может использоваться для создания [цели цепочки](purpose-strings.md) из предохранителя корневой ограниченной по времени.

* Защита (byte [] открытого текста, истечение срока действия DateTimeOffset): byte]

* Защита (открытого текста byte [], время существования TimeSpan): byte]

* Защита (byte [] текстовая): byte]

* Защита (текстовая строка, истечение срока действия DateTimeOffset): строка

* Защита (обычный текст строки, время существования TimeSpan): строка

* Защита (текстовая строка): строка

Помимо базовых `Protect` методов, принимающих только обычный текст, существуют новые перегрузки, которые разрешается указывать срок действия полезных данных. Дата окончания срока действия может быть указан как абсолютная Дата (через `DateTimeOffset`) или как относительное время (из текущей системы времени через `TimeSpan`). При вызове перегрузки, которая не принимает срока действия полезных данных предполагается никогда не истекает.

* Снять защиту (byte [] protectedData, ожидания истечения срока действия DateTimeOffset): byte]

* Снять защиту (protectedData byte []): byte]

* Снять защиту (ожидания истечения срока действия DateTimeOffset, protectedData строка): строка

* Снять защиту (строка protectedData): строка

`Unprotect` Методы возвращают незащищенных исходных данных. Если полезные данные еще не истек, абсолютный срок действия возвращается как Необязательный выходной параметр вместе с исходной незащищенные данные. Если истек срок действия полезных данных всех перегруженных версий метода Unprotect вызовет CryptographicException.

>[!WARNING]
> Не рекомендуется использовать эти API-интерфейсы для защиты полезных нагрузок, требующих сохраняемости долгосрочной или не определено. «Я допускают для защищенных нагрузок быть невозможности восстановить после месяца?» может служить хорошим правило; Если ответ не затем разработчики следует рассмотреть альтернативные API-интерфейсы.

Пример ниже, используется [пути кода не DI](../configuration/non-di-scenarios.md) для экземпляров системы защиты данных. Чтобы запустить этот образец, убедитесь, сначала добавили ссылку на пакет Microsoft.AspNetCore.DataProtection.Extensions.

[!code-csharp[Main](limited-lifetime-payloads/samples/limitedlifetimepayloads.cs)]
