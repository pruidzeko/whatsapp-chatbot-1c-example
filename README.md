# Внешняя демо обработка с чат-ботом для WhatsApp  

## Для чего нужна обработка

Обработа - демо пример чат-бота для WhatsApp на базе 1С, который может быть взят за основу дял написания своих чат-ботов. Интеграция сделана с использованием http сервиса, предоставляемого [Green API](https://green-api.com/)

## Что потребуется, чтобы запустить бота

* Платформа 1С не ниже версии 8.3.10
* Два номера на разных телефонах с приложением WhatsApp, либо два номера на одном телефоне с двумя сим-картами.  Два номер нужны, чтобы отправлять сообщения как бы "самому себе". При этом первый номер будет тот, с которого  мы будем писать команды чат-боту (далее ``клиентский номер``), а второй номер - с которого будет отвечать сам бот (далее ``номер бота``). Отправлять с одного номера не получится.
* Аккаунт в сервисе Green API. Подойдет тариф Разработчика. Он бесплатный и неограниченный по времени.

## Как запустить бота по шагам:

1. Если у Вас телефон с двумя сим-картами, то устанавливаем на него два приложения - обычный [WhatsApp Messenger](https://play.google.com/store/apps/details?id=com.whatsapp) и [Whatsapp Business](https://play.google.com/store/apps/details?id=com.whatsapp.w4b).

2. Один номер телефона регистрируем в WhatsApp Messenger, а другой в  Whatsapp Business.

3. Скачиваем [обработку чат-бота](https://github.com/green-api/whatsapp-chatbot-1c-example/releases/download/1.0/GreenAPI_ChatBot.epf),

4. Открываем обработку в режиме 1С Предприятие, переходим на вкладку ``Настройки`` и нажимаем на ``Помощник подключения``. Далее следуем ме=нструкциям помощника. В помощнике будет предложено сканировать QR код. Сканируем его для``номера бота`` - того, с которого чат-бот будет писать на наш ``клиентский номер``

![`Интерфейс помощника`](media/HelperReg.png)

5. Переходим на вкладку ``Чат-бот`` и нажимаем кнопку ``Запустить бота``
6. Открываем WhatsApp, на котором зарегистрирован ``клиентский номер`` и пишем любое сообщение на ``номер бота``. Бот отвечает приветственным сообщением:

![`Чат-бот начало`](media/chatBotHello.png)

7. Бот запущен. Теперь мы можем с ним общаться. Например, если написать в ответ цифру 1, то получим список номенклатуры

![`Чат бот запрос`](media/chatBotAction.jpg)

## Какие сценарии поддерживает бот

Сообщения и ответы бота можно настроить в коде обработки под себя. С помощью бота можно запрашивать у клиента информацию по шагам, уточняя свой вопрос с каждым новым ответом клиента.

Также поддерживаются глбальные команды. Например, если написать боту текстом слово ``Выход``, то диалог сброситься и начнется с самого начала. Эту функцию можно использовать для реализации команд типа ``Отписаться``, ``Стоп`` и т.п.

## Как настроить свои сценарии

1. Открываем обработку в режиме конфигуратора
2. В модуле объекта смотрим на функцию ``ПодготовитьШаблоныСообщений()``. Эта функция содержит все сообщения, которые пишет сам бот в ответ на сообщения пользователя
3. Чтобы добавить новый ответ внутри этой функции вызываем метод ``НовыйОтветБота`` по аналогии как это сделано в демо-примере: 

``` bsl
Функция ПодготовитьШаблоныСообщений(КоллекцияШаблонов, КоллекцияГлобальныхШаблонов)

    ОтветПоНоменклатуре = НовыйОтветБота(НовоеСообщение, "1", "Выводим 
        список товаров...", "Подключаемый_ВывестиНоменклатуру");

КонецФункции

```
Если нужно, то прописываем обработчик, который срабатывает после того, как клиент ответил боту верной командой. Обработчик должен быть в модуле объекта и иметь два входных параметра, например как в демо:

```bsl
Процедура Подключаемый_ВывестиНоменклатуру(Чат, Шаблон)

    ОтправитьСообщениеВЧат(Чат.ИдЧата, 
    "Арт: 123456, Чайник Электролюкс; цена: 1 200 руб; на складе: 12 шт,
    |Арт: 12223, Холодильник Хайер; цена: 45 000 руб; на складе: 4 шт
    |Арт: 54656, Телевизор Самсунг; цена: 110 000 руб; на складе: 2 шт");

КонецПроцедуры

```

Чтобы сделать вложенный ответ используем тот же метод ``НовыйОтветБота()``, но только в качестве первого параметра передаем результат ``родительского`` метода, например как в демо:

```bsl
// Родитеский ответ
ОтветПоНоменклатуре = НовыйОтветБота(НовоеСообщение, "1", "Выводим 
    список товаров...", "Подключаемый_ВывестиНоменклатуру");

// Вложенный ответ
ОтветМенеджер = НовыйОтветБота(ОтветПоНоменклатуре,, "Хотите связаться с менеджером (Да/Нет)?");

```

Чтобы прописать глобальную команду, используем метод ``НовыйГлобальныйОтветБота()`` и первым параметром в него передаем  переменную ``КоллекцияГлобальныхШаблонов``. Пример:

```bsl
Функция ПодготовитьШаблоныСообщений(КоллекцияШаблонов, КоллекцияГлобальныхШаблонов)

    НовыйГлобальныйОтветБота(КоллекцияГлобальныхШаблонов, "Выход", "Работа с    чат ботом завершается...", "Подключаемый_ВыключитьЧатБот");

КонецФункции
```
