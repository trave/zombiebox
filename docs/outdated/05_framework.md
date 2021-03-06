Фреймворк
=========
Архитектура приложения
----------------------
### Приложение (Application)

Глобальный объект приложения, который инстанцируется в точке входа и
доступен из глобальной переменной app. Наследуясь от сгенерированного
boilerplate-класса, он реализует определение платформы и первоначальную
регистрацию сцен.

### Представление (View)

-   **Контейнер (Container)** — объект, который
    является контейнером для виджетов. Реализует пространственную
    навигацию по помещённым в него виджетам. При наступлении события
    навигации передвижение курсора или нажатие навигационных кнопок на
    пульте) перемещает фокус на подходящий виджет;
-   **Слой (Layer)** — контейнер, основной элемент <abbr title="Document Object Model">DOM'a</abbr>. Слоем может
    быть сцена (scene) или попап (popup). Может содержать дочерние слои;
-   **Виджет (Widget)** — контейнер, помещающийся в слой. Управляет
    своим состоянием и видимостью. Также содержит методы `beforeDOMShow`,
    `afterDOMShow`, `beforeDOMHide`, `afterDOMHide`, которые вызываются либо
    из слоя, куда включён виджет, либо при изменении видимости.

### Модель (Model)

Для структурного представления используются модели (models). Модель
(`AbstractModel`) используется как контейнер для хранения данных, с
распространением события при изменении хранимых данных (`EVENT_CHANGE`).  

Управление слоями
-----------------
Управлением слоями занимается менеджер слоёв (`LayerManager`).
Регистрирует слои, которые соответствующим образом обрабатываются. Так,
например, при инициализации в нем регистрируются сцены, с помощью
метода `addScene`.

Глобальный объект приложения предоставляет следующие методы, которые
делегируют действия к менеджеру слоёв:

-   `addScene` — регистрирует сцену по указанному имени;
-   `show` — открывает зарегистрированную сцену по указанному имени,
    также принимает на вход объект данных, с которым будет вызван метод
    `preload`;
-   `home` – открывает домашнюю сцену.

Включение слоя в <abbr title="Document Object Model">DOM</abbr> осуществляется при регистрации сцены или вручную, с
помощью метода `app.showChildLayer` или `app.showChildLayerInstance`,
принимающие конструктор или инстанс слоя соответственно. 

В процессе обработки менеджером слоёв (`LayerManager`) выполняются
методы, дополнением которых можно проконтролировать процесс обработки
слоя:

-   `preload` — принимает объект данных и возвращает Promise.
    Вызывается перед показом слоя. Показ сцены осуществится только когда
    Promise разрешится. Используется, когда слою требуются внешние
    данные, нередко подгружаемые асинхронным запросом;
-   `update` — принимает снэпшот (snapshot) и
    объект данных из истории переходов, возвращает Promise. Вызывается
    при перемещении назад по истории переходов. Показ сцены осуществится
    только когда Promise разрешится. Используется, когда нужно обновить
    данные, хранящиеся в истории переходов;
-   `beforeDOMShow` — принимает снэпшот и объект данных.  Вызывается
    после `preload`, но перед тем, как показать слой.
    Вызывает `beforeDOMShow` для всех своих виджетов;
-   `afterDOMShow` — принимает снэпшот и объект данных. Вызывается
    после показа слоя. Вызывает `afterDOMShow` для всех своих виджетов.
    При наличии снэпшота загружает его, в противном случае активирует
    виджет по умолчанию (если такой установлен);
-   `beforeDOMHide` — вызывается перед скрытием слоя.
    Вызывает `beforeDOMHide` для всех своих виджетов;
-   `afterDOMHide` — вызывается после скрытия слоя. Вызывает
    `afterDOMHide` для всех своих виджетов.  

Пространственная навигация
--------------------------
Навигация осуществляется одним из двух способов:

