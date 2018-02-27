#Rutube Player API

Пример встраивания плеера Rutube на HTML-страницу:
```html
<iframe width="720" height="405" src="//rutube.ru/play/embed/7163336" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowfullscreen></iframe>
```

Опционально, в адресную строку можно передать параметры:

`bmstart` - время старта, в секундах;

`quality` - отдаваемое качество видео. `"1"` - одно качество начиная с самого высокого, `"-2"` - два качества начиная с самого низкого;

Пример встраивания плеера Rutube с параметрами:
```html
<iframe width="720" height="405" src="//rutube.ru/play/embed/7163336?quality=1&platform=someplatform" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowfullscreen></iframe>
```

Управлять загруженным плеером можно с помощью специального API, реализация которого основана на интерфейсе [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window.postMessage).


## Управление проигрыванием

_Здесь и далее все примеры приведены на VanillaJS, однако, существуют сторонние плагины, упрощающие работу с интерфейсом postMessage._


### Пример отправки сообщения плееру:
```javascript
var player = document.getElementById('my-player');
player.contentWindow.postMessage(JSON.stringify({
    type: 'some:method',
    data: 'some data'
}), '*');
```

### Команды, которые можно отправить плееру:

*player:play*
Начать проигрывание видео
```javascript
{
    type: 'player:play',
    data: {}
}
```

*player:pause*
Поставить видео на паузу
```javascript
// пауза
{
    type: 'player:pause',
    data: {}
}
```

*player:stop*
Закончить цикл проигрывания (сброс буфера видео и рекламы)
```javascript
{
    type: 'player:stop',
    data: {}
}
```

*player:setCurrentTime*
Переход к определенной секунде видео
```javascript
{
    type: 'player:setCurrentTime',
    data: {
        time: 20
    }
}
```

*player:relativelySeek*
Перемотать видео на определенное количество секунд вперед или назад. 
time - количество секунд для перемотки
- (знак минус) - перемотка назад 
+ (знак плюс) - вперед
```javascript
{
    type: 'player:relativelySeek',
    data: {
        time: +20
    }
}
```

*player:changeVideo*
Загрузить в плеер другое видео
```javascript
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

*player:mute*
Выключение звука
```javascript
{
    type: 'player:mute'
}
```

*player:unMute*
Включение звука
```javascript
{
    type: 'player:unMute'
}
```

*player:setVolume*
Установка уровня звука
```javascript
{
    type: 'player:setVolume',
    data: {
            volume: 0.20//значение от 0 до 1
          }

}
```

*player:setSkinColor*
Изменить цветовую схему скина
```javascript
{
    type: 'player:setSkinColor',
    data: {
        params: {
            color: '393939' // цвет в RGB, HEX (без решетки)
        }
    }
}
```

*player:remove*
Удаление плеера, освобождение используемых ресурсов. Рекомендуется вызывать перед удалением эмбеда со страницы. 
```javascript
{
    type: 'player:remove'
}
```


## Отслеживание статуса проигрывания

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

### Шаблон приходящих сообщений:
```javascript
{
    type: 'event:type',
    data: 'some data'
}
```

### Сообщения:

*player:ready*
Плеер загружен готов к проигрыванию (отправляется один раз, при вставке плеера на страницу):
```javascript
{
	data: {},
	type: 'player:ready'
}
```

*player:changeState*
Изменилось состояние поигрывания.
Параметр:
* state - текущее состояние плеера
Возможные значения параметра state:
* playing - плеер перешел в состояние проигрывания видео
* paused - видео поставлено на паузу
* stopped - плеер закончил проигрывание видео
* lockScreenOn - появление заглушки в плеере
* lockScreenOff - снятие заглушки в плеере
```javascript
{
	data: {
		state: 'playing',
		isLicensed: true
	},
	type: 'player:changeState'
}
```

*player:durationChange*
Обновление/уточнение длительности видео.
Параметры
* duration - длительности видео, в секундах
```javascript
{
    data: {
        duration: 50.5
    },
    type: 'player:durationChange'
}
```

*player:currentTime*
Информация о текущем времени проигрывания ролика.
Параметры
* time - текущее время проигрывания ролика, в секундах
```javascript
{
    data: {
        time: 4.009
    },
    type: 'player:currentTime'
}
```
*player:rollState*
Информация о факте и статусе проигрывания рекламы в плеере.
Параметры
* rollState - тип проигрываемой рекламы
* state - состояние проигрывания рекламного ролика
* guid - уникальный идентификатор сессии плеера
Возможные значения параметра rollState:
* Preroll - преролл
* Pausebanner - БНП (баннер на паузе)
* Postroll - постролл
* Midroll - мидролл
* Pauseroll - паузролл
* Overlay - оверлей
Возможные значения параметра state:
* play - плеер перешел в состояние проигрывания рекламы
* complete - плеер закончил проигрывание рекламы\ошибка\нет рекламы
```javascript
{
    data: {
        rollState: 'playPreroll',
        state: 'play',
        guid: 'BFEE79FD-F328-86A1-83F7-58489FF4AEB4'
    },
    type: 'player:rollState'
}
```

*player:changeFullscreen*
Переход/выход из фуллскрина
```javascript
{
    data: {
        isFullscreen:true/false
    },
    type: 'player:changeFullscreen'
}
```

*player:error*
Ошибка во время проигрывания
Параметры
* code - код ошибки проигрывания
* text - текст ошибки
```javascript
{
    data: {
        code: '404',
        text: 'Все пропало',
    },
    type: 'player:error'
}
```

*player:playComplete*
Событие окончания проигрывания (видео и рекламы). Переход плеера в эндскрин.
```javascript
{
	data: {},
	type: 'player:playComplete'
}
```

*player:volumeChange*
Информация о смене уровня звука.
Параметры
* volume - текущий уровень звука, от 0 до 1
```javascript
{
    data: {
        volume: 0.5
    },
    type: 'player:volumeChange'
}
```
