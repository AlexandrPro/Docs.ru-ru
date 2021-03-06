---
title: "Управление ключами"
author: rick-anderson
description: "В этом документе перечислены сведения о реализации ASP.NET Core данных защиты ключа API управления."
keywords: "Управление ключами ASP.NET Core, защиты данных"
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: fb9b807a-d143-4861-9ddb-005d8796afa3
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/implementation/key-management
ms.openlocfilehash: d9e38fd5c8de2b10ad24fe557aa6e3063e40236e
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/10/2017
---
# <a name="key-management"></a>Управление ключами

<a name="data-protection-implementation-key-management"></a>

Система защиты данных автоматически управляет временем существования главные ключи, используемые для установки и снятия защиты полезных данных. Каждый раздел может существовать в одном из четырех этапов:

* Создана - ключ существует в ключ обмена, но еще не активирована. Ключ не должны использоваться для новых операций защитить пока не истечет время, достаточное, что ключ имели возможность распространить на все компьютеры, использующие этот ключ обмена.

* Активный — ключ существует в ключ обмена и должен использоваться для всех новых операций защитить.

* Срок действия истек - ключ был выполнен существования естественным и больше не должен использоваться для новых операций защитить.

* Был отозван - ключ скомпрометирован и не должны использоваться для новых операций защитить.

