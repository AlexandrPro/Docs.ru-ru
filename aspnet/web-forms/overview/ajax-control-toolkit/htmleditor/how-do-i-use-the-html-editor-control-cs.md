---
uid: web-forms/overview/ajax-control-toolkit/htmleditor/how-do-i-use-the-html-editor-control-cs
title: "Использование управления редактора HTML (C#) | Документы Microsoft"
author: microsoft
description: "HTMLEditor является элементом управления ASP.NET AJAX, который позволяет легко создавать и изменять данные HTML, с помощью кнопок на панели инструментов."
ms.author: aspnetcontent
manager: wpickett
ms.date: 05/12/2009
ms.topic: article
ms.assetid: f47e6224-c2e5-4472-b069-b6c7b6115200
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/ajax-control-toolkit/htmleditor/how-do-i-use-the-html-editor-control-cs
msc.type: authoredcontent
ms.openlocfilehash: 17343660d7bf7aa6210fa9c6c9c0206598d34b18
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/10/2017
---
<a name="how-do-i-use-the-html-editor-control-c"></a>Использование управления редактора HTML (C#)
====================
по [Microsoft](https://github.com/microsoft)

> HTMLEditor является элементом управления ASP.NET AJAX, который позволяет легко создавать и изменять данные HTML, с помощью кнопок на панели инструментов.


Целью данного учебника является предоставляются общие сведения о управления редактора HTML, входящий в состав набора элементов управления AJAX. Редактор HTML включает параметры для изменения размера шрифта, выбрав шрифт, изменение цвета фона, изменяя цвет переднего плана, добавив ссылки, добавив изображения, изменение выравнивания текста и выполнение вырезания, копирования и вставки операции (см. рис. 1).


[![Редактор HTML](how-do-i-use-the-html-editor-control-cs/_static/image1.jpg)](how-do-i-use-the-html-editor-control-cs/_static/image1.png)

**На рисунке 01**: В редакторе HTML ([Просмотр полноразмерное изображение](how-do-i-use-the-html-editor-control-cs/_static/image2.png))


Редактор HTML позволяет ввести содержимое с помощью режим конструктора или HTML можно ввести непосредственно. Можно также настроить параметр, чтобы просмотреть содержимое HTML (см. рис. 2).


[![Проектирование, HTML и предварительного просмотра кнопок](how-do-i-use-the-html-editor-control-cs/_static/image2.jpg)](how-do-i-use-the-html-editor-control-cs/_static/image3.png)

**На рисунке 02**: кнопки конструктора, HTML и предварительного просмотра ([Просмотр полноразмерное изображение](how-do-i-use-the-html-editor-control-cs/_static/image4.png))


В этом учебнике вы узнаете, как отобразить редактор HTML, Настройка кнопки панели инструментов, отображаются в редакторе HTML и как избежать межсайтовых сценариев атак.

## <a name="displaying-the-html-editor"></a>Отображение редактора HTML

Прежде чем использовать редактор HTML на странице ASP.NET, необходимо сначала добавить элемент управления ScriptManager на страницу. Элемент управления ScriptManager расположен под вкладки расширения AJAX в области элементов Visual Studio или Visual Web Developer Express.

Рекомендуется поместить элемент управления ScriptManager в верхней части страницы, перед другими элементами управления на странице. Например, можно поместить его сразу под открывающей серверную &lt;формы&gt; тег.

В области элементов с остальной частью панели элементов управления AJAX расположен элемент управления HTML-редактора. Он называется управления редактора (см. рис. 3).


[![Элемент управления редактора HTML](how-do-i-use-the-html-editor-control-cs/_static/image3.jpg)](how-do-i-use-the-html-editor-control-cs/_static/image5.png)

**На рисунке 03**: элемент управления в редакторе HTML ([Просмотр полноразмерное изображение](how-do-i-use-the-html-editor-control-cs/_static/image6.png))


После перетаскивания редактора HTML на странице, можно установить его свойства в окне свойств. Например обычно необходимо задать свойства ширины и высоты. Листинг 1 содержит источник для страницы ASP.NET, которая содержит HTML-редактора.

**Листинг 1 - SimpleEditor.aspx**

[!code-aspx[Main](how-do-i-use-the-html-editor-control-cs/samples/sample1.aspx)]

Страницы в список 1 содержит элемент управления редактора HTML, элемент управления Button и текстовым элементом управления. При нажатии кнопки, содержимое редактора HTML отображается в управлении литералов (см. рис. 4).


[![Отправка формы с помощью HTML-редактора](how-do-i-use-the-html-editor-control-cs/_static/image4.jpg)](how-do-i-use-the-html-editor-control-cs/_static/image7.png)

**На рисунке 04**: Отправка формы с редактором HTML ([Просмотр полноразмерное изображение](how-do-i-use-the-html-editor-control-cs/_static/image8.png))


Свойство содержимого HTML-редактор используется для получения HTML-содержимое, введенное в редакторе HTML. Имейте в виду, что это HTML-содержимое может содержать JavaScript. В следующем разделе мы рассмотрим, каким образом можно предотвратить атаки путем внедрения кода JavaScript.

## <a name="customizing-the-html-editor-toolbar"></a>Настройка панели инструментов редактора HTML

Можно настроить только кнопки, которые отображаются в редакторе. Например может потребоваться удалить вкладки «HTML», чтобы запретить пользователям Переключение редактора HTML в режиме HTML. Или, может потребоваться удалить раскрывающегося списка размер шрифта, чтобы запретить пользователям создавать чрезмерно большим текст на форуме сообщения post (см. рис. 5).


[![Настраиваемый редактор HTML](how-do-i-use-the-html-editor-control-cs/_static/image5.jpg)](how-do-i-use-the-html-editor-control-cs/_static/image9.png)

**На рисунке 05**: A настроить редактор HTML ([Просмотр полноразмерное изображение](how-do-i-use-the-html-editor-control-cs/_static/image10.png))


Настройка кнопок панели инструментов путем наследования от базового класса редактора новый редактор HTML. Например пользовательский редактор в списке 2 содержит только кнопки панели инструментов для полужирным шрифтом или курсивом. Будут удалены все кнопки панели инструментов. Кроме того был удален из нижней части редактора HTML-вкладка (но вкладки конструктора и предварительного просмотра по-прежнему существуют).

**2 — приложение вывод\_Code\CustomEditor.cs**

[!code-csharp[Main](how-do-i-use-the-html-editor-control-cs/samples/sample2.cs)]

В приложение, необходимо добавить класс в списке 2\_кода папки, чтобы класс компилируются автоматически. Если приложение\_веб-сайта не существует папки с кодом, то можно просто добавить папку.

После создания специализированного редактора добавлением его на страницу ASP.NET так же при добавлении обычный HTML-редактор (см. список 3).

**Листинг 3 - ShowCustomEditor.aspx**

[!code-aspx[Main](how-do-i-use-the-html-editor-control-cs/samples/sample3.aspx)]

## <a name="avoiding-cross-site-scripting-xss-attacks"></a>Избегайте использования атаки межузловых сценариев (XSS)

Каждый раз, когда принимать входные данные от пользователя и отобразите входа на веб-сайте, вы потенциально открыть веб-сайта к атакам межсайтовых сценариев (XSS). В теории злоумышленнику удалось отправить код JavaScript, выполняемый, когда отобразится входных данных. JavaScript может использоваться для кражи паролей пользователей или другую конфиденциальную информацию.

Как правило можно отказаться от атак с XSS с HTML-кодирование любые входные данные, получить от пользователя перед его отображением на веб-странице. Тем не менее, HTML-кодирование вывода редактор HTML будет не только кодирования &lt;сценарий&gt; тегов, он будет также закодировать все теги HTML. Другими словами будут потеряны все форматирование, такие как тип шрифта, размер шрифта и цвет фона.

Если выполняется сбор конфиденциальных сведений от пользователей — например, пароли, номера кредитных карт и номера социального страхования - следует не отображаются без кодировки содержимого, получаемые от пользователя с помощью редактора HTML. Редактор HTML следует использовать только в ситуациях, в которых не являются повторное отображение HTML-содержимого или HTML-содержимого передается на веб-сайт надежной стороной.

Допустим, что вы создаете приложение блога. В этом случае имеет смысл использовать редактор HTML при создании сообщения в блогах. Вы являетесь единственным, кто отправляет сообщение в блоге и, скорее всего, можно доверять самостоятельно не для отправки вредоносных JavaScript. Тем не менее он не имеет смысла при разрешении анонимным пользователям отправлять комментарии с помощью редактора HTML. Вам следует особенно внимательно в ситуациях, в которых пользователи отправляют конфиденциальные сведения, например пароли. Возможно пользователь-злоумышленник может post комментарий, который содержит правой JavaScript для перехвата пароль.

## <a name="summary"></a>Сводка

В этом учебнике вам были предоставлены краткий обзор элемента управления редактора HTML, включенных в набор элементов управления AJAX. Вы узнали, как использовать редактор HTML для принятия более информативного содержимого от пользователя и отправить содержимое на сервер. Также мы рассмотрели, как можно настроить кнопки панели инструментов, которые отображаются в редакторе HTML. Наконец вы узнали, как во избежание межсайтовых сценариев атак при использовании редактора HTML для приема потенциально опасные входные данные.

>[!div class="step-by-step"]
[Вперед](how-do-i-use-the-html-editor-control-vb.md)
