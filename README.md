# Документация для интеграции с 3D Платформой

(c) 2023 ООО "3Д ПЛАТФОРМА" ИНН 7702437500

- [Регистрация аккаунта и получение API ключа](#регистрация-аккаунта-и-получение-api-ключа)
- [Загрузка 3D-изображений](#загрузка-3d-изображений)
- [Методы API](#методы-api)
  - [Получение кода встройки по артикулу](#получение-кода-встройки-по-артикулу)
  - [Получение кода встройки по URL](#получение-кода-встройки-по-url)
  - [Дополнительные настройки проигрывателя](#дополнительные-настройки-проигрывателя)
  - [Список загруженных файлов](#список-загруженных-файлов)
  - [Получить информацию о модели по артикулу или ID](#получить-информацию-о-модели-по-артикулу-или-id)
  - [Получить превью-изображение для модели](#получить-превью-изображение-для-модели)
- [Взаимодействие с проигрывателем](#взаимодействие-с-проигрывателем)
  - [Типы событий](#типы-событий)
    - [Уведомление по отрисовке фрейма](#уведомление-по-отрисовке-фрейма)
    - [Полная загрузка модели](#полная-загрузка-модели)
    - [Ошибки отображения проигрывателя](#ошибки-отображения-проигрывателя)
- [Управление поведением проигрывателя](#управление-поведением-проигрывателя)
  - [Доступные методы](#доступные-методы)
    - [`rotateToDeg`](#rotatetodeg)
    - [`enterFullscreen`](#enterfullscreen)
    - [`cancelFullscreen`](#cancelfullscreen)
    - [`cancelZoom`](#cancelzoom)
- [Передача аналитических данных](#передача-аналитических-данных)
- [Ограничения на запросы](#ограничения-на-запросы)

## Регистрация аккаунта и получение API ключа

1. Зарегистрируйте свой аккаунт https://3dplatforma.ru/register 
2. Подтвердите вашу учетную запись
3. Откройте аккаунт и создайте токен - https://3dplatforma.ru/account/security - он потребуется для взаимодействия с API. Пожалуйста сохраните его, так как мы не даем возможность его смотреть повторно по соображениям безопасности.

## Загрузка 3D-изображений

1. Используйте приложение для macOS, Windows или мобильные приложения 3D Платформы для загрузки контента. 3D-модели и панорамные изображения можно загружать прямо с платформы.
2. При загрузке контента вам нужно назначить номер артикула, чтобы автоматически синхронизировать контент с вашим магазином.
3. Номер артикула можно отредактировать после загрузки контента.

## Методы API

Есть два способа интеграции с платформой - ручной и через API. Ручной метод подразумевают генерацию кода встройки (iframe) и копирование его в код вашего сайта. Но если у вас большое количество товаров, то мы рекомендуем использовать API для получения кода встройки. Ответ от API мы рекомендуем кешировать на вашей стороне. Обратите внимание, что API доступно только для корпоративных клиентов и есть [Ограничения по отправлению запросов](#ограничения-на-запросы). Свяжитесь с вашим менеджером от ООО "3Д ПЛАТФОРМА", чтобы получить доступ к API.

### Получение кода встройки по артикулу

1. Мы поддерживаем gzip. Используйте  'Accept-Encoding: gzip'. Curl автоматически кодирует/декодирует с параметром  `--compressed`.
2. Мы стремимся соответствовать схеме `http://jsonapi.org/`, и поэтому мы отвечаем типом содержимого `application/vnd.api+json`, это по сути то же самое, что и `application/json`, но уведомляет вам, что мы следуем этой спецификации.
3. Мы ожидаем, что вы отправляете контент в формате jsonapi, поэтому укажите в заголовке `Content-Type: application/vnd.api+json` .
4. Этот запрос требует аутентификации - укажите заголовок `Authorization: Bearer <ваш токен>` .
5. Чтобы обеспечить согласованность версий API, вы можете передать дополнительный заголовок `Accept-Version: ~1` который сообщит, что вы хотите получить ответ API версии 1. В дальнейшем если мы что-то изменим - в вашей интеграции ничего не сломается.

Это пример запроса CURL для этого метода. Более подробный пример вы найдете в [файле sku-embed.js](./sku-embed.js).

```bash
curl -X POST --compressed \
  -H "Content-Type: application/vnd.api+json" \
  -H "Authorization: Bearer hash.token.signature" \
  "https://api.3dplatforma.ru/api/files/embed" \
  -d '{
    "data": {
      "id": "sku2137189",
      "type": "embed",
      "attributes": {
        "width": 100,
        "height": 600,
        "autorun": true,
        "closebutton": true,
        "hidecontrols": false,
        "logo": true,
        "hidefullscreen": false
      }
    }
  }'
```

HTTP ответ будет с кодом `200` и следующей JSON структурой:
```json
{
  "meta": {
    "id": "request-id"
  },
  "data": "iframe code"
}
```

### Получение кода встройки по URL

Вы можете достать iframe и из прямой ссылки на модель. Пример ссылки`https://3dplatforma.ru/3dplatforma/9636a6c3-64e5-4054-8def-a8c185389a1e`. Полный пример можно найти [здесь](./marketplace.js).

```bash
curl -X POST --compressed \
  -H "Content-Type: application/vnd.api+json" \
  -H "Authorization: Bearer hash.token.signature" \
  "https://api.3dplatforma.ru/api/oembed/marketplace?url=https%3A%2F%2F3dplatforma.ru%2F3dplatforma%2F9636a6c3-64e5-4054-8def-a8c185389a1e" \
  -d '{
    "data": {
      "type": "embed",
      "attributes": {
        "width": 100,
        "height": 600
      }
    }
  }'
```

HTTP ответ будет с кодом `200` и следующей JSON структурой:

```json
  {
    "meta": {
      "id": "<request id>"
    },
    "data": {
      "id": "<user id>/<model id>",
      "type": "embed",
      "attributes": {
        "html": "<iframe ...></iframe>"
      }
    }
  }
```

### Дополнительные настройки проигрывателя

Ниже список допустимых параметров:

| Параметр    | Тип  | По-умолчанию | Описание                                                                        |
|---------------------|---------|---------|-------------------------------------------------------------------------------------------|
| `width`             | integer, >= 100  | 100    | Ширина iFrame, px или %. Считаем 100 за `100%` иначе px                              |
| `height`            | integer, >= 100  | 600    | Высота iFrame, px или %. Считаем 100 за `100%` иначе px                                         |
| `autorun`           | boolean | false   | Запускаем загрузку сразу или ждем нажатие на кнопку |
| `closebutton`       | boolean | true    | Показать иконку закрытия                                                  |
| `logo`              | boolean | true    | Показать логотип платформы                                             |
| `autorotate`        | boolean | false   | Запустить автовращение                                            |
| `autorotatetime`    | float   | 10      | Время полного круга, секунды                          |
| `autorotatedelay`   | float   | 2       | Ожидание если автовращение прервали, секунды       |
| `autorotatedir`     | integer | 1       | Направление автовращения (по часовой `1`, против часовой  `-1`) |
| `hidefullscreen`    | boolean | true    | Скрыть кнопку активации режима полного экрана                 |
| `hideautorotateopt` | boolean | true    | Скрыть кнопку активации автовращения                                 |
| `hidesettingsbtn`   | boolean | false   | Скрыть кнопку показа настроек                                          |
| `enableimagezoom`   | boolean | true    | Включить зум                                                                    |
| `zoomquality`       | integer | 1       | Качество зума (SD в `1`, HD в `2`)                                         |
| `hidezoomopt`       | boolean | false   | Скрыть кнопку зума                                                         |
| `uipadx`            | integer | 0       | Горизонтальное (влево, вправо) смещение UI в пикселях |
| `uipady`            | integer | 0       | Вертикальное (вверх, вниз) смещение UI в пикселях |
| `enablestoreurl`    | boolean | false   | Активировать кнопку переход по URL                 |
| `storeurl`          | string  | ''      | URL для перехода                                                     |
| `hidehints`         | boolean | false   | Скрыть подсказки                                                     |
| `starthint`         | boolean | false   | Скрыть только первую подсказку                                |
| `arbutton`          | boolean | true    | Показать AR кнопку                                                           |

### Список загруженных файлов

Этот метод позволяет вернуть список загруженных файлов и разбить их на страницы. Например, вы можете получить список загруженных файлов между двумя моментами времени для вашего пользователя. Максимальный интервал - 30 дней.
Из-за архитектуры базы данных список возвращенных моделей будет внутренне кэшироваться до тех пор, пока не произойдет один из трех случаев: последний доступ к списку был выполнен более 30 секунд назад, модель была загружена или модель была удалена. В будущем это может измениться.

Ниже описание параметров:

| Тип    | Имя             | По-умолчанию | Доступно         | Пример                          | Комментарий                                                  |
| ------ | --------------- | ------------ | ---------------- | ------------------------------- | ------------------------------------------------------------ |
| Header | Authorization   |              |                  | `Authorization: Bearer <token>` | Если не указано - будут возвращены только публичные модели   |
| Header | Accept-Encoding |              |                  | `Accept-Encoding: gzip`         | Если не указано - будет возвращен обычный текст, пожалуйста, используйте его |
| Header | Accept-Version  |              |                  | `Accept-Version: ~1`            | Если не указано - будет использоваться самая последняя версия при критических изменениях |
| Query  | pub             |              | 0, 1             | `?pub=0`                        | Если установлен заголовок авторизации & pub=0 - будут включены приватные модели |
| Query  | order           | ASC          | ASC, DESC        | `?order=DESC`                   | По умолчанию по возрастанию                                  |
| Query  | offset          | 0            | 0 < offset       | `?offset=24`                    | Используется для пагинации                                   |
| Query  | limit           | 12           | 0 < limit <= 100 | `?limit=24`                     | Моделей на страницу                                          |
| Query  | filter          | `%7B%7D`     |                  | `?filter=%7B%7D`                | Используется для фильтрации ответа                           |
| Query  | criteria        | id           |                  | `?criteria=uploadedAt`          | Сортировка по этому полю                                     |
| Query  | shallow         | 0            |                  | `?shallow=1`                    | Установите значение 1, чтобы уменьшить трафик. Он опускает информацию о загруженных файлах |
| Query  | owner           |              |                  | `?owner=3dplatforma`            | Для публичных моделей — можно выбрать любой аккаунт, для показа приватных моделей — необходимо указать токен аутентификации |
| Query  | embed           | 0            | 0, 1             | `?embed=1`                      | Выведет embed.code в embed.html                              |
| Query  | embedParams     | %7B%7D       |                  | `?embedParams=%7B%7D`           | Переопределение параметров embed.code шаблона                |


Самым важным из всех параметров является фильтр. Для его создания используйте следующую функцию:

```js
function encodeFilter(obj) {
  return encodeURIComponent(JSON.stringify(obj));
}
```

Пример:

```js
const filter = encodeFilter({
  uploadedAt: {
    gte: 1486694997327, // miliseconds
    lte: 1487447115708, // miliseconds
  },
});
```

Пример запроса:

```bash
curl -X GET --compressed \
  -H "Authorization: Bearer hash.token.signature" \
  "https://api.3dplatforma.ru/api/files?owner=3dplatforma&sortBy=uploadedAt&order=DESC&shallow=1&offset=0&limit=24&filter=%7B%22uploadedAt%22%3A%7B%22gte%22%3A1486694997327%2C%22lte%22%3A1487447115708%7D%7D"
```

### Получить информацию о модели по артикулу или ID

Этот эндпоинт предоставляет расширенную информацию о модели.

```
  https://api.3dplatforma.ru/api/files/info/<user-alias>/<sku>
  https://api.3dplatforma.ru/api/files/info/<user-alias>/<id>
```

`<user-alias>` - это имя аккаунта, которое вы зарегистрировали на платформе.  `sku` (артикул) или `ID` должны соответствовать правилам, которые вы можете почитать в [File.json#/properties/id](file.json)

Пример запроса для модели с артикулом "A B C":

```bash
curl -X GET --compressed \
  -H "Content-Type: application/vnd.api+json" \
  "https://api.3dplatforma.ru/api/files/info/3dplatforma/A%20B%20C"
```

Example response:

```json5
{
  "meta": {
    "id": "6adbc6d2-69b4-4c88-91a8-8f99f54b083d"
  },
  "data": {
    "type": "file",
    "id": "9636a6c3-64e5-4054-8def-a8c185389a1e", /* model's ID */
    "attributes": {
      "public": "1",
      "contentLength": 19919914,
      "name": "Prosecco",
      "files": [/* internal file data */],
      "parts": 5,
      "tags": [
        "drink",
        "alcoholic beverage",
        "liqueur",
        "champagne",
        "wine",
        "distilled beverage",
        "bottle",
        "alcohol",
        "sparkling wine",
        "glass bottle"
      ],
      "type": "object",
      "uploadedAt": 1579521161823,
      "embed": {
        "ai": "<script async src=\"https://api.3dplatforma.ru/api/player/3dplatforma-ai\"></script>",
        "code": "<iframe allowfullscreen mozallowfullscreen=\"true\" webkitallowfullscreen=\"true\" width=\"{{ width }}\" height=\"{{ height }}\" frameborder=\"0\" style=\"border:0;\" src=\"https://api.3dplatforma.ru/api/player/95f59308-3e2c-4151-a541-1db0aac3ad0d/embedded?autorun={{ autorun }}&closebutton={{ closebutton }}&logo={{ logo }}&analytics={{ analytics }}&uipadx={{ uipadx }}&uipady={{ uipady }}&enablestoreurl={{ enablestoreurl }}&storeurl={{ storeurl }}&hidehints={{ hidehints }}&autorotate={{ autorotate }}&autorotatetime={{ autorotatetime }}&autorotatedelay={{ autorotatedelay }}&autorotatedir={{ autorotatedir }}&hidefullscreen={{ hidefullscreen }}&hideautorotateopt={{ hideautorotateopt }}&hidesettingsbtn={{ hidesettingsbtn }}&enableimagezoom={{ enableimagezoom }}&zoomquality={{ zoomquality }}&hidezoomopt={{ hidezoomopt }}\"></iframe>",
        "params": {/* embedded parameters available for the model */}
      },
      "bucket": "cdn.3dplatforma.ru",
      "alias": "A B C",
      "uploadType": "simple",
      "backgroundColor": "#FFFFFF",
      "packed": "1",
      "c_ver": "4.1.0",
      "owner": "3dplatforma"
    },
    "links": {
      "self": "https://api.3dplatforma.ru/api/files/9636a6c3-64e5-4054-8def-a8c185389a1e",
      "owner": "https://api.3dplatforma.ru/api/users/3dplatforma",
      "player": "https://3dplatforma.ru/3dplatforma/9636a6c3-64e5-4054-8def-a8c185389a1e",
      "user": "https://3dplatforma.ru/3dplatforma"
    }
  }
}
```

Для приватных моделей не забывайте про заголовок `authorization`.

### Получить превью-изображение для модели

`https://api.3dplatforma.ru/api/files/preview/<user-alias>/w640-h400-cpad-bffffff/<ID>.jpeg`
`https://api.3dplatforma.ru/api/files/preview/<user-alias>/<ID>.jpeg`
`https://api.3dplatforma.ru/api/files/preview/<user-alias>/w640-h400-cpad-bffffff/<sku>.jpeg`
`https://api.3dplatforma.ru/api/files/preview/<user-alias>/<sku>.jpeg`

Поддерживаемые модификаторы:

```
  - height:       eg. h500
  - width:        eg. w200
  - square:       eg. s50
  - crop:         eg. cfill
  - top:          eg. y12
  - left:         eg. x200
  - gravity:      eg. gs, gne
  - quality:      eg. q90
  - background:   eg. b252525
```

Примеры:

* 640x400, с сохранением пропорций (aspect ratio и padding) c белым фоном (по-умолчанию): https://api.3dplatforma.ru/api/files/preview/3dplatforma/w640-h400-cpad/9636a6c3-64e5-4054-8def-a8c185389a1e.jpeg
* оригинальные файл: https://api.3dplatforma.ru/api/files/preview/3dplatforma/9636a6c3-64e5-4054-8def-a8c185389a1e.jpeg

## Взаимодействие с проигрывателем

iframe проигрывателя вызывает события используя [window.parent.postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) для определенных действий, которые происходят внутри игрока. Вы можете подключить прослушиватель верхнего окна, который позволит вам получать эти события.

Для этого вам нужно выполнить следующий код в окне верхнего уровня, которое содержит iframe с проигрывателем:

```js
window.addEventListener('message', receiveMessage, false);

// event: https://developer.mozilla.org/en-US/docs/Web/API/MessageEvent
function receiveMessage(event) {
  // verify that origin of event is 3D Platforma iframe
  if (event.origin !== 'https://api.3dplatforma.ru') return // ignore events which originate outside of iframe
  if (!event.data || !event.data.actionType) return
  
  // use event.source to determine which iframe sent the loaded message if you have multiple
  // iframes with the same model embedded on a single page
  
  switch (event.data.actionType) {
    case 'loaded':
      // model fully loaded
      // use event.data.modelId to determine which model had loaded
      break;
    case 'draw':
      // embed performed a frame draw
      // use event.data.modelId to determine which one
      break;
  }
}
```

### Типы событий

#### Уведомление по отрисовке фрейма

Уведомление о рендеринге кадра будет отправляться каждый раз, когда происходит отрисовка, в том числе при вращении.

```json
{
  "modelId": "uuid-v4-of-the-model",
  "actionType": "draw"
}
```

#### Полная загрузка модели

Это происходит, когда все сегменты 3D-изображения были загружены. Это не первая отрисовка - это именно полная загрузка всех данных. Чтобы иметь возможность отслеживать начальную отрисовку и интерактивность плеера - слушайте уведомление об отрисовке кадра.

```json
{
  "modelId": "uuid-v4-of-the-model",
  "actionType": "loaded"
}
```

#### Ошибки отображения проигрывателя

Все типы обработанных ошибок уведомляют родительский фрейм сообщением, структурированным следующим образом:

```json5
{
  "source": "3dplatforma-player",
  "actionType": "error",
  "modelId": "uuid-v4-of-the-requested-model",
  "message": "error details",
  "code": 404 // numeric code of the error
}
```

## Управление поведением проигрывателя

Вы можете контролировать поведение проигрывателя, отправляя `postMessage` сообщения к объекту `window` object относящемуся к iframe проигрывателя.

Например, если iframe проигрывателя имеет  `id="player"` вы можете отправить сообщение используя код:

```js
  var player = document.getElementById('player')
  
  if (player && player.contentWindow) {
    player.contentWindow.postMessage({ fn: <methodName>, args: [...] }, 'https://api.3dplatforma.ru')
  }
```

### Доступные методы

#### `rotateToDeg`

Поверните модель на определенный градус. Принимает единственный аргумент: градус для поворота. Число в диапазоне [0..360].

Метод будет работать когда произойдет хотя бы одно событие `draw`. Пример (поворот на 90 градусов):

```js
  document.getElementById('player').contentWindow.postMessage({ fn: 'rotateToDeg', args: [90] }, 'https://api.3dplatforma.ru')
```

#### `enterFullscreen`

Открыть проигрыватель в полноэкранном режиме.

```js
  document.getElementById('player').contentWindow.postMessage({ fn: 'enterFullscreen', args: [] }, 'https://api.3dplatforma.ru')
```

#### `cancelFullscreen`

Если проигрыватель находится в полноэкранном режиме, он сразу выходит из него. В противном случае ничего не делает.

```js
  document.getElementById('player').contentWindow.postMessage({ fn: 'cancelFullscreen', args: [] }, 'https://api.3dplatforma.ru')
```

#### `cancelZoom`

Если модель в проигрывателе увеличена, эта команда вернет ее обратно в обычное состояние. В противном случае ничего не делает.

```js
  document.getElementById('player').contentWindow.postMessage({ fn: 'cancelZoom', args: [] }, 'https://api.3dplatforma.ru')
```

## Передача аналитических данных
Вы должны подключить этот скрипт для правильного подсчета тарификации и работы AR:
```
<script async src="https://api.3dplatforma.ru/api/player/3dplatforma-ai" />
```


### Ограничения на запросы

#### По синхронизации элементов для регистрации
##### По заданию
* Максимум 500 моделей за раз

Чтобы обработать каталог большего размера, разбейте ваши элементы на несколько заданий.

##### По число заданий за 24 часа
* Минимум 10,000 моделей за 24 часа. Этот параметр зависит от вашего тарифного плана.

  

#### По запросам

##### По IP
* Максимум 1000 запросов за 10 секунд

#### По соединениям

##### На один IP
* Максимум 60 новых соединений за 3 секунды
* Максимум 30 активных соединений (всего)

Переиспользуйте соединения.
