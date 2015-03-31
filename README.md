#Rutube Player API

Пример встраивания плеера Rutube на HTML-страницу:
```html
<iframe width="720" height="405" src="//rutube.ru/play/embed/7163336" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowfullscreen></iframe>
```

Опционально, в адресную строку можно передать параметры:

`bmstart` - время старта, в секундах;

`quality` - отдаваемое качество видео. `"1"` - одно качество начиная с самого высокого, `"-2"` - два качества начиная с самого низкого;

_* При встраивании плеера Rutube в среде Samsung Smart TV лучше отдавать одно самое высокое качество, так как при автоматическом переключении качества ресайз картинки может происходить некорректно._

Пример встраивания плеера Rutube с параметрами:
```html
<iframe width="720" height="405" src="//rutube.ru/play/embed/7163336?quality=1&platform=samsung-smarttv" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowfullscreen></iframe>
```

Управлять загруженным плеером можно с помощью специального API, реализация которого основана на интерфейсе [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window.postMessage).

## Отправка команд плееру

_Здесь и далее все примеры приведены на VanillaJS, однако, существуют сторонние плагины, упрощающие работу с интерфейсом postMessage._

Пример отправки сообщения плееру:
```javascript
var player = document.getElementById('my-player');
player.contentWindow.postMessage(JSON.stringify({
    type: 'some:method',
    data: 'some data'
}), '*');
```

События, которые можно отправить плееру:
```javascript
// проиграть видео
{
    type: 'player:play',
    data: {}
}
```

```javascript
// пауза
{
    type: 'player:pause',
    data: {}
}
```

```javascript
// стоп (сброс буфера видео и рекламы)
{
    type: 'player:stop',
    data: {}
}
```

```javascript
// установка текущего времени проигрывания, в секундах
{
    type: 'player:setCurrentTime',
    data: {
        time: 20
    }
}
```

```javascript
// смена ролика
{
    type: 'player:changeVideo',
    data: {
        id: 'xyz123' // id ролика
    }
}
```

С помощью этого метода также можно загрузить в плеер скрытый ролик, передав параметр `"p"` (приватный ключ):
```javascript
{
    type: 'player:changeVideo',
    data: {
        params: {
            hash: 'xyz123', // id ролика
            p: 'abc789'
        }
    }
}
```

И загрузить видео сразу в самом высоком качестве (без автоматического переключения), передавая параметр `"quality"` со значением `"1"`:
```javascript
{
    type: 'player:changeVideo',
    data: {
        params: {
            hash: 'xyz123', // id ролика
            quality: 1
        }
    }
}
```

## Подписка на сообщения плеера

Пример подписки на сообщения от плеера:
```javascript
window.addEventListener('message', function (event) {
    var message = JSON.parse(event.data);
    console.log(message.type); // some type
    switch (message.type) {
        case 'player:changeState':
            console.log(message.data.state); // текущее состояние плеера
            break;
    };
});
```

Шаблон приходящих сообщений:
```javascript
{
    type: 'event:type',
    data: 'some data'
}
```

Типы приходящих сообщений (начинаются с префикса player):

`player:ready` - плеер загружен готов к проигрыванию (отправляется один раз, при вставке плеера на страницу);

`player:changeState` - изменение состояния плеера;

Объект `data` в этом случае cодержит параметры:

- `state` - состояние плеера: _playing_, _paused_, _stopped_, _lockScreenOn (появление заглушки)_, _lockScreenOff (снятие заглушки)_;

- `isLicensed` - флаг лицензионности ролика.

`player:currentTime` - текущее вермя проигрывания ролика;

Объект `data` в этом случае cодержит параметр _time_ со значением времени в секундах.

`player:rollState` - проигрывание рекламы в эмбеде;

Объект `data` в этом случае cодержит параметры со значениями:

- `rollState` - тип рекламного ролика: _playPreroll_, _playPausebanner_, _playPostroll_, _playMidroll_, _playPauseroll_, _playOverlay_;

- `state` - состояние рекламного ролика: _play_, _complete_.

`player:playComplete` - событие, отправляемое в момент когда ролик завершил проигрывание.