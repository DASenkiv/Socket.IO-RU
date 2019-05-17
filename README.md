# Socket.IO-Documentation
This is russian translation of official documentation socket.io (https://socket.io/docs/).

От переводчика: 

> не смог найти достаточно подробный обзор, сайт с описанием данной
> библиотеки, кроме англоязычной документации socket.io, а на
> хабраютубах не находил ничего сложнее чата на вебсокетах.  Поэтому
> принял решение перевести официальную документацию на великий и могучий
> :)

Также перевод будет дополнен ответами на часто задаваемые вопросы и примерами.

Пожелания, замечания пишите в twitter [@dasenkiv](https://twitter.com/dasenkiv) или в комментарии здесь, на гитхабе.
Лучшей помощью, не считая доната, является помощь с переводом и распространение данной информации. Ну и, конечно, звезда на репозитории.

**P. S. здесь и далее будет удалена точка из названий библиотек, поскольку точка генерирует ссылку и вводит читателей в заблуждение (например, вместо Engine.IO будет написано EngineIO, вместо Socket.IO - SocketIO и т. д.
В тех местах, где ссылка присутствует, она является верной.**

# Содержание

 - [Обзор](#Обзор)
	 - [Что такое SocketIO](#Обзор)
	 - [Чем SocketIO не является](#Чем-SocketIO-не-является)
	 - [Установка](#Установка)
	 - [Использование с http-сервером Node](Использование-с-http-сервером-Node)
	 - [Использование с Express](Использование-с-Express)
	 - [Отправка и получение событий](Отправка-и-получение-событий)
	 - [Ограничение пространством имен](Ограничение-пространством-имен)
	 - [Отправка нестабильных сообщений](Отправка-нестабильных-сообщений)
	 - [Отправка и получение данных (подтверждений)](Отправка-и-получение-данных-(подтверждений))
	 - [Широковещательные сообщения](Широковещательные-сообщения)
	 - [Использование в качестве кроссбраузерного WebSocket](Использование-в-качестве-кроссбраузерного-WebSocket)
 - Миграция с 0,9
 - Использование нескольких узлов
 - Логирование и отладка
 - Шпаргалка по Emit
 - Обзор фреймворка изнутри
 - Часто задаваемые вопросы
## Обзор
SocketIO - это библиотека, которая обеспечивает двустороннюю и основанную на событиях связь в режиме реального времени между браузером и сервером. Она состоит из:
 - сервера Node.js: [Исходный код](https://github.com/socketio/socket.io) | [API](https://socket.io/docs/server-api/)
 - клиентской библиотеки Javascript для браузера (которую также можно запустить из Node.js): [Исходный код](https://github.com/socketio/socket.io-client) | [API](https://socket.io/docs/client-api/)
 

Его основными **особенностями** являются:

### Надежность

Соединения устанавливаются даже при наличии:

 - прокси-серверов и балансировщиков нагрузки;
- персональных брандмауэров и антивирусного ПО.
Для этой цели используется EngineIO, который сначала устанавливает long-polling соединение, а затем пытается улучшить его до  WebSocket. 

### Поддержка автоматического переподключения

Если не указано иное, отключенный клиент будет пытаться восстановить соединение до тех пор, пока сервер снова не станет доступен. 

### Обнаружение разъединения

Механизм heartbeat ([вики](https://ru.wikipedia.org/wiki/Heartbeat-%D1%81%D0%BE%D0%BE%D0%B1%D1%89%D0%B5%D0%BD%D0%B8%D0%B5): «сердцебиение» — это периодический сигнал, генерируемый аппаратным или программным обеспечением для индикации нормальной работы или для синхронизации других частей компьютерной системы) реализован на уровне EngineIO, что позволяет как серверу, так и клиенту знать, когда собеседник больше не отвечает.

Эта функциональность достигается с помощью таймеров, установленных как на сервере, так и на клиенте, со значениями тайм-аута (параметры *pingInterval* и *pingTimeout*), которые используются совместно при установлении соединения. Эти таймеры требуют, чтобы любые последующие клиентские вызовы направлялись на один и тот же сервер, поэтому при использовании нескольких узлов (нод) требуется механизм ***sticky session.***

**Sticky session** — метод балансировки нагрузки, при котором запросы клиента передаются на один и тот же сервер группы.
### Поддержка бинарных типов данных

Любые сериализуемые структуры данных могут быть переданы, в том числе:

 - [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) и [Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob) в браузере 
 - [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) и [Buffer](https://nodejs.org/api/buffer.html) в Node.js

***Сериализация*** — процесс перевода какой-либо структуры данных в последовательность битов.

### Поддержка мультиплексирования

Чтобы создать разделение групп в вашем приложении (например, для каждого модуля или на основе разрешений), SocketIO позволяет вам создать несколько пространств имен, которые будут действовать как отдельные каналы связи, но будут использовать одно и то же базовое соединение.

### Поддержка комнат (room)

В каждом пространстве имен вы можете определить произвольные каналы, называемые комнатами, к которым сокеты могут присоединяться и отсоединяться. Затем вы можете транслировать в любую комнату, достигнув каждого сокета, который присоединился к ней.

Это полезная функция для отправки уведомлений группе пользователей или данному пользователю, например, подключенному к нескольким устройствам.
Эти функции поставляются с простым и удобным API, который выглядит следующим образом:
```js
io.on('connection', function(socket){
  socket.emit('request', /* */); // отправить событие в сокет
  io.emit('broadcast', /* */); // отправить событие на все подключенные сокеты
  socket.on('reply', function(){ /* */ }); // слушать событие
});
```
## Чем SocketIO не является

SocketIO **НЕ** является реализацией WebSocket. Хотя SocketIO действительно использует WebSocket в качестве протокола передачи данных, когда это возможно, он добавляет некоторые метаданные к каждому пакету: тип пакета, пространство имен и идентификатор ACK, когда требуется подтверждение сообщения. Вот почему клиент WebSocket не сможет успешно подключиться к серверу SocketIO, а клиент SocketIO также не сможет подключиться к серверу WebSocket. 

```js
// ВНИМАНИЕ: клиент НЕ сможет подключиться!
const client = io ('ws: //echo.websocket.org');
```


## Установка

### Серверная часть

```
npm install --save socket.io
```

[Исходный код](https://github.com/socketio/socket.io)

### Javascript клиент

Автономная сборка клиента предоставляется по умолчанию сервером по адресу `/socket.io/socket.io.js`.

Также это может быть сделано из CDN, через [cdnjs](https://cdnjs.com/libraries/socket.io).

Для использования c Node.js, или с помощью сборщика, например, [webpack](https://webpack.js.org/) или [browserify](http://browserify.org/), вы также можете установить пакетную сборку из npm:

```
npm install --save socket.io-client
```

[Исходный код](https://github.com/socketio/socket.io-client)

### Реализации клиента на других языках программирования

Существует несколько реализаций клиента на других языках, которые поддерживаются сообществом:

- Java: https://github.com/socketio/socket.io-client-java
- C++: https://github.com/socketio/socket.io-client-cpp
- Swift: https://github.com/socketio/socket.io-client-swift
- Dart: https://github.com/rikulo/socket.io-client-dart
- Python: https://github.com/miguelgrinberg/python-socketio
- .Net: https://github.com/Quobject/SocketIoClientDotNet

## Использование с http-сервером Node

### Сервер (app.js)

```js
var app = require('http').createServer(handler)
var io = require('socket.io')(app);
var fs = require('fs');

app.listen(80);

function handler (req, res) {
  fs.readFile(__dirname + '/index.html',
  function (err, data) {
    if (err) {
      res.writeHead(500);
      return res.end('Error loading index.html');
    }

    res.writeHead(200);
    res.end(data);
  });
}

io.on('connection', function (socket) {
  socket.emit('news', { hello: 'world' });
  socket.on('my other event', function (data) {
    console.log(data);
  });
});
```

### Клиент (index.html)

```html
<script src="/socket.io/socket.io.js"></script>
<script>
  var socket = io('http://localhost');
  socket.on('news', function (data) {
    console.log(data);
    socket.emit('my other event', { my: 'data' });
  });
</script>
```

## Использование с Express

### Сервер (app.js)

```js
var app = require('express')();
var server = require('http').Server(app);
var io = require('socket.io')(server);

server.listen(80);
// WARNING: app.listen(80) will NOT work here!

app.get('/', function (req, res) {
  res.sendFile(__dirname + '/index.html');
});

io.on('connection', function (socket) {
  socket.emit('news', { hello: 'world' });
  socket.on('my other event', function (data) {
    console.log(data);
  });
});
```

### Клиент (index.html)

```html
<script src="/socket.io/socket.io.js"></script>
<script>
  var socket = io.connect('http://localhost');
  socket.on('news', function (data) {
    console.log(data);
    socket.emit('my other event', { my: 'data' });
  });
</script>
```

## Отправка и получение событий

Socket.IO позволяет отправлять и получать пользовательские события. Кроме `connect`, `message` и `disconnect`, вы можете создавать свои события:

### Сервер

```js
// note, io(<port>) will create a http server for you
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  io.emit('this', { will: 'be received by everyone'});

  socket.on('private message', function (from, msg) {
    console.log('I received a private message by ', from, ' saying ', msg);
  });

  socket.on('disconnect', function () {
    io.emit('user disconnected');
  });
});
```

## Ограничение пространством имен

Если у вас есть контроль над всеми сообщениями и событиями, генерируемыми и отправляемыми для конкретного приложения, работает пространство имен по умолчанию. Если вы хотите использовать сторонний код или создавать код для совместного использования с другими, socket.io предоставляет способ создания пространства имен для сокета.

Это дает возможность `мультиплексировать` одно соединение. Вместо использования двух соединений типа `WebSocket`, SocketIO использует одно.

### Сервер (app.js)

```js
var io = require('socket.io')(80);
var chat = io
  .of('/chat')
  .on('connection', function (socket) {
    socket.emit('a message', {
        that: 'only'
      , '/chat': 'will get'
    });
    chat.emit('a message', {
        everyone: 'in'
      , '/chat': 'will get'
    });
  });

var news = io
  .of('/news')
  .on('connection', function (socket) {
    socket.emit('item', { news: 'item' });
  });
```

### Клиент (index.html)

```html
<script>
  var chat = io.connect('http://localhost/chat')
    , news = io.connect('http://localhost/news');
  
  chat.on('connect', function () {
    chat.emit('hi!');
  });
  
  news.on('news', function () {
    news.emit('woot');
  });
</script>
```

## Отправка нестабильных сообщений

Иногда определенные сообщения могут быть сброшены. Допустим, у вас есть приложение, которое показывает твиты в реальном времени по ключевому слову `bieber`.

Если определенный клиент не готов к приему сообщений (из-за медленной работы сети, других проблем, или из-за того, что он подключен через long polling и находится в середине цикла запрос-ответ), если он не получает ВСЕ твиты,связанные с `bieber`,то ваше приложение не пострадает

В этом случае вы можете отправить эти сообщения как нестабильные(изменчивые) сообщения.

### Сервер

```js
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  var tweets = setInterval(function () {
    getBieberTweet(function (tweet) {
      socket.volatile.emit('bieber tweet', tweet);
    });
  }, 100);

  socket.on('disconnect', function () {
    clearInterval(tweets);
  });
});
```

## Отправка и получение данных (подтверждений)

Иногда может потребоваться _callback_, когда клиент подтвердил получение сообщения

Для этого просто передайте функцию в качестве последнего параметра `.send` или` .emit`. Более того, когда вы используете `.emit`, подтверждение делается вами, что означает, что вы также можете передавать данные:

### Сервер (app.js)

```js
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  socket.on('ferret', function (name, word, fn) {
    fn(name + ' says ' + word);
  });
});
```

### Клиент (index.html)

```html
<script>
  var socket = io(); // СОВЕТ: io() без аргументов производит автообнаружение
  socket.on('connect', function () { // СОВЕТ: вы можете избежать прослушивания через `connect` и прослушивать события напрямую!
    socket.emit('ferret', 'tobi', 'woot', function (data) { // аргументы отправляются для подтверждения функции
      console.log(data); // данные 'tobi says woot'
    });
  });
</script>
```

## Широковещательные сообщения

Чтобы отправить широковещательное сообщение (broadcasting), просто добавьте флаг `broadcast` в вызовы методов `emit` и `send`.

***Broadcasting*** - отправка сообщения всем, кроме сокета, который его запускает.

### Server

```js
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  socket.broadcast.emit('user connected');
});
```

## Использование в качестве кроссбраузерного WebSocket

Если вам нужно понимание WebSocket, то используйте `send` и прослушайте событие `message`:

### Сервер (app.js)

```js
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  socket.on('message', function () { });
  socket.on('disconnect', function () { });
});
```

### Клиент (index.html)

```html
<script>
  var socket = io('http://localhost/');
  socket.on('connect', function () {
    socket.send('hi');

    socket.on('message', function (msg) {
      // моё сообщение
    });
  });
</script>
```

Если вам не важна логика переподключения и тому подобное, посмотрите <a href="https://github.com/socketio/engine.io">Engine.IO</a>,  который использует Socket.IO и является транспортным уровнем WebSocket, .
