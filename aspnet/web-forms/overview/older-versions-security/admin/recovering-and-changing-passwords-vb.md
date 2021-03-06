---
uid: web-forms/overview/older-versions-security/admin/recovering-and-changing-passwords-vb
title: "Восстановление и изменение паролей (VB) | Документы Microsoft"
author: rick-anderson
description: "ASP.NET включает в себя два веб-элементы управления для соблюдения восстановление и изменение паролей. Элемент управления PasswordRecovery позволяет восстановить его потеряны pa посетитель..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 04/01/2008
ms.topic: article
ms.assetid: f9adcb5d-6d70-4885-a3bf-ed95efb4da1a
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/older-versions-security/admin/recovering-and-changing-passwords-vb
msc.type: authoredcontent
ms.openlocfilehash: f7f6e7e4bc3a8cc7e70911bc22a28d385f762af0
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/10/2017
---
<a name="recovering-and-changing-passwords-vb"></a>Восстановление и изменение паролей (Visual Basic)
====================
по [Скотт Митчелл](https://twitter.com/ScottOnWriting)

[Загрузить код](http://download.microsoft.com/download/6/0/e/60e1bd94-e5f9-4d5a-a079-f23c98f4f67d/VB.13.zip) или [скачать PDF](http://download.microsoft.com/download/6/0/e/60e1bd94-e5f9-4d5a-a079-f23c98f4f67d/aspnet_tutorial13_ChangingPasswords_vb.pdf)

> ASP.NET включает в себя два веб-элементы управления для соблюдения восстановление и изменение паролей. Элемент управления PasswordRecovery позволяет посетитель для восстановления пароля потеряны. Элемент управления ChangePassword позволяет пользователю изменить свой пароль. Как и другие связанные с именем входа веб-элементов управления мы видели на протяжении этого учебника ряда PasswordRecovery и ChangePassword элементы управления работают с платформой членство в фоновом режиме для сброса или изменения паролей пользователей.


## <a name="introduction"></a>Вступление

Между веб-сайты для моей банка, программа компании, телефона компании, учетные записи электронной почты и настраиваемый веб-порталы I, как большинство пользователей имеют множество различных паролей. С учетными данными так много в запоминании эти дни не люди забудете свой пароль. С учетом этого веб-сайтов, которые содержат учетные записи пользователей должны включать способом для пользователя восстановить свой пароль. Этот процесс обычно включает создание случайного новый пароль и отправка по электронной почте на адрес электронной почты пользователя для файла. После получения нового пароля большинству пользователей вернуться на сайт и изменить свой пароль от того, случайно запоминающийся ему.

ASP.NET включает в себя два веб-элементы управления для соблюдения восстановление и изменение паролей. Элемент управления PasswordRecovery позволяет посетитель для восстановления пароля потеряны. Элемент управления ChangePassword позволяет пользователю изменить свой пароль. Как и другие связанные с именем входа веб-элементов управления мы видели на протяжении этого учебника ряда PasswordRecovery и ChangePassword элементы управления работают с платформой членство в фоновом режиме для сброса или изменения паролей пользователей.

В этом учебнике мы рассмотрим использование этих двух элементов управления. Также будет показано, как программным образом измените и сбросьте пароль пользователя через `MembershipUser` класса `ChangePassword` и `ResetPassword` методы.

## <a name="step-1-helping-users-recover-lost-passwords"></a>Шаг 1: Помочь пользователям восстановить потери пароли

Все веб-сайтов, поддерживающих учетные записи пользователей должны предоставлять пользователям возможность какого-либо механизма, при восстановлении забытых паролей. Хорошо то, что реализация такие функциональные возможности в ASP.NET выполняется очень просто благодаря PasswordRecovery веб-управления. Элемент управления PasswordRecovery предоставляет интерфейс, который запрашивает имя пользователя и при необходимости, ответ на вопрос их безопасности. Он затем отправляется сообщение электронной почты пользователя пароль.

> [!NOTE]
> Так как сообщения электронной почты передаются по сети в виде обычного текста участвуют угрозы безопасности при отправке пароля пользователя по электронной почте.


Элемент управления PasswordRecovery состоит из трех представлений.

- **Имя пользователя** -предлагает посетителя свое имя пользователя. Это начальное представление.
- **Вопрос**-отображает вопрос пользователя имя пользователя и безопасности в виде текста, и текстовое поле для пользователя, введите ответ на свой вопрос безопасности.
- **Успех**-отображает сообщение, сообщая пользователю, что отправлена свой пароль.

Отображаются представления и действия, выполняемые с помощью элемента управления PasswordRecovery зависят следующие параметры конфигурации членства:

- `RequiresQuestionAndAnswer`
- `EnablePasswordRetrieval`
- `EnablePasswordReset`

Платформа членства `RequiresQuestionAndAnswer` указывает, является ли пользователи должны указать секретный вопрос и ответ, при регистрации учетной записи. Как уже говорилось в <a id="_msoanchor_1"> </a> [ *Создание учетных записей пользователей* ](../membership/creating-user-accounts-vb.md) учебника, если `RequiresQuestionAndAnswer` имеет значение True (по умолчанию), то интерфейс CreateUserWizard включает текстовое поле элементы управления для нового пользователя защитный вопрос и ответ; Если `RequiresQuestionAndAnswer` имеет значение False, собранные сведения отсутствуют. Аналогично Если `RequiresQuestionAndAnswer` имеет значение True, то элемент управления отображает PasswordRecovery, просмотреть вопрос после пользователь ввел свое имя пользователя, пароль будет восстановлена только в том случае, если пользователь введет правильный защитный ответ. Если `RequiresQuestionAndAnswer` имеет значение False, однако управления PasswordRecovery перемещает непосредственно из представления имени пользователя для представления успешного выполнения.

После пользователь предоставил свое имя - или его имя пользователя и безопасность ответов, если `RequiresQuestionAndAnswer` имеет значение True - PasswordRecovery отправляется сообщение электронной почты пользователя свой пароль. Если `EnablePasswordRetrieval` параметр имеет значение True, то пользователю по электронной почте их текущий пароль. Если задано значение False и `EnablePasswordReset` имеет значение True, то управления PasswordRecovery приводит к возникновению случайных новый пароль для пользователя и отправляется сообщение электронной почты в этот новый пароль для их. Если оба `EnablePasswordRetrieval` и `EnablePasswordReset` имеют значение False, элемент управления PasswordRecovery выбрасывает исключение.

> [!NOTE]
> Помните, что `SqlMembershipProvider` хранит пароли пользователей в одном из трех форматов: Clear, Hashed (по умолчанию) или шифрование. Использовать механизм хранения зависит от параметров конфигурации членства; демонстрационное приложение в формате хэшированный пароль. При использовании формата хэшированный пароль `EnablePasswordRetrieval` параметр необходимо задать значение False, так как система не может определить фактический пароль из хэшированное версии, хранящейся в базе данных.


Рис. 1 показано, как интерфейс и поведение PasswordRecovery зависит от конфигурации членства.


[![RequiresQuestionAndAnswer, EnablePasswordRetrieval и EnablePasswordReset влияют на внешний вид и поведение элемента управления PasswordRecovery](recovering-and-changing-passwords-vb/_static/image2.png)](recovering-and-changing-passwords-vb/_static/image1.png)

**Рис. 1**: `RequiresQuestionAndAnswer`, `EnablePasswordRetrieval`, и `EnablePasswordReset` влияют на внешний вид и поведение элемента управления PasswordRecovery ([Просмотр полноразмерное изображение](recovering-and-changing-passwords-vb/_static/image3.png))


> [!NOTE]
> В <a id="_msoanchor_2"> </a> [ *создания схемы членства в SQL Server* ](../membership/creating-the-membership-schema-in-sql-server-vb.md) учебник был настроен поставщик членства, задав `RequiresQuestionAndAnswer` значение True, `EnablePasswordRetrieval` для Значение равно false, и `EnablePasswordReset` значение True.


### <a name="using-the-passwordrecovery-control"></a>С помощью элемента управления PasswordRecovery

Давайте взглянем на использование PasswordRecovery элемента управления на странице ASP.NET. Откройте `RecoverPassword.aspx` и перетащите и удалить из области элементов в конструктор элемента управления PasswordRecovery; установить его `ID` для `RecoverPwd`. Как и элементы управления входа и веб-CreateUserWizard PasswordRecovery элемента управления представления отображают многофункциональных составных интерфейс, который содержит метки, текстовые поля, кнопки и проверяющие элементы управления. Можно настроить внешний вид представления через свойства стиля элемента управления или путем преобразования в шаблонов представлений. Я это поле оставить в качестве упражнения для интересующихся.

При посещении этой страницы она введите ее имя пользователя и нажмите кнопку "Отправить". Так как мы задали `RequiresQuestionAndAnswer` в значение True, если в параметрах конфигурации нашей членства PasswordRecovery будет управлять, а затем отображать представления вопроса. После пользователь вводит свой правильный ответ на контрольный вопрос и нажимает кнопку Submit, управления PasswordRecovery обновить пароль пользователя с предложением формируется случайным образом и адрес электронной почты на адрес электронной почты в файле этот пароль. Все это можно было без нам писать одна строка кода!

Перед запуском этой странице имеется Заключительная часть конфигурации, как правило: необходимо указать параметры доставки почты в `Web.config`. Управления PasswordRecovery использует эти параметры для отправки сообщения электронной почты.

Конфигурация доставки почты определяется через [ `<system.net>` элемент](https://msdn.microsoft.com/en-us/library/6484zdc1.aspx) [ `<mailSettings>` элемент](https://msdn.microsoft.com/en-us/library/w355a94k.aspx). Используйте [ `<smtp>` элемент](https://msdn.microsoft.com/en-us/library/ms164240.aspx) для указания метода доставки и адрес отправителя по умолчанию. Приведенный ниже код настраивает параметры почты для использования сетевой SMTP сервера с именем `smtp.example.com` через порт 25 и имя пользователя и пароль учетных данных имени пользователя и пароля.

> [!NOTE]
> `<system.net>`является дочерним элементом корневого `<configuration>` элемент и одноуровневым `<system.web>`. Таким образом, не следует помещать `<system.net>` элемент в пределах `<system.web>` элемента; вместо этого поместите его на том же уровне.


[!code-xml[Main](recovering-and-changing-passwords-vb/samples/sample1.xml)]

Помимо использования SMTP-сервер в сети, можно также указать каталог подбора сообщений, где следует внесен отправку сообщений электронной почты.

После настройки параметров SMTP посетите `RecoverPassword.aspx` страницу в обозревателе. Сначала попробуйте ввести имя пользователя, который не существует в хранилище пользователя. Как показано на рисунке 2, элемент управления PasswordRecovery отображает сообщение о том, что сведения о пользователе будет недоступен. Текст сообщения можно настроить с помощью элемента управления [ `UserNameFailureText` свойства](https://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.passwordrecovery.usernamefailuretext.aspx).


[![Сообщение об ошибке отображается, если введено неправильное имя пользователя](recovering-and-changing-passwords-vb/_static/image5.png)](recovering-and-changing-passwords-vb/_static/image4.png)

**На рисунке 2**: сообщение об ошибке отображается, если введено неправильное имя пользователя ([Просмотр полноразмерное изображение](recovering-and-changing-passwords-vb/_static/image6.png))


Теперь можно ввести имя пользователя. Используйте имя пользователя учетной записи в системе с адресом электронной почты, можно открыть и которого безопасности ответить вам известно. После ввода имени пользователя и нажмите кнопку Отправить, элемент управления PasswordRecovery отображает его представления вопроса. Как с представления имени пользователя при вводе неправильного ответа на PasswordRecovery отображает элемент управления, сообщения об ошибке (см. рис. 3). Используйте [ `QuestionFailureText` свойство](https://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.passwordrecovery.questionfailuretext.aspx) для настройки это сообщение об ошибке.


[![Сообщение об ошибке отображается, если пользователь ввел недопустимый ответ на контрольный вопрос](recovering-and-changing-passwords-vb/_static/image8.png)](recovering-and-changing-passwords-vb/_static/image7.png)

**Рис. 3**: сообщение об ошибке отображается, если пользователь ввел недопустимый ответ на контрольный вопрос ([Просмотр полноразмерное изображение](recovering-and-changing-passwords-vb/_static/image9.png))


Наконец введите правильный защитный ответ и нажмите кнопку Отправить. В фоновом управления PasswordRecovery создает случайный пароль, присваивает его учетной записи пользователя, отправляет сообщение электронной почты, информирующее пользователя нового пароля (см. рис. 4), а затем отображает представления успешного выполнения.


[![Пользователю отправляется сообщение электронной почты с His новый пароль](recovering-and-changing-passwords-vb/_static/image11.png)](recovering-and-changing-passwords-vb/_static/image10.png)

**Рис. 4**: пользователю отправляется сообщение электронной почты с His нового пароля ([Просмотр полноразмерное изображение](recovering-and-changing-passwords-vb/_static/image12.png))


### <a name="customizing-the-email"></a>Настройка электронной почты

По умолчанию сообщений электронной почты, отправляемых с помощью элемента управления PasswordRecovery является довольно скромным (см. рис. 4). Сообщение отправляется из учетной записи, указанной в `<smtp>` элемента `from` атрибута с темой пароль и тело обычного текста:

Вернитесь на сайт и войдите, используя следующие сведения.

Имя пользователя: *имя пользователя*

пароль: *пароль*

Это сообщение можно настроить программно через обработчик событий для элемента управления PasswordRecovery [ `SendingMail` событий](https://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.passwordrecovery.sendingmail.aspx), или декларативно с помощью [ `MailDefinition` свойства](https://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.passwordrecovery.maildefinition.aspx). Давайте рассмотрим оба варианта.

`SendingMail` Событие непосредственно перед сообщение электронной почты отправляется и наши последняя возможность программное изменение сообщения электронной почты. Когда происходит событие, обработчик событий передается объект типа [ `MailMessageEventArgs` ](https://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.mailmessageeventargs.aspx), чей `Message` свойство содержит ссылку на адреса электронной почты, который будет отправлен.

Создайте обработчик событий для `SendingMail` событий и добавьте следующий код, который программными средствами добавляет `webmaster@example.com` в список "Копия".

[!code-vb[Main](recovering-and-changing-passwords-vb/samples/sample2.vb)]

Сообщения электронной почты также можно настроить через декларативным образом. PasswordRecovery `MailDefinition` является объектом типа [ `MailDefinition` ](https://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.maildefinition.aspx). `MailDefinition` Класс предлагает целый ряд свойств, связанных с электронной почтой, включая `From`, `CC`, `Priority`, `Subject`, `IsBodyHtml`, `BodyFileName`и др. Во-первых, задайте [ `Subject` свойство](https://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.maildefinition.subject.aspx) на более описательное имя, отличной от используемых по умолчанию (пароль), такие как сброса пароля...

Чтобы настроить текст сообщения электронной почты, необходимо создать файл шаблона одно сообщение электронной почты, содержащее содержимое тела. Начните с создания новой папки в веб-сайт с именем `EmailTemplates`. Добавьте новый текстовый файл в эту папку с именем `PasswordRecovery.txt` и добавьте следующее содержимое:

[!code-aspx[Main](recovering-and-changing-passwords-vb/samples/sample3.aspx)]

Обратите внимание на использование заполнители `<%UserName%>` и `<%Password%>`. Управления PasswordRecovery автоматически заменяет эти заполнители два пользователя и пароль восстановленные перед отправкой сообщения электронной почты.

Наконец, точки `MailDefinition` [ `BodyFileName` свойство](https://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.maildefinition.bodyfilename.aspx) в только что созданный шаблон электронной почты (`~/EmailTemplates/PasswordRecovery.txt`).

После внесения этих изменений повторное использование `RecoverPassword.aspx` страницы и введите имя пользователя и безопасности ответ. Вы получаете следует сообщение электронной почты, который выглядит как показано на рис. 5. Обратите внимание, что `webmaster@example.com` была бы копия и что поле темы и текст были обновлены.


[![Тема, текст и список копий были обновлены](recovering-and-changing-passwords-vb/_static/image14.png)](recovering-and-changing-passwords-vb/_static/image13.png)

**Рис. 5**: Тема, текст и копия список был обновлен ([Просмотр полноразмерное изображение](recovering-and-changing-passwords-vb/_static/image15.png))


Чтобы отправить сообщение электронной почты в формате HTML задайте [ `IsBodyHtml` ](https://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.maildefinition.isbodyhtml.aspx) в значение True (по умолчанию: False) и обновление шаблона электронной почты для включения HTML.

`MailDefinition` Свойство не является уникальным для PasswordRecovery класса. Как будет показано в шаге 2, управления ChangePassword также предлагает `MailDefinition` свойство. Кроме того, элемент управления CreateUserWizard включает такие свойства, которые можно настроить для автоматической отправки сообщений приветственное сообщение электронной почты новым пользователям.

> [!NOTE]
> В настоящее время нет ссылок на левой панели навигации для достижения `RecoverPassword.aspx` страницы. Только пользователь будут заинтересованы в посещении этой страницы, если она не смогла успешно Войдите на сайт. Поэтому обновлять `Login.aspx` страницу, чтобы включить ссылку на `RecoverPassword.aspx` страницы.


### <a name="programmatically-resetting-a-users-password"></a>Программным образом сбросе пароля пользователя

Если сброс пароля пользователя PasswordRecovery управление вызовы `MembershipUser` объекта [ `ResetPassword` метод](https://msdn.microsoft.com/en-us/library/system.web.security.membershipuser.resetpassword.aspx). Этот метод имеет две перегрузки:

- **[`ResetPassword`](https://msdn.microsoft.com/en-us/library/d94bdzz2.aspx)**-Сбрасывает пароль пользователя. Используйте этот перегруженный метод, если `RequiresQuestionAndAnswer` имеет значение False.
- **[`ResetPassword(securityAnswer)`](https://msdn.microsoft.com/en-us/library/d90zte4w.aspx)**-Сбрасывает только если пароль пользователя, предоставленного *securityAnswer* указано правильно. Используйте этот перегруженный метод, если `RequiresQuestionAndAnswer` имеет значение True.

Обе перегрузки возвращают новый созданный случайным образом пароль.

Как и с другими методами в платформе членства `ResetPassword` метод делегирует настроенного поставщика. `SqlMembershipProvider` Вызывает `aspnet_Membership_ResetPassword` передача хранимой процедуры в имя пользователя, пароль и ответов введенного пароля, в других полях. Хранимая процедура гарантирует, что соответствует ответ пароля, а затем обновляет пароль пользователя.

Пара замечаний Низкоуровневую реализацию:

- Заблокированной пользователя не удается сбросить свой пароль. Тем не менее могут неутвержденных пользователя. Мы обсудим заблокированные и утверждена состояний более подробно в <a id="_msoanchor_3"> </a> [ *Unlocking и утверждение пользователя* ](unlocking-and-approving-user-accounts-vb.md) учебника учетных записей.
- Если ответ пароль неверен, увеличивается число попыток ответов неудачных пароль пользователя. Если указанное число попыток недействительный безопасности в течение указанного интервала времени, когда пользователь заблокирован.

### <a name="a-word-on-how-the-random-passwords-are-generated"></a>Создаются слово на как случайными паролями

Пароли формируется случайным образом, показанный в сообщениях электронной почты в иллюстрациях 4 и 5 создаются с помощью класса членства [ `GeneratePassword` метод](https://msdn.microsoft.com/en-us/library/system.web.security.membership.generatepassword.aspx). Этот метод принимает два входных параметров целое - *длина* и *numberOfNonAlphanumericCharacters* — и возвращает строку, по крайней мере *длина* символов цифр по бы *numberOfNonAlphanumericCharacters* число не буквенно-цифровых символов. При вызове этого метода в классы членства или связанные с именем входа веб-элементов управления из значения для этих двух параметров определяются конфигурации членства `MinRequiredPasswordLength` и `MinRequiredNonalphanumericCharacters` свойства, которые рекомендуется задать значение 7 и 1, соответственно.

`GeneratePassword` Метод использует криптостойкой генератора случайных чисел для убедитесь, что в выбраны какие случайных символов не учитывается. Кроме того `GeneratePassword` — `Public`, это означает, что можно использовать его непосредственно из приложения ASP.NET для формирования случайных строк или пароли.

> [!NOTE]
> `SqlMembershipProvider` Класс всегда создает случайный пароль менее 14 символов, поэтому, если `MinRequiredPasswordLength` имеет меньше 14, то его значение игнорируется.


## <a name="step-2-changing-passwords"></a>Шаг 2: Изменение паролей

Пароль формируется случайным образом трудны для запоминания. Рассмотрим пароль, показанный на рисунке 4: `WWGUZv(f2yM:Bd`. Попробуйте фиксации, в памяти! Следует отметить после отправки пароль формируется случайным образом такого рода она потребуется изменить пароль на более запоминающееся.

Используйте элемент управления ChangePassword для создания интерфейса пользователя изменить свой пароль. Гораздо как управления PasswordRecovery ChangePassword управления состоит из двух представлений: изменение пароля и успех. Смена пароля представление пользователю старый и новый пароли. После указания старый пароль и новый пароль, который соответствует требованиям к минимальной длине и не буквенно-цифровых символов, управления ChangePassword обновляет пароль пользователя и отображает представления успешного выполнения.

> [!NOTE]
> Элемент управления ChangePassword изменяет пароль пользователя путем вызова `MembershipUser` объекта [ `ChangePassword` метод](https://msdn.microsoft.com/en-us/library/system.web.security.membershipuser.changepassword.aspx). Метод ChangePassword принимает два `String` входных параметров - *Старый_пароль* и *newPassword*- и обновляет учетную запись пользователя с *newPassword*, при условии, что предоставленный *Старый_пароль* указано правильно.


Откройте `ChangePassword.aspx` и добавьте элемент управления ChangePassword страницу, присвоив ему имя `ChangePwd`. На этом этапе конструктора должны показывать Смена пароля представления (см. рис. 6). Как с помощью элемента управления PasswordRecovery можно переключаться между представлениями через смарт-тег элемента управления. Кроме того эти представления внешний вид настраиваются через свойства различные стиля или путем их преобразования в шаблон.


[![Добавить на страницу элемент управления ChangePassword](recovering-and-changing-passwords-vb/_static/image17.png)](recovering-and-changing-passwords-vb/_static/image16.png)

**Рис. 6**: добавить на страницу элемент управления ChangePassword ([Просмотр полноразмерное изображение](recovering-and-changing-passwords-vb/_static/image18.png))


Элемент управления ChangePassword можно обновить пароль текущего выполнившего вход пользователя *или* пароль другой, указанного пользователя. Как показано на рис. 6, представление сменить пароль по умолчанию отображает только три элемента управления TextBox: один для старого пароля и два для нового пароля. Этот интерфейс по умолчанию используется для обновления пароля пользователя, выполнившего вход.

Чтобы обновить пароль другого пользователя с помощью элемента управления ChangePassword, задать для элемента управления [ `DisplayUserName` свойства](https://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.changepassword.displayusername.aspx) значение True. Это добавляет четвертый текстовое поле на страницу, запрашивает имя пользователя, пароль которого для изменения.

Параметр `DisplayUserName` для является полезным, если вы хотите разрешить out вошедшего пользователя сменить пароль без необходимости входить значение True. Лично я считаю, что нет ничего плохого требуют пользователя для входа, прежде чем позволяя ему сменить пароль. Таким образом, оставьте `DisplayUserName` значение False (по умолчанию). При принятии, тем не менее, мы по существу за исключением выполнения анонимным пользователям получать доступ этой страницы. Обновить правила авторизации URL-адрес сайта таким образом, чтобы запретить анонимные пользователи из посетив `ChangePassword.aspx`. Если вам необходимо обновить память на синтаксис правила авторизации URL-адрес, обращаться к <a id="_msoanchor_4"> </a> [ *авторизации на основе пользователя* ](../membership/user-based-authorization-vb.md) учебника.

> [!NOTE]
> Может показаться, что `DisplayUserName` свойство полезно для дает администраторам возможность изменять пароли других пользователей. Тем не менее, даже если `DisplayUserName` имеет значение True, старый пароль необходимо известные и ввести. Мы рассмотрим способы дает администраторам возможность изменения паролей пользователей на шаге 3.


Посетите `ChangePassword.aspx` через браузер и изменить свой пароль. Обратите внимание, что сообщение об ошибке отображается, если ввести новый пароль, который не удается обеспечить соблюдение требований не буквенно-цифровых символов, указанный в конфигурации членства и длина пароля (см. рис. 7).


[![Добавить на страницу элемент управления ChangePassword](recovering-and-changing-passwords-vb/_static/image20.png)](recovering-and-changing-passwords-vb/_static/image19.png)

**Рис. 7**: добавить на страницу элемент управления ChangePassword ([Просмотр полноразмерное изображение](recovering-and-changing-passwords-vb/_static/image21.png))


После ввода правильный пароль старого и нового пароля, вошедшего в систему на пользователя изменить пароль и отображаются представления успешного выполнения.

### <a name="sending-a-confirmation-email"></a>Отправка подтверждения адреса электронной почты

По умолчанию ChangePassword управления не отправляет сообщение электронной почты для пользователя, пароль которого только что обновлено. Если вы хотите отправить сообщение электронной почты, просто настроить элемент управления `MailDefinition` свойство. Позволяет настроить элемент управления ChangePassword таким образом, чтобы пользователю отправляется сообщение электронной почты формате HTML, содержащий новый пароль.

Начните с создания в новом файле в `EmailTemplates` папку с именем `ChangePassword.htm`. Добавьте следующую разметку:

[!code-html[Main](recovering-and-changing-passwords-vb/samples/sample4.html)]

Переключите элемент управления ChangePassword `MailDefinition` свойства `BodyFileName`, `IsBodyHtml`, и `Subject` свойства ~ / EmailTemplates/ChangePassword.htm "," True "и" пароль был изменен! соответственно.

После внесения этих изменений, повторном посещении страницы и изменить пароль еще раз. На этот раз управления ChangePassword Отправка сообщения настроенный, в формате HTML адреса электронной почты пользователя на файл (см. рис. 8).


[![По электронной почте сообщение о том, что их пароль пользователя изменилось](recovering-and-changing-passwords-vb/_static/image23.png)](recovering-and-changing-passwords-vb/_static/image22.png)

**Рис. 8**: по электронной почте сообщение о том, что их пароль пользователя изменилась ([Просмотр полноразмерное изображение](recovering-and-changing-passwords-vb/_static/image24.png))


## <a name="step-3-allowing-administrators-to-change-users-passwords"></a>Шаг 3: Дает администраторам возможность изменения паролей пользователей

Общие функции в приложениях, поддерживающих учетные записи пользователей — это возможность пользователь с правами администратора изменить пароли других пользователей. Иногда эта функциональность необходима, поскольку в системе нет возможности, чтобы пользователи могли изменять свои пароли. В этом случае администратор может назначить их новый пароль будет единственным способом для пользователя восстановить забытый пароль. С элементами управления PasswordRecovery и изменение пароля тем не менее, пользователи с правами администратора требуется не заняты сами с изменение паролей пользователей, как пользователи способны делать это самостоятельно.

Но что делать, если ваш клиент настаивает, административные пользователи должны иметь возможность изменять пароли других пользователей? К сожалению Добавление этой функции может быть немного усилий. Чтобы изменить пароль пользователя, старый и новый пароль должен быть введен для `MembershipUser` объекта `ChangePassword` метода, но администратор не должны знать пароль, чтобы изменить его.

Одно решение — сбросить пароль пользователя, а затем измените его на новый пароль, с помощью следующего кода:

[!code-vb[Main](recovering-and-changing-passwords-vb/samples/sample5.vb)]

Этот код сначала получает сведения о *username*, который является имя пользователя, пароль которого администратор хочет изменить. Далее, `ResetPassword` вызова метода какие назначает и случайных новый пароль пользователя. Данный пароль формируется случайным образом возвращается с помощью метода, которая хранится в переменной `resetPwd`. Теперь, когда мы знаем, пароль пользователя, мы можем изменить через вызов `ChangePassword`.

Проблема заключается в том, что этот код работает, только если конфигурация системы членства настроена таким образом, что `RequiresQuestionAndAnswer` имеет значение False. Если `RequiresQuestionAndAnswer` имеет значение True, как в приложении, а затем `ResetPassword` метода должно быть передано защитный ответ, в противном случае будет вызвано исключение.

Если framework членства настроен требовать секретный вопрос и ответ на него еще ваш клиент настаивает администраторы смогут изменять пароли пользователей, у вас есть три варианта:

- Вызывать рук в воздухе и сообщить ваш клиент, что это лишь один интересный момент, не может быть выполнена.
- Задать `RequiresQuestionAndAnswer` значение False. Это приводит к менее безопасного приложения. Предположим, что пользователь гнусных получившим доступ в ящик входящих сообщений электронной почты другого пользователя. Возможно скомпрометированный пользователь покинул их поддержки, чтобы перейти на обед не заблокировать своей рабочей станции и может быть они доступ к электронной почте из открытых терминала не выйдите из системы. В любом случае можно посетить гнусных пользователя `RecoverPassword.aspx` страницы и введите имя пользователя. Система будет отправить по электронной почте восстановленный пароль не запрашиваются защитный ответ.
- Обход уровень абстракции, созданные в framework членства и работать непосредственно с базой данных SQL Server. Схема членства включает хранимую процедуру с именем `aspnet_Membership_SetPassword` , задает пароль пользователя и не требуется для выполнения своей задачи ответ на контрольный вопрос или старый пароль.

Ни один из этих параметров не особенно удобно, но как жизненного цикла разработчик иногда переход.

Я пошли дальше и реализации третий способ написания кода, который обходит `Membership` и `MembershipUser` классы и работает непосредственно с `SecurityTutorials` базы данных.

> [!NOTE]
> Работая непосредственно с базой данных, является shattered инкапсуляцией, предоставляет платформа членства. Это решение связывает нам `SqlMembershipProvider`, делая менее portable наш код. Более того этот код может не работать должным образом в будущих версиях ASP.NET при изменении схемы членства. Этот подход является временное решение и, как и большинство обойти это ограничение, не примером советы и рекомендации.


Этот код содержит некоторые все биты и довольно длинный. Таким образом не следует, чтобы не перегружать этого учебника с глубокого анализа. Если вы заинтересованы в дополнительной информацией, загрузить код для этого учебника и посещение `~/Administration/ManageUsers.aspx` страницы. Мы создали на этой странице <a id="_msoanchor_5"> </a> [предыдущего учебника](building-an-interface-to-select-one-user-account-from-many-vb.md), список всех пользователей. Было выполнено обновление GridView, чтобы включить ссылку на `UserInformation.aspx` страницы, передав имя выбранного пользователя через строку запроса. `UserInformation.aspx` Странице отображаются сведения о выбранный пользователь и текстовые поля для изменения пароля (см. рис. 9).

После ввода нового пароля, подтверждение его второго текстового поля и кнопку обновления пользователя, выполняется обратная и `aspnet_Membership_SetPassword` вызова хранимой процедуры, обновление пароля пользователя. Читателю эти средства чтения интересует эта функциональность для подробного знакомства с кодом и повторите расширение функциональных возможностей для включения отправки сообщения электронной почты для пользователя, пароль которого было изменено.


[![Администратор может изменить пароль пользователя](recovering-and-changing-passwords-vb/_static/image26.png)](recovering-and-changing-passwords-vb/_static/image25.png)

**Рис. 9**: администратор может изменить пароль ([Просмотр полноразмерное изображение](recovering-and-changing-passwords-vb/_static/image27.png))


> [!NOTE]
> `UserInformation.aspx` Страница в данный момент работает только если framework членства настроен для хранения паролей в формате Clear или Hashed. Несмотря на то, что вам будет предложено добавить эта функция не имеет кода для шифрования новый пароль. Ситуации, рекомендуется добавить необходимый код рекомендуется использовать декомпилятор как [Reflector](http://www.aisto.com/roeder/dotnet/) для изучите исходный код для методов в платформе .NET Framework; первоначальный анализ `SqlMembershipProvider` класса `ChangePassword` метод. Это методика, использованной для написания кода для создания хэш пароля.


## <a name="summary"></a>Сводка

ASP.NET предоставляет два элемента управления, чтобы помочь пользователям управлять свой пароль. Управления PasswordRecovery полезно для тех, кто забывшим свои пароли. В зависимости от конфигурации framework членства пользователя либо по электронной почте свой текущий пароль или новый пароль формируется случайным образом. Элемент управления ChangePassword позволяет пользователю изменить свой пароль.

Как и элементы управления входа и CreateUserWizard PasswordRecovery и ChangePassword элементы управления отображаются многофункциональный пользовательский интерфейс без написания ни строчки декларативная разметка или строку кода. Если пользовательский интерфейс по умолчанию не удовлетворяют вашим потребностям, можно настроить его через различные свойства стиля. Кроме того интерфейсы элементов управления может быть преобразован в шаблоны для еще более точно степень контроля. В фоновом эти элементы управления используют API членства, вызов `MembershipUser` объекта `ResetPassword` и `ChangePassword` методы.

Программирование довольны!

### <a name="further-reading"></a>Дополнительные сведения

Дополнительные сведения по темам, рассматриваемые в этом учебнике см. в следующих ресурсах:

- [Примеры использования элемента управления ChangePassword](https://quickstarts.asp.net/QuickStartv20/aspnet/doc/ctrlref/login/changepassword.aspx)
- [Примеры использования элемента управления PasswordRecovery](https://quickstarts.asp.net/QuickStartv20/aspnet/doc/ctrlref/login/passwordrecovery.aspx)
- [При отправке сообщения в ASP.NET](http://aspnet.4guysfromrolla.com/articles/072606-1.aspx)
- [`System.Net.Mail`Часто задаваемые вопросы](http://www.systemnetmail.com/)

### <a name="about-the-author"></a>Об авторе

Скотт Митчелл, автор нескольких ASP/ASP.NET и основатель 4GuysFromRolla.com, работает с веб-технологиями Майкрософт с 1998 года. Скотт — независимый консультант, trainer и записи. Его последняя книга —  *[диспетчерами учат самостоятельно ASP.NET 2.0 в течение 24 часов](https://www.amazon.com/exec/obidos/ASIN/0672327384/4guysfromrollaco)*. Скотт может быть достигнута по [ mitchell@4guysfromrolla.com ](mailto:mitchell@4guysfromrolla.com) или через его блог по [http://ScottOnWriting.NET](http://scottonwriting.net/).

### <a name="special-thanks-to"></a>Благодарности

Этот учебник ряд прошел проверку многие полезные рецензентов. Основными редакторами этого учебника включают Майкл Emmings и Suchi Банерджи. Объясняются моих последующих статей для MSDN? Если Да, напишите мне по[mitchell@4GuysFromRolla.com](mailto:mitchell@4GuysFromRolla.com)

>[!div class="step-by-step"]
[Назад](building-an-interface-to-select-one-user-account-from-many-vb.md)
[Вперед](unlocking-and-approving-user-accounts-vb.md)
