Работа с URL
============

>Замечание: раздел находится в разработке.

Концепция работы с URL в Yii довольно проста. Предполагается, что в приложении используются внутренние маршруты и параметры вместо жестко заданных URL. Тогда фреймворк сам преобразует маршруты в URL и обратно, в соотвествии с конфигурацией URL менеджера. Такой подход позволяет изменять вид URL на всем сайте, редактируя единственный конфигурационный файл не трогая код самого приложения.

Внутренние маршруты
-------------------

При создании приложения с помощью Yii, вы будете работать с внутренними маршрутами, которые также часто называются маршрутами c параметрами. Каждому контроллеру и действию соответствует внутренний маршрут, например `site/index` или `user/create`. В первом примере, site - это `ID контроллера` а `create` это ID действия. Если контроллер находится в модуле, то перед его ID контроллера в маршруте ставится `ID модуля`, например так: `blog/post/index`. Здесь `blog` - это ID модуля, а `post` и `index` - ID контроллера и действия соответственно.

Создание URL
------------

Самое важное при работе с URL = всегда создавать их через URL manager. Это встроенный компонент приложения, к которому можно обращаться по имени `urlManager` как в консольном, так и в веб-приложении, таким образом: `\Yii::$app->urlManager`. Через этот компонент доступны следующие методы создания URL:
- `createUrl($params)`
- `createAbsoluteUrl($params, $schema = null)`

Метод `createUrl` создает относительный от корня приложения URL, например `index.php/site/index` . Метод `createAbsoluteUrl` создает абсолютный URL, добавляя к относительному протокол и имя хоста: `http://www.example.com/index.php/site/index` . `createUrl` больше подходит для URL внутри самого приложения, тогда как `createAbsoluteUrl` - для создания ссылок на внешние ресурсы и для них, для решения задач отправки почты, генерации RSS и т.п.

Примеры:

```php
echo \Yii::$app->urlManager->createUrl(['site/page', 'id' => 'about']);
// /index.php/site/page/id/about/
echo \Yii::$app->urlManager->createUrl(['date-time/fast-forward', 'id' => 105])
// /index.php?r=date-time/fast-forward&id=105
echo \Yii::$app->urlManager->createAbsoluteUrl('blog/post/index');
// http://www.example.com/index.php/blog/post/index/
```

Точный формат URL зависит от конфигурации URL manager. В другой конфигурации примеры выше могут выводить:

*  `/site/page/id/about/`
*  `/index.php?r=site/page&id=about`
*  `/index.php?r=date-time/fast-forward&id=105`
*  `/index.php/date-time/fast-forward?id=105`
*  `http://www.example.com/blog/post/index/`
*  `http://www.example.com/index.php?r=blog/post/index`

Чтобы упростить создание URL рекомендуется пользоваться встроенным хелпером [[yii\helpers\Url]]. Покажем, как работает хелпер на примерах. Предположим, что мы находимся на /index.php?r=management/default/users&id=10 . Тогда:

```php
use yii\helpers\Url;

// текущий активный маршрут
// /index.php?r=management/default/users
echo Url::to('');

// тот же контроллер, другое действие
// /index.php?r=management/default/page&id=contact
echo Url::toRoute(['page', 'id' => 'contact']);


// тот же модуль, другие контроллер и действие
// /index.php?r=management/post/index
echo Url::toRoute('post/index');

// абсолютный маршрут вне зависимости от того, в каком контроллере происходит вызов
// /index.php?r=site/index
echo Url::toRoute('/site/index');

// url для регистрозависимого действия hiTech текущего контроллера
// /index.php?r=management/default/hi-tech
echo Url::toRoute('hi-tech');

// url для регистрозависимого контроллера, `DateTimeController::actionFastForward`
// /index.php?r=date-time/fast-forward&id=105
echo Url::toRoute(['/date-time/fast-forward', 'id' => 105]);

// получение URL через alias
// http://google.com/
Yii::setAlias('@google', 'http://google.com/');
echo Url::to('@google');

// получение URL "домой"
// /index.php?r=site/index
echo Url::home();

Url::remember(); // сохранить URL, чтобы использовать его позже
Url::previous(); // получить ранее сохраненный URL
```

>**Совет**: чтобы сгенерировать URL с хэштэгом, например `/index.php?r=site/page&id=100#title`, укажите параметр `#` в хелпере таким образом:
`Url::to(['post/read', 'id' => 100, '#' => 'title'])`.