Ключи созданный active и истекшим сроком действия может использовать для снятия защиты входящих полезных данных. Отозванные ключей по умолчанию не может использоваться для снятия защиты полезных данных, но разработчик приложения может [переопределить это поведение](../consumer-apis/dangerous-unprotect.md#data-protection-consumer-apis-dangerous-unprotect) при необходимости.

>[!WARNING]
> Разработчик может возникнуть желание удаления ключа из ключа обмена (например, при удалении соответствующего файла из файловой системы). На этом этапе окончательно получить все данные, защищенные с помощью ключа и не аварийный переопределение как отозванных ключи. Удаление ключа является действительно разрушительных поведение и в результате система защиты данных обеспечивает доступ без первого класса API для выполнения данной операции.

## <a name="default-key-selection"></a>Выбор ключа по умолчанию

Когда система защиты данных считывает кольцо ключ из резервного хранилища, он предпринимает попытки найти ключ «default» из ключ обмена. Ключ по умолчанию используется для новых операций защитить.

Общие эвристики является, система защиты данных выбирает ключ с самой последней даты активации в качестве ключа по умолчанию. (Нет небольшой настроечным параметром, чтобы предусмотреть наклона часов сервера на сервер). Если ключ просрочен или отозван, создания ключа и если приложение не отключил автоматическое, то будет создан новый ключ с немедленного активации на [ключа истечение срока действия и накат](xref:security/data-protection/implementation/key-management#data-protection-implementation-key-management-expiration) политики ниже.

По причине система защиты данных немедленно формирует новый ключ, а не возврата к другой ключ, создание нового ключа должно рассматриваться как неявные истечение срока действия всех ключей, которые были активированы до нового ключа. Общая идея заключается в новые ключи были настроены с различных алгоритмов или механизмы шифрования статических более старые ключи, что система должна предпочтение возврата текущей конфигурации.

Существует одно исключение. Если разработчик приложения должен [отключено автоматическое создание ключа](xref:security/data-protection/configuration/overview#disableautomatickeygeneration), то система защиты данных необходимо выбрать что-либо ключ по умолчанию. В этом случае резервная системой будет выбран ключ не отозван с самая поздняя дата активации с предпочтением в ключи, для которых времени для распространения на других компьютерах в кластере. Система резервного использования может оказаться в результате выбора ключ с истекшим сроком действия по умолчанию. Система резервного использования никогда не будет выбирать отозванных ключ как ключ по умолчанию, и если кольцо ключ является пустым или каждый ключ был отозван система будет создавать ошибку при инициализации.

<a name="data-protection-implementation-key-management-expiration"></a>

## <a name="key-expiration-and-rolling"></a>Срок действия ключа и накат

Если создать ключ, он автоматически присваивается Дата активации {now + 2 дня} и дату окончания срока действия {теперь + 90 дней}. Задержка 2 дня до активации дает ключевое время для распространения через систему. Т. е он позволяет другим приложениям, навести указатель мыши на резервное хранилище позволяет продемонстрировать ключ их следующего автоматического обновления период, таким образом хотите максимально повысить вероятность того, что при ключ кольцо может стать активно, его завершения распространения для всех приложений, которые может потребоваться использовать его.

Если ключ по умолчанию истекает в течение 2 дней и кольцо ключ еще не содержит ключ, который будет активна по истечении срока действия ключа по умолчанию, система защиты данных автоматически будет сохранен новый ключ для ключей. Этот новый ключ имеет Дата активации {ключ по умолчанию Дата истечения срока действия} и дату окончания срока действия {теперь + 90 дней}. Это позволяет системе автоматически развертывания ключей на регулярной основе без прерывания работы службы.

Могут возникнуть обстоятельства с активацией немедленно, в котором будет создана ключа. Примером может служить, когда приложение не выполнялось в течение времени и завершаются все ключи в ключ обмена. В этом случае ключ предоставляется Дата активации {сейчас} без задержки активации обычный 2 дня.

Время жизни ключа по умолчанию составляет 90 дней, хотя это можно настроить, как показано в следующем примере.

```csharp
services.AddDataProtection()
       // use 14-day lifetime instead of 90-day lifetime
       .SetDefaultKeyLifetime(TimeSpan.FromDays(14));
```

Администратор также может изменять всей системы по умолчанию, если явный вызов `SetDefaultKeyLifetime` переопределяет любую политику во всей системе. По умолчанию срок жизни ключа не может быть менее 7 дней.

## <a name="automatic-key-ring-refresh"></a>Обновление автоматического ключей

При инициализации системы защиты данных считывает кольцо ключ из базового хранилища и кэширует его в памяти. Этот кэш позволяет защитить и Unprotect операций для продолжения работы без прерывания в резервное хранилище. Система автоматически проверит резервного хранилища для изменения примерно каждые 24 часа или по истечении срока действия текущего ключа по умолчанию наступит раньше.

>[!WARNING]
> Разработчикам следует очень редко (почти никогда) должны использовать ключа API управления напрямую. Система защиты данных выполнит автоматическое управление ключами, как описано выше.

Система защиты данных предоставляет интерфейс `IKeyManager` может использоваться для проверки и внести изменения в кольцевой ключ. Система DI, предоставленный экземпляр `IDataProtectionProvider` также можно указать экземпляр `IKeyManager` для потребления. Кроме того, вы можете извлечь `IKeyManager` прямо из `IServiceProvider` как в примере ниже.

Любая операция, изменяющего ключей (Создание нового ключа явным образом или выполнение отзыв) станут недействительными кэш в памяти. При следующем вызове `Protect` или `Unprotect` система защиты данных, повторное считывание ключей и повторное создание кэша.

В следующем примере показано использование `IKeyManager` интерфейса для проверки и обработки ключей, включая отзыв существующие ключи и создание нового ключа вручную.

[!code-csharp[Main](key-management/samples/key-management.cs)]

## <a name="key-storage"></a>Хранение ключей

Система защиты данных имеет эвристики, при котором он пытается автоматически определить расположение соответствующего хранилища ключей и шифрование на механизм rest. Это также можно настроить разработчиком приложения. Следующие документы рассматриваются в реализации этих механизмов:

* [Поле поставщиков хранилища ключей](key-storage-providers.md#data-protection-implementation-key-storage-providers)

* [В поле ключа шифрования поставщиков rest](key-encryption-at-rest.md#data-protection-implementation-key-encryption-at-rest-providers)