1.  Проверяется наличие подходящего правила перехода. Правило перехода
    задаётся с помощью метода `setNavigationRule`, в который передаётся
    виджет, с которого осуществляется переход, направление, виджет на
    который осуществляется переход, а также необязательный флаг
    двунаправленности, при наличии которого будет справедливо обратное
    правило для данного перехода;
2.  Если подходящего правила перехода не найдено, задействуется
    пространственная навигация. В этом случае подходящий виджет
    определяется на основании положения предыдущего виджета.

Виджеты, помещённые в контейнер, также являются контейнерами и могут
содержать другие виджеты. В связи с этим иногда может потребоваться
задать виджет по умолчанию, который будет активирован в случае, когда
предыдущего активного виджета ещё нет. Это делается с помощью метода
`setDefaultWidget`, который принимает инстанс виджета и делает его
виджетом по умолчанию.

В целях отладки существует возможность активации debug-режима с помощью
`setNavigationDebug`. В этом случае в процессе навигации границы
виджетов будут подсвечиваться разными цветами:

-   Красный — предыдущий виджет, который находился в фокусе до
    наступления события навигации;
-   Зелёный — текущий виджет в фокусе;
-   Серый — все остальные виджеты, попавшие в область навигации.

Управление историей переходов
-----------------------------
Управлением историей переходов занимается менеджер истории (`HistoryManager`).
История представляет собой список записей истории (history
record). Каждая запись содержит ссылку на объект сцены, которой она
принадлежит, снэпшот, а также объект данных, с которым она была
загружена. 

Глобальный объект приложения предоставляет следующие методы, которые
делегируют действия к менеджеру истории:

-   `clearHistory` — очищает все существующие записи истории;
-   `back` – движение назад по записям истории. В случае окончания
    истории переходов — выполняется выход из устройства. Так как
    выполняется показ сцены, который может занять время, на этот период
    компонент ввода устройства (`IInput`) блокируется, после того как
    сцена будет показана — ввод разблокируется;
-   `forward` – движение вперёд по записям истории. При этом также
    блокируется ввод.

Сохранение состояния
--------------------
**Снэпшот (Snapshot)** — объект, который содержит в себе состояние,
сделавшего его слоя, а также снэпшоты всех виджетов, включённых в слой.
Состоянием могут являться данные любого типа, которые помогут на их
основании актуализировать слой.

Перед тем, как скрыть слой, менеджер истории делает снэпшот и сохраняет
его в записи истории. При последующем открытии слоя, сохранённый снэпшот
передаётся в метод `afterDOMShow`, где он потом загружается и
устанавливает состояние слоя и его виджетов.

Так как и слой и виджеты — это контейнеры, чтобы осуществить сохранение
и применение состояния, достаточно переопределить методы `saveState` и
`loadState`, которые являются частью API контейнера.  

Обработка ввода
---------------
Обработка нажатий кнопок пульта осуществляется по принципу всплытия
событий (event propagation). Местом зарождения события является
`InputDispatcher`, который в результате взаимодействия с компонентом ввода
платформы (`IInput`) производит обработку событий нажатия. При
наступлении соответствующего события, внутренний код нажатой кнопки
переводится в унифицированный код фреймворка и дальше передаётся на
обработку глобальному объекту приложения.

Глобальный объект приложения затем запускает погружение события нажатия
по иерархии композиции слоёв. Вначале будет вызван метод `processKey` у
текущей сцены, которая при наличии в ней дочерних слоёв, вызовет
`processKey` у верхнего дочернего слоя (top child layer), в противном
случае передаст обработку нажатия своему активному виджету, а тот своему
и т.д. Если событие не обработано в процессе погружения, то виджет по
умолчанию вызывет метод `_processKey`, в котором производит обработку
событий связанных с его собственной логикой. В базовой реализации виджет
обрабатывает в методе `_processKey` события кнопок навигации (**UP**, **DOWN**,
**LEFT**, **RIGHT**). В случае, если ни на одном из уровней погружения событие
не было обработано, вызывается метод `_processKey` уже непосредственно у
самого глобального объекта приложения, который в случае, если нажата
кнопка **BACK**, выполнит перемещение по истории записей назад.