Также существует метод `Url::canonical()` , который позволяет создать [канонический URL](https://en.wikipedia.org/wiki/Canonical_link_element) (статья на англ., перевода пока нет) для текущего действия. Этот метод при создании игнорирует все параметры действия кроме тех, которые были переданы как аргументы. 
Пример:

```php
namespace app\controllers;

use yii\web\Controller;
use yii\helpers\Url;

class CanonicalController extends Controller
{
    public function actionTest($page)
    {
        echo Url::canonical();
    }
}
```

При этом при текущем URL `/index.php?r=canonical/test&page=hello&number=42` канонический URL будет `/index.php?r=canonical/test&page=hello`.

Модификация URL
------------------

По умолчанию Yii использует url формата query string, например `/index.php?r=news/view&id=100`. Чтобы сделать более [человеко-понятные URL](https://ru.wikipedia.org/wiki/%D0%A7%D0%9F%D0%A3_%28%D0%98%D0%BD%D1%82%D0%B5%D1%80%D0%BD%D0%B5%D1%82%29), например для улучшения их читабельности, необходимо сконфигурировать компонент `urlManager` в конфигурационном файле приложения. Придав параметру `enablePrettyUrl` значение `true`, мы тем самым получим URL такого вида: `/index.php/news/view?id=100`, а выставив параметр `showScriptName' => false` мы исключим index.php из URL. Вот фрагмент конфигурационного файла:

```php
<?php
return [
    // ...
    'components' => [
        'urlManager' => [
            'enablePrettyUrl' => true,
            'showScriptName' => false,
        ],
    ],
];
```

Обратите внимание, что конфигурация с `'showScriptName' => false` будет работать только, если веб сервер был должным образом сконфигурирован. Смотрите раздел [installation](installation.md#recommended-apache-configuration) (ссылка не работает).

###Именованные параметры

Правило  может быть связано с несколькими `GET` параметрами. Эти `GET` параметры появляются в паттерне правила как специальные управляющие конструкции в таком формате:

```
<ИмяПараметра:ПаттернПараметра>
```

`ИмяПараметра` это имя `GET` параметра, а необязательный `ПаттернПараметра` - это регулярное выражение, которое используется для того, чтобы найти определенный `GET` параметр. В случае, если `ПаттернПараметра` не указан, это значит, что значение параметра это любая последовательность символов, кроме `/`. Правила используются и при парсинге URL, и при их создании; при создании URL параметры будут заменены на соответствующие значения, а при парсинге - `GET` параметры с соответствующими именами получат соответствующие значения из URL.

Приведем несколько примеров, чтобы пояснить, как работают правила. Предположим, что наш массив правил содержит три правила:

```php
[
    'posts'=>'post/list',
    'post/<id:\d+>'=>'post/read',
    'post/<year:\d{4}>/<title>'=>'post/read',
]
```


- Вызов `Url::toRoute('post/list')` генерирует `/index.php/posts`. Применяется первое правило.
- Вызов `Url::toRoute(['post/read', 'id' => 100])` генерирует `/index.php/post/100`. Применяется второе правило.
- Вызов `Url::toRoute(['post/read', 'year' => 2008, 'title' => 'a sample post'])` генерирует
  `/index.php/post/2008/a%20sample%20post`. Применяется третье правило.
- Вызов `Url::toRoute('post/read')` генерируется `/index.php/post/read`. Не применяется ни одного правила, вместо этого URL формируется по алгоритму по умолчанию.

Таким образом, при использовании `createUrl` для генерации URL, маршрут и переданные методу `GET` параметры используются для принятия решения о том, какое правило применить. 
Конкретное правило используется для генерации URL, если каждый параметр, указанный в правиле, может быть найден среди `GET` параметров, переданных `createUrl`, и сам маршрут, указанный в правиле, соответствует маршруту, переданному в качестве параметра.

Если параметров, переданных `Url::toRoute` больше, чем указано в правиле, то дополнительные параметры будут указаны в формате строки запроса. Например, при вызове `Url::toRoute(['post/read', 'id' => 100, 'year' => 2008])`, мы получим `/index.php/post/100?year=2008`.

Как было сказано ранее, другое назначение правил - парсинг URL. Это процесс, обратный их созданию. Например, когда пользователь запрашивает `/index.php/post/100`, будет применено второе правило из примера выше, которое извлечет маршрут `post/read` и `GET` параметр `['id' => 100]` (который можно получить так: 
`Yii::$app->request->get('id')`).

### Параметры в маршрутах

Мы можем обращаться к именованным параметрам в маршрутной части правила. Это позволяет нескольким маршрутам подпадать под правила, в соответствии с переданными параметрами. Это также может помочь минимизировать количество правил URL в приложении, тем самым улучшив его общую производительность.

Вот пример, показывающий как использовать маршруты с именованными параметрами.

```php
[
    '<controller:(post|comment)>/<id:\d+>/<action:(create|update|delete)>' => '<controller>/<action>',
    '<controller:(post|comment)>/<id:\d+>' => '<controller>/read',
    '<controller:(post|comment)>s' => '<controller>/list',
]
```

В этом примере, мы использовали два именованных параметра в маршрутной части правил: `controller` и `action`. Первый параметр подходит в случае, если controller ID - это post или comment, а последний - в случае, если action ID - create, update или delete. Имена параметров могут быть разными, но не должны совпадать по имени с GET-параметрами, которые могут появляться в URL.

Используя эти правила, URL `/index.php/post/123/create` после парсинга будет иметь маршрут `post/create` с `GET` параметром `id=123`. А с маршрутом `comment/list` и `GET` параметром `page=2` можно создать URL `/index.php/comments?page=2`.

### Параметры в имени хоста

Существует возможность включать имена хостов в правила парсинга и создания URL. Например, может возникнуть необходимость получить часть имени хоста в качестве `GET` параметра, чтобы работать с поддоменами. Например, URL `http://admin.example.com/en/profile` может быть разобран на `GET` параметры `user=admin` и `lang=en`. Аналогично, правила с именами хостов могут использоваться для создания URL.

Чтобы использовать параметры в имени хоста, просто объявите правило с именем хоста, например,

```php
[
    'http://<user:\w+>.example.com/<lang:\w+>/profile' => 'user/profile',
]
```

В этом примере первый сегмент имени хоста обрабатывается как имя пользователя (user), тогда как первый сегмент пути обрабатывается как параметр языка (lang). Правило относится к маршруту `user/profile`.

Обратите внимание, что атрибут [[yii\web\UrlManager::showScriptName]] не оказывает действия на правила, в которых используются параметры в имени хоста.

Также учтите, что любое правило с параметрами в имени хоста не должно содержать подпапку, если приложение размещено в подпапке web рута. Например, если приложение находится в `http://www.example.com/sandbox/blog`, тогда нам все равно в правиле нужно его прописать без `sandbox/blog`.

### Добавка суффикса URL

```php
<?php
return [
    // ...
    'components' => [
        'urlManager' => [
            'suffix' => '.html',
        ],
    ],
];
```

### Обработка REST запросов

TBD:
- RESTful маршрутизация: [[yii\filters\VerbFilter]], [[yii\web\UrlManager::$rules]]
- Json API:
  - response: [[yii\web\Response::format]]
  - request: [[yii\web\Request::$parsers]], [[yii\web\JsonParser]]


Парсинг URL
-----------

Помимо создания URL Yii также умеет парсить URL, получая на выходе маршруты и параметры.

### Строгий парсинг URL

По умолчанию если для URL не подошло ни одного заданного правила и URL соответствует стандартному формату, как, например `/site/page`, Yii делает попытку запустить указанный action соответствующего контроллера. Можно запретить обрабатывать URL стандартного формата, тогда, если URL не соответствует ни одному правилу, будет выброшена 404 ошибка.

```php
<?php
return [
    // ...
    'components' => [
        'urlManager' => [
            'enableStrictParsing' => true,
        ],
    ],
];
```

Создание классов для произвольных правил
------------------------------

[[yii\web\UrlRule]] класс используется для парсинга URL на параметры и создания URL по параметрам. Обычное решение для правил подойдет для большинства проектов, но бывают ситуации, когда лучшим выбором будет использования своего класса для задания правил. Например, на сайте автомобильного дилера могут быть URL типа `/Производитель/Модель`, где и `Производитель` и `Модель` хранятся в таблице БД. Обычное правило не сработает, т.к. оно основывается на статически заданных регулярных выражений, и извлечение информации из БД в нем не предусмотрено.

Мы можем создать новый класс для правил URL, унаследовав его от [[yii\web\UrlRule]] и использовав его в одном или нескольких URL правил. Ниже - реализация для вышеописанного примера с сайтом автомобильного дилера. Вот указание произвольного правила в конфигурации приложения:

```php
// ...
'components' => [
    'urlManager' => [
        'rules' => [
            '<action:(login|logout|about)>' => 'site/<action>',

            // ...

            ['class' => 'app\components\CarUrlRule', 'connectionID' => 'db', /* ... */],
        ],
    ],
],
```

В примере мы используем произвольное URL правило `CarUrlRule` для обработки URL формата `/Производитель/Модель`.

Код самого класса может быть примерно таким:

```php
namespace app\components;

use yii\web\UrlRule;

class CarUrlRule extends UrlRule
{
    public $connectionID = 'db';

    public function init()
    {
        if ($this->name === null) {
            $this->name = __CLASS__;
        }
    }

    public function createUrl($manager, $route, $params)
    {
        if ($route === 'car/index') {
            if (isset($params['manufacturer'], $params['model'])) {
                return $params['manufacturer'] . '/' . $params['model'];
            } elseif (isset($params['manufacturer'])) {
                return $params['manufacturer'];
            }
        }
        return false;  // это правило не подходит
    }

    public function parseRequest($manager, $request)
    {
        $pathInfo = $request->getPathInfo();
        if (preg_match('%^(\w+)(/(\w+))?$%', $pathInfo, $matches)) {
            // check $matches[1] and $matches[3] to see
            // if they match a manufacturer and a model in the database
            // If so, set $params['manufacturer'] and/or $params['model']
            // and return ['car/index', $params]
        }
        return false;  // это правило не подходит
    }
}
```

Кроме использования произвольных классов URL в случаях, подобных примеру выше, их также можно использовать для других целей. Например, мы можем написать класс правила для логгирования парсинга URL и создания запросов. Это может быть полезно на этапе разработки. Мы также можем написать класс правила чтобы показывать особую 404 страницу в случае, если другие правила не подошли для обработки текущего запроса. В этом случае произвольное правило должно стоять последним.
