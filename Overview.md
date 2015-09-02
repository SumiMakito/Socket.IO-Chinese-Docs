## 总览

### 如何使用？
#### 安装
```shell
$ npm install socket.io
```
#### 使用Node服务器
##### 服务器端 (app.js)
```javascript
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

##### 客户端 (index.html)
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
#### 使用Express 3或4
##### 服务器端 (app.js)

```javascript
var app = require('express')();
var server = require('http').Server(app);
var io = require('socket.io')(server);

server.listen(80);

app.get('/', function (req, res) {
  res.sendfile(__dirname + '/index.html');
});

io.on('connection', function (socket) {
  socket.emit('news', { hello: 'world' });
  socket.on('my other event', function (data) {
    console.log(data);
  });
});
```
##### 客户端 (index.html)
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
#### 使用Express框架
##### 服务器端 (app.js)
```javascript
var app = require('express').createServer();
var io = require('socket.io')(app);

app.listen(80);

app.get('/', function (req, res) {
  res.sendfile(__dirname + '/index.html');
});

io.on('connection', function (socket) {
  socket.emit('news', { hello: 'world' });
  socket.on('my other event', function (data) {
    console.log(data);
  });
});
```
##### 客户端 (index.html)
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
#### 发送与接收事件

Socket.IO 允许你发送与接收自定义的事件。在```连接建立```、```信息交换```与```连接断开```时，你都可以推送自定义的事件：

##### 服务器端
```javascript
// 注意，io(<port>)将会创建一个HTTP服务器。
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
#### 使用命名空间来管理不同的Socket

如果你只需要管理一个程序的所有推送消息与事件，使用默认的```/```就能满足你的需求。如果你想要引入第三方的代码，或是想要写一些可以与别人共享的代码，Socket.IO提供了为Socket连接命名的方法。
这种做法能使一条连接被更高效的利用。而不是创建两个甚至多个连接。

##### 服务器端 (app.js)
```javascript
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
##### 客户端 (index.html)
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

#### 发送次要的消息
有时，因为种种原因，某些消息并不能顺利的推送到所有的客户端。来做个假设，想象一下你的程序在实时的显示关于“GitHub”的推文。

当一个客户端并没有准备好接收消息时(由于网络缓慢或其他问题，或因为他们正处于一个长轮询循环的空闲间隔处)，假如这些客户端不能收到全部有关于“GitHub”的推文，对于这个程序来说并不会有什么影响。

在这种情况下，你或许会把这种消息当作次要的消息来发送。

##### 服务器端
```javascript
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  var tweets = setInterval(function () {
    getGitHubTweet(function (tweet) {
      socket.volatile.emit('GitHub tweet', tweet);
    });
  }, 100);

  socket.on('disconnect', function () {
    clearInterval(tweets);
  });
});
```
#### 发送消息与接收反馈
有时，你或许需要客户端在收到并接受消息后向你反馈结果。

为了实现这个回调，简单地在```.send```或```.emit```的最后一个参数中传递一个回调函数就可以了。此外，当你在```.emit```中使用回调时，确认的工作是由你完成的，这意味着你也可以通过这个回调传回数据。

##### 服务器端 (app.js)
```javascript
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  socket.on('ferret', function (name, fn) {
    fn('woot');
  });
});
```
##### 客户端 (index.html)
```html
<script>
  var socket = io(); // TIP: io() with no args does auto-discovery
  socket.on('connect', function () { // TIP: you can avoid listening on `connect` and listen on events directly too!
    socket.emit('ferret', 'tobi', function (data) {
      console.log(data); // data will be 'woot'
    });
  });
</script>
```
#Broadcasting messages
To broadcast, simply add a broadcast flag to emit and send method calls. Broadcasting means sending a message to everyone else except for the socket that starts it.

Server
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  socket.broadcast.emit('user connected');
});
#Using it just as a cross-browser WebSocket
If you just want the WebSocket semantics, you can do that too. Simply leverage send and listen on the message event:

Server (app.js)
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  socket.on('message', function () { });
  socket.on('disconnect', function () { });
});
Client (index.html)
<script>
  var socket = io('http://localhost/');
  socket.on('connect', function () {
    socket.send('hi');

    socket.on('message', function (msg) {
      // my msg
    });
  });
</script>
If you don’t care about reconnection logic and such, take a look at Engine.IO, which is the WebSocket semantics transport layer Socket.IO uses.