Так же `InputDispatcher` отвечает за обработку курсора, если он
поддерживается устройством. Для этого контейнер, после передачи в него
виджета, с помощью `InputDispatcher` назначает обработчики на события
мышки <abbr title="Document Object Model">DOM</abbr> элемента виджета
(`mouseover`, `click`, `mousewheel`), при наступлении которых,
виджет обрабатывает его соответствующим образом.
Так, например, на событие `mouseover` виджет активируется, а на событие
`click` — сэмулирует нажатие на него кнопки **ENTER**.  

Выполнение асинхронных запросов
-------------------------------
В состав фреймворка входит компонент `Transport`, который реализует
сохраняемый запрос (persistent request). Это значит, что при неуспешном
выполнении (promise of query was rejected) сценарий может пойти по трём
путям: повтор запроса, отмена или выход из устройства. Для этого ему
передаётся обработчик, который должен вернуть один из трёх кодов
состояния. Таким обработчиком может быть, например, показ попапа,
который разрешится с соответствующим кодом по нажатию одной из его
кнопок.  

Запуск приложения
-----------------
После инстанцирования в точке входа, приложение проходит следующие
стадии:

-  Создание глобального <abbr title="Document Object Model">DOM</abbr> — сюда входят контейнеры для видео-объекта,
    слоёв, системный контейнер, контейнер для плагинов устройства.
    Контейнер — это простой `HTMLDivElement`;
-  Файрится событие `EVENT_DOM_READY`;
-  Выбор платформы и инициализация устройства;
-  Файрится событие `EVENT_DEVICE_READY`;
-  Инициализируются менеджеры истории и слоёв, `InputDispatcher`, к
    <abbr title="Document Object Model">DOM'у</abbr> применяется разрешение экрана;
-  Вызывается метод `onReady` — он может быть переопределён и нужен в
    том случае, если необходимо выполнить какие-то действия до
    регистрации сцен;
-  Регистрируются сцены приложения;
-  Вызывается метод `onStart` — также переопределяется. Здесь может
    находиться процесс открытия домашней сцены с помощью метода `home`,
    если предварительно были установлены её имя и необязательный объект
    данных (`setHomeScene`) или альтернативные действия.  

Логирование
-----------
Логирование реализуется несколькими способами, за каждый из которых
отвечает свой тип логгера:

-   Вывод в консоль (`Console`);
-   При помощи модального окна (`Alert`);
-   Методом отправки лог-сообщений на удалённый сервер (`Remote`).

Каждый тип логгера реализует интерфейс `ILogger`, тем самым возможно
создание собственных вариантов логирования.

Есть возможность назначить требуемый уровень логирования. В этом случае,
если вызываемый метод логгера не удовлетворяет текущему уровню, его вызов
будет проигнорирован. Доступные уровни: **ALL**, **LOG**, **DEBUG**, **INFO**, **WARN**,
**ERROR**, **ASSERT**, **DIR**, **TIME**.

Процесс и конфигурирование осуществляются с помощью объекта `zb.console`
со следующим API:

-   `zb.console.[log|debug|info|warn|error|assert|dir]` —
    логирование с соответствующим уровнем. В случае типа `Console`
    соответствующие методы заменяются на нативные браузерные;
-   `zb.console.[time|timeEnd]` — управление таймером;
-   `zb.console.setLevel` — устанавливает требуемый уровень
    логирования. Содержит маску, полученную путём применения побитового
    **ИЛИ** к набору уровней.
    Например: `zb.console.setLevel(zb.console.Level.LOG | zb.console.Level.DEBUG)`;
-   `zb.console.setLogger` — устанавливает тип логирования.
    Принимает инстанс логгера, реализующий интерфейс `ILogger`.
