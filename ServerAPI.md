## 服务器端API

### 服务器端

首先使用```require('socket.io')```引入Socket.IO。

### Server()

创建一个服务器，既可以使用```new```也可以不使用```new```：

```javascript
var io = require('socket.io')();
// or
var Server = require('socket.io');
var io = new Server();
```
### Server(opts:Object)

这是可选的，构造器的第一个或第二个参数(参照下一条)可以是一个用于设置的对象。

以下是支持的设置：

- serveClient 设置```Server#serveClient()```的值
- path 设置```Server#path()```的值

传递给Socket.IO的设置同时也会被传递给创建的Engine.IO。关于此处，请参见Engine.IO。

### Server(srv:http#Server, opts:Object)

新建一个服务器并将其附加至给出的```srv```，第二个参数是可选的，并将被传递。

### Server(port:Number, opts:Object)
将Socket.IO服务器绑定至一个新的```http.Server```，并监听指定端口

### Server#serveClient(v:Boolean):Server

> 此处翻译可能存在问题，欢迎参与修改。[原文链接](http://socket.io/docs/server-api/#server#serveclient(v:boolean):server)

如果```v```为```true```，绑定的服务器(参考 ```Server#attach```)将为客户端提供文件服务。默认为```true```。

当```attach```被调用后，这个方法将不起作用。

```javascript
// 你可以同时传递这两个参数 ...
var io = require('socket.io')(http, { serveClient: false });

// ... 也可以分开进行
var io = require('socket.io')();
io.serveClient(false);
io.attach(http);
```
如果没有提供参数，这个方法将返回当前值。

#Server#path(v:String):Server
Sets the path v under which engine.io and the static files will be
served. Defaults to /socket.io.

If no arguments are supplied this method returns the current value.

#Server#adapter(v:Adapter):Server
Sets the adapter v. Defaults to an instance of the Adapter that
ships with socket.io which is memory based. See
socket.io-adapter.

If no arguments are supplied this method returns the current value.

#Server#origins(v:String):Server
Sets the allowed origins v. Defaults to any origins being allowed.

If no arguments are supplied this method returns the current value.

#Server#sockets:Namespace
The default (/) namespace.

#Server#attach(srv:http#Server, opts:Object):Server
Attaches the Server to an engine.io instance on srv with the
supplied opts (optionally).

#Server#attach(port:Number, opts:Object):Server
Attaches the Server to an engine.io instance that is bound to port
with the given opts (optionally).

#Server#listen
Synonym of Server#attach.

#Server#bind(srv:engine#Server):Server
Advanced use only. Binds the server to a specific engine.io Server
(or compatible API) instance.

#Server#onconnection(socket:engine#Socket):Server
Advanced use only. Creates a new socket.io client from the incoming
engine.io (or compatible API) socket.

#Server#of(nsp:String):Namespace
Initializes and retrieves the given Namespace by its pathname
identifier nsp.

If the namespace was already initialized it returns it right away.

#Server#emit
Emits an event to all connected clients. The following two are
equivalent:

var io = require('socket.io')();
io.sockets.emit('an event sent to all connected clients');
io.emit('an event sent to all connected clients');
For other available methods, see Namespace below.

#Server#use
See Namespace#use below.

#Namespace
Represents a pool of sockets connected under a given scope identified
by a pathname (eg: /chat).

By default the client always connects to /.

Events
connection / connect. Fired upon a connection.
Parameters:

Socket the incoming socket.
#Namespace#name:String
The namespace identifier property.

#Namespace#connected:Object
Hash of Socket objects that are connected to this namespace indexed
by id.

#Namespace#use(fn:Function):Namespace
Registers a middleware, which is a function that gets executed for
every incoming Socket and receives as parameter the socket and a
function to optionally defer execution to the next registered
middleware.

var io = require('socket.io')();
io.use(function(socket, next){
  if (socket.request.headers.cookie) return next();
  next(new Error('Authentication error'));
});
Errors passed to middleware callbacks are sent as special error
packets to clients.

#Socket
A Socket is the fundamental class for interacting with browser
clients. A Socket belongs to a certain Namespace (by default /)
and uses an underlying Client to communicate.

#Socket#rooms:Array
A list of strings identifying the rooms this socket is in.

#Socket#client:Client
A reference to the underlying Client object.

#Socket#conn:Socket
A reference to the underyling Client transport connection (engine.io
Socket object).

#Socket#request:Request
A getter proxy that returns the reference to the request that
originated the underlying engine.io Client. Useful for accessing
request headers such as Cookie or User-Agent.

#Socket#id:String
A unique identifier for the socket session, that comes from the
underlying Client.

#Socket#emit(name:String[, …]):Socket
Emits an event to the socket identified by the string name. Any
other parameters can be included.

All datastructures are supported, including Buffer. JavaScript
functions can’t be serialized/deserialized.

var io = require('socket.io')();
io.on('connection', function(socket){
  socket.emit('an event', { some: 'data' });
});
#Socket#join(name:String[, fn:Function]):Socket
Adds the socket to the room, and fires optionally a callback fn
with err signature (if any).

The socket is automatically a member of a room identified with its
session id (see Socket#id).

The mechanics of joining rooms are handled by the Adapter
that has been configured (see Server#adapter above), defaulting to
socket.io-adapter.

#Socket#leave(name:String[, fn:Function]):Socket
Removes the socket from room, and fires optionally a callback fn
with err signature (if any).

Rooms are left automatically upon disconnection.

The mechanics of leaving rooms are handled by the Adapter
that has been configured (see Server#adapter above), defaulting to
socket.io-adapter.

#Socket#to(room:String):Socket
#Socket#in(room:String):Socket
Sets a modifier for a subsequent event emission that the event will
only be broadcasted to sockets that have joined the given room.

To emit to multiple rooms, you can call to several times.

var io = require('socket.io')();
io.on('connection', function(socket){
  socket.to('others').emit('an event', { some: 'data' });
});
#Client
The Client class represents an incoming transport (engine.io)
connection. A Client can be associated with many multiplexed Socket
that belong to different Namespaces.

#Client#conn
A reference to the underlying engine.io Socket connection.

#Client#request
A getter proxy that returns the reference to the request that
originated the engine.io connection. Useful for accessing
request headers such as Cookie or User-Agent.
