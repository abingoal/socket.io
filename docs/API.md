## 目录

* [Class: Server](#server)
  * [new Server(httpServer[, options])](#new-serverhttpserver-options)
  * [new Server(port[, options])](#new-serverport-options)
  * [new Server(options)](#new-serveroptions)
  * [server.sockets](#serversockets)
  * [server.engine.generateId](#serverenginegenerateid)
  * [server.serveClient([value])](#serverserveclientvalue)
  * [server.path([value])](#serverpathvalue)
  * [server.adapter([value])](#serveradaptervalue)
  * [server.origins([value])](#serveroriginsvalue)
  * [server.origins(fn)](#serveroriginsfn)
  * [server.attach(httpServer[, options])](#serverattachhttpserver-options)
  * [server.attach(port[, options])](#serverattachport-options)
  * [server.listen(httpServer[, options])](#serverlistenhttpserver-options)
  * [server.listen(port[, options])](#serverlistenport-options)
  * [server.bind(engine)](#serverbindengine)
  * [server.onconnection(socket)](#serveronconnectionsocket)
  * [server.of(nsp)](#serverofnsp)
  * [server.close([callback])](#serverclosecallback)
* [Class: Namespace](#namespace)
  * [namespace.name](#namespacename)
  * [namespace.connected](#namespaceconnected)
  * [namespace.adapter](#namespaceadapter)
  * [namespace.to(room)](#namespacetoroom)
  * [namespace.in(room)](#namespaceinroom)
  * [namespace.emit(eventName[, ...args])](#namespaceemiteventname-args)
  * [namespace.clients(callback)](#namespaceclientscallback)
  * [namespace.use(fn)](#namespaceusefn)
  * [Event: 'connect'](#event-connect)
  * [Event: 'connection'](#event-connect)
  * [Flag: 'volatile'](#flag-volatile)
  * [Flag: 'local'](#flag-local)
* [Class: Socket](#socket)
  * [socket.id](#socketid)
  * [socket.rooms](#socketrooms)
  * [socket.client](#socketclient)
  * [socket.conn](#socketconn)
  * [socket.request](#socketrequest)
  * [socket.handshake](#sockethandshake)
  * [socket.use(fn)](#socketusefn)
  * [socket.send([...args][, ack])](#socketsendargs-ack)
  * [socket.emit(eventName[, ...args][, ack])](#socketemiteventname-args-ack)
  * [socket.on(eventName, callback)](#socketoneventname-callback)
  * [socket.once(eventName, listener)](#socketonceeventname-listener)
  * [socket.removeListener(eventName, listener)](#socketremovelistenereventname-listener)
  * [socket.removeAllListeners([eventName])](#socketremovealllistenerseventname)
  * [socket.eventNames()](#socketeventnames)
  * [socket.join(room[, callback])](#socketjoinroom-callback)
  * [socket.join(rooms[, callback])](#socketjoinrooms-callback)
  * [socket.leave(room[, callback])](#socketleaveroom-callback)
  * [socket.to(room)](#sockettoroom)
  * [socket.in(room)](#socketinroom)
  * [socket.compress(value)](#socketcompressvalue)
  * [socket.disconnect(close)](#socketdisconnectclose)
  * [Flag: 'broadcast'](#flag-broadcast)
  * [Flag: 'volatile'](#flag-volatile-1)
  * [Event: 'disconnect'](#event-disconnect)
  * [Event: 'error'](#event-error)
  * [Event: 'disconnecting'](#event-disconnecting)
* [Class: Client](#client)
  * [client.conn](#clientconn)
  * [client.request](#clientrequest)

### Server

使用方式： `require('socket.io')`

#### new Server(httpServer[, options])

* `httpServer` _(http.Server)_ 要绑定的服务器
* `options` _(Object)_
  * `path` _(String)_: 要获取的路径 (`/socket.io`)
  * `serveClient` _(Boolean)_: 是否提供客户端文件 (`true`)
  * `adapter` _(Adapter)_: 使用适配器。默认为基于内存的 socket.io 附带的 Adapter
    实例。请参
    阅[socket.io-adapter](https://github.com/socketio/socket.io-adapter)
  * `origins` _(String)_: 允许来源 (`*`)
  * `parser` _(Parser)_: 使用解析器。默认为 socket.io 附带的`Parser`实例。请参
    阅[socket.io-parser](https://github.com/socketio/socket.io-parser).

初始化的两种方式 :

```js
const io = require("socket.io")();
// 或者
const Server = require("socket.io");
const io = new Server();
```

传递给`socket.io`的选项也会传递给`engine.io` Server。请参
阅`engine.io`[选项](https://github.com/socketio/engine.io#methods-1)。

以下选项 :

* `pingTimeout` _(Number)_: 多少毫秒没有 pong packet 则连接关闭（60000)
* `pingInterval` _(Number)_: 在发送一个新的 ping 数据包之前有多少 ms（25000)

设置在服务端断开多久后的延迟断开。

例如由于底层 TCP 连接由于网络问题的异常断开，客户端会等带`pingTimeout +
pingInterval`毫秒之后执行`disconnect`事件。

* `transports` _(Array<String>)_: 允许连接的方式 (`['polling', 'websocket']`).

**注意 :** 顺序很重要。默认情况下会首先建立一个 long-polling 连接，如果
WebSocket 可用的话则升级为 WebSocket 连接。使用`['websocket']`就意味着如果一个
WebSocket 连接断开后将无法再打开。

```js
const server = require("http").createServer();

const io = require("socket.io")(server, {
  path: "/test",
  serveClient: false,
  // 下面是engine.IO的选项
  pingInterval: 10000,
  pingTimeout: 5000,
  cookie: false
});

server.listen(3000);
```

#### new Server(port[, options])

* `port` _(Number)_ 监听端口 ( 会创建一个新的`http.Server`)
* `options` _(Object)_

[查看](#new-serverhttpserver-options) 可用选项

```js
const server = require("http").createServer();

const io = require("socket.io")(3000, {
  path: "/test",
  serveClient: false,
  // 下面是engine.IO选项
  pingInterval: 10000,
  pingTimeout: 5000,
  cookie: false
});
```

#### new Server(options)

* `options` _(Object)_

[查看](#new-serverhttpserver-options) 可用选项

```js
const io = require("socket.io")({
  path: "/test",
  serveClient: false
});

// 或者
const server = require("http").createServer();

io.attach(server, {
  pingInterval: 10000,
  pingTimeout: 5000,
  cookie: false
});

server.listen(3000);

// 或者
io.attach(3000, {
  pingInterval: 10000,
  pingTimeout: 5000,
  cookie: false
});
```

#### server.sockets

* _(Namespace)_

默认命名空间 (`/`)

#### server.serveClient([value])

* `value` _(Boolean)_
* **Returns** `Server|Boolean`

如果`value` 是 `true，则连接的服务器（请参阅 Server＃attach ）将为客户端文件提供
服务。这个方法在 attach 后调用没有效果。如果没有参数提供，这个方法返回当前值。

```js
// 传递一个server和`serveClient`选项
const io = require("socket.io")(http, { serveClient: false });

// 或者不传递server,直接调用该方法
const io = require("socket.io")();
io.serveClient(false);
io.attach(http);
```

#### server.path([value])

* `value` _(String)_
* **Returns** `Server|String`

设置给`engine.io`和静态文件提供的路径值。默认为`/socket.io`。如果没有参数提供的
话，将会返回当前值。

```js
const io = require("socket.io")();
io.path("/myownpath");

// 客户端
const socket = io({
  path: "/myownpath"
});
```

#### server.adapter([value])

* `value` _(Adapter)_
* **Returns** `Server|Adapter` 设置适配器值。默认为基于内存的 socket.io 附带的
  Adapter 实例。请参
  阅[socket.io-adapter](https://github.com/socketio/socket.io-adapter)。如果没有
  提供参数则返回当前值。

```js
const io = require("socket.io")(3000);
const redis = require("socket.io-redis");
io.adapter(redis({ host: "localhost", port: 6379 }));
```

#### server.origins([value])

* `value` _(String)_
* **Returns** `Server|String`

设置允许来源。默认允许任何来源。如果没有提供参数则返回当前值。

```js
io.origins(["foo.example.com:443"]);
```

#### server.origins(fn)

* `fn` _(Function)_
* **Returns** `Server`

提供一个带有两个参数的函数`origin：String 和 callback（error ， success）`，其
中`success`是一个布尔值，指示是否允许来源。

**潜在的缺点**:

* 在某些情况下，如果不确定`origin`, 则有可能是允许来源`*`。
* 由于每个请求都会执行这个函数，因此建议尽可能保证该函数效率最优。
* 如果`socket.io`和`Express`一起使用，CORS 头文件只会受到 socket.io 请求的影响
  。Express 的话请使用[cors](https://github.com/expressjs/cors)。

```js
io.origins((origin, callback) => {
  if (origin !== "https://foo.example.com") {
    return callback("origin not allowed", false);
  }
  callback(null, true);
});
```

#### server.attach(httpServer[, options])

* `httpServer` _(http.Server)_ 要附加的服务器
* `options` _(Object)_

将 `Server`连接到`httpServer`上的 engine.io 实例选项 ( 可选 )

#### server.attach(port[, options])

* `port` _(Number)_ 监听的端口
* `options` _(Object)_

将`Server`连接到新的 http.Server 上的 engine.io 实例提供选项（可选）。

#### server.listen(httpServer[, options])

类似于 [server.attach(httpServer[, options])](#serverattachhttpserver-options).

#### server.listen(port[, options])

类似于 [server.attach(port[, options])](#serverattachport-options).

#### server.bind(engine)

* `engine` _(engine.Server)_
* **Returns** `Server`

Advanced use only. Binds the server to a specific engine.io `Server` (or
compatible API) instance.

#### server.onconnection(socket)

* `socket` _(engine.Socket)_
* **Returns** `Server`

Advanced use only. Creates a new `socket.io` client from the incoming engine.io
(or compatible API) `Socket`.

#### server.of(nsp)

* `nsp` _(String)_
* **Returns** `Namespace`

Initializes and retrieves the given `Namespace` by its pathname identifier
`nsp`. If the namespace was already initialized it returns it immediately.

```js
const adminNamespace = io.of("/admin");
```

#### server.close([callback])

* `callback` _(Function)_

Closes the socket.io server. The `callback` argument is optional and will be
called when all connections are closed.

```js
const Server = require("socket.io");
const PORT = 3030;
const server = require("http").Server();

const io = Server(PORT);

io.close(); // Close current server

server.listen(PORT); // PORT is free to use

io = Server(server);
```

#### server.engine.generateId

Overwrites the default method to generate your custom socket id.

The function is called with a node request object (`http.IncomingMessage`) as
first parameter.

```js
io.engine.generateId = req => {
  return "custom:id:" + custom_id++; // custom id must be unique
};
```

### Namespace

Represents a pool of sockets connected under a given scope identified by a
pathname (eg: `/chat`).

A client always connects to `/` (the main namespace), then potentially connect
to other namespaces (while using the same underlying connection).

#### namespace.name

* _(String)_

The namespace identifier property.

#### namespace.connected

* _(Object<Socket>)_

The hash of `Socket` objects that are connected to this namespace, indexed by
`id`.

#### namespace.adapter

* _(Adapter)_

The `Adapter` used for the namespace. Useful when using the `Adapter` based on
[Redis](https://github.com/socketio/socket.io-redis), as it exposes methods to
manage sockets and rooms accross your cluster.

**Note:** the adapter of the main namespace can be accessed with
`io.of('/').adapter`.

#### namespace.to(room)

* `room` _(String)_
* **Returns** `Namespace` for chaining

Sets a modifier for a subsequent event emission that the event will only be
_broadcasted_ to clients that have joined the given `room`.

To emit to multiple rooms, you can call `to` several times.

```js
const io = require("socket.io")();
const adminNamespace = io.of("/admin");

adminNamespace.to("level1").emit("an event", { some: "data" });
```

#### namespace.in(room)

Synonym of [namespace.to(room)](#namespacetoroom).

#### namespace.emit(eventName[, ...args])

* `eventName` _(String)_
* `args`

Emits an event to all connected clients. The following two are equivalent:

```js
const io = require("socket.io")();
io.emit("an event sent to all connected clients"); // main namespace

const chat = io.of("/chat");
chat.emit("an event sent to all connected clients in chat namespace");
```

**Note:** acknowledgements are not supported when emitting from namespace.

#### namespace.clients(callback)

* `callback` _(Function)_

Gets a list of client IDs connected to this namespace (across all nodes if
applicable).

```js
const io = require("socket.io")();
io.of("/chat").clients((error, clients) => {
  if (error) throw error;
  console.log(clients); // => [PZDoMHjiu8PYfRiKAAAF, Anw2LatarvGVVXEIAAAD]
});
```

An example to get all clients in namespace's room:

```js
io
  .of("/chat")
  .in("general")
  .clients((error, clients) => {
    if (error) throw error;
    console.log(clients); // => [Anw2LatarvGVVXEIAAAD]
  });
```

As with broadcasting, the default is all clients from the default namespace
('/'):

```js
io.clients((error, clients) => {
  if (error) throw error;
  console.log(clients); // => [6em3d4TJP8Et9EMNAAAA, G5p55dHhGgUnLUctAAAB]
});
```

#### namespace.use(fn)

* `fn` _(Function)_

Registers a middleware, which is a function that gets executed for every
incoming `Socket`, and receives as parameters the socket and a function to
optionally defer execution to the next registered middleware.

Errors passed to middleware callbacks are sent as special `error` packets to
clients.

```js
io.use((socket, next) => {
  if (socket.request.headers.cookie) return next();
  next(new Error("Authentication error"));
});
```

#### Event: 'connect'

* `socket` _(Socket)_ socket connection with client

Fired upon a connection from client.

```js
io.on("connect", socket => {
  // ...
});

io.of("/admin").on("connect", socket => {
  // ...
});
```

#### Event: 'connection'

Synonym of [Event: 'connect'](#event-connect).

#### Flag: 'volatile'

Sets a modifier for a subsequent event emission that the event data may be lost
if the clients are not ready to receive messages (because of network slowness or
other issues, or because they’re connected through long polling and is in the
middle of a request-response cycle).

```js
io.volatile.emit("an event", { some: "data" }); // the clients may or may not receive it
```

#### Flag: 'local'

Sets a modifier for a subsequent event emission that the event data will only be
_broadcast_ to the current node (when the
[Redis adapter](https://github.com/socketio/socket.io-redis) is used).

```js
io.local.emit("an event", { some: "data" });
```

### Socket

A `Socket` is the fundamental class for interacting with browser clients. A
`Socket` belongs to a certain `Namespace` (by default `/`) and uses an
underlying `Client` to communicate.

It should be noted the `Socket` doesn't relate directly to the actual underlying
TCP/IP `socket` and it is only the name of the class.

Within each `Namespace`, you can also define arbitrary channels (called `room`)
that the `Socket` can join and leave. That provides a convenient way to
broadcast to a group of `Socket`s (see `Socket#to` below).

The `Socket` class inherits from
[EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter).
The `Socket` class overrides the `emit` method, and does not modify any other
`EventEmitter` method. All methods documented here which also appear as
`EventEmitter` methods (apart from `emit`) are implemented by `EventEmitter`,
and documentation for `EventEmitter` applies.

#### socket.id

* _(String)_

A unique identifier for the session, that comes from the underlying `Client`.

#### socket.rooms

* _(Object)_

A hash of strings identifying the rooms this client is in, indexed by room name.

```js
io.on("connection", socket => {
  socket.join("room 237", () => {
    let rooms = Object.keys(socket.rooms);
    console.log(rooms); // [ <socket.id>, 'room 237' ]
  });
});
```

#### socket.client

* _(Client)_

A reference to the underlying `Client` object.

#### socket.conn

* _(engine.Socket)_

A reference to the underlying `Client` transport connection (engine.io `Socket`
object). This allows access to the IO transport layer, which still (mostly)
abstracts the actual TCP/IP socket.

#### socket.request

* _(Request)_

A getter proxy that returns the reference to the `request` that originated the
underlying engine.io `Client`. Useful for accessing request headers such as
`Cookie` or `User-Agent`.

#### socket.handshake

* _(Object)_

The handshake details:

```js
{
  headers: /* the headers sent as part of the handshake */,
  time: /* the date of creation (as string) */,
  address: /* the ip of the client */,
  xdomain: /* whether the connection is cross-domain */,
  secure: /* whether the connection is secure */,
  issued: /* the date of creation (as unix timestamp) */,
  url: /* the request URL string */,
  query: /* the query object */
}
```

Usage:

```js
io.use((socket, next) => {
  let handshake = socket.handshake;
  // ...
});

io.on("connection", socket => {
  let handshake = socket.handshake;
  // ...
});
```

#### socket.use(fn)

* `fn` _(Function)_

Registers a middleware, which is a function that gets executed for every
incoming `Packet` and receives as parameter the packet and a function to
optionally defer execution to the next registered middleware.

Errors passed to middleware callbacks are sent as special `error` packets to
clients.

```js
io.on("connection", socket => {
  socket.use((packet, next) => {
    if (packet.doge === true) return next();
    next(new Error("Not a doge error"));
  });
});
```

#### socket.send([...args][, ack])

* `args`
* `ack` _(Function)_
* **Returns** `Socket`

Sends a `message` event. See
[socket.emit(eventName[, ...args][, ack])](#socketemiteventname-args-ack).

#### socket.emit(eventName[, ...args][, ack])

_(overrides `EventEmitter.emit`)_

* `eventName` _(String)_
* `args`
* `ack` _(Function)_
* **Returns** `Socket`

Emits an event to the socket identified by the string name. Any other parameters
can be included. All serializable datastructures are supported, including
`Buffer`.

```js
socket.emit("hello", "world");
socket.emit("with-binary", 1, "2", { 3: "4", 5: new Buffer(6) });
```

The `ack` argument is optional and will be called with the client's answer.

```js
io.on("connection", socket => {
  socket.emit("an event", { some: "data" });

  socket.emit("ferret", "tobi", data => {
    console.log(data); // data will be 'woot'
  });

  // the client code
  // client.on('ferret', (name, fn) => {
  //   fn('woot');
  // });
});
```

#### socket.on(eventName, callback)

_(inherited from `EventEmitter`)_

* `eventName` _(String)_
* `callback` _(Function)_
* **Returns** `Socket`

Register a new handler for the given event.

```js
socket.on("news", data => {
  console.log(data);
});
// with several arguments
socket.on("news", (arg1, arg2, arg3) => {
  // ...
});
// or with acknowledgement
socket.on("news", (data, callback) => {
  callback(0);
});
```

#### socket.once(eventName, listener)

#### socket.removeListener(eventName, listener)

#### socket.removeAllListeners([eventName])

#### socket.eventNames()

Inherited from `EventEmitter` (along with other methods not mentioned here). See
Node.js documentation for the `events` module.

#### socket.join(room[, callback])

* `room` _(String)_
* `callback` _(Function)_
* **Returns** `Socket` for chaining

Adds the client to the `room`, and fires optionally a callback with `err`
signature (if any).

```js
io.on("connection", socket => {
  socket.join("room 237", () => {
    let rooms = Object.keys(socket.rooms);
    console.log(rooms); // [ <socket.id>, 'room 237' ]
    io.to("room 237").emit("a new user has joined the room"); // broadcast to everyone in the room
  });
});
```

The mechanics of joining rooms are handled by the `Adapter` that has been
configured (see `Server#adapter` above), defaulting to
[socket.io-adapter](https://github.com/socketio/socket.io-adapter).

For your convenience, each socket automatically joins a room identified by its
id (see `Socket#id`). This makes it easy to broadcast messages to other sockets:

```js
io.on("connection", socket => {
  socket.on("say to someone", (id, msg) => {
    // send a private message to the socket with the given id
    socket.to(id).emit("my message", msg);
  });
});
```

#### socket.join(rooms[, callback])

* `rooms` _(Array)_
* `callback` _(Function)_
* **Returns** `Socket` for chaining

Adds the client to the list of room, and fires optionally a callback with `err`
signature (if any).

#### socket.leave(room[, callback])

* `room` _(String)_
* `callback` _(Function)_
* **Returns** `Socket` for chaining

Removes the client from `room`, and fires optionally a callback with `err`
signature (if any).

**Rooms are left automatically upon disconnection**.

#### socket.to(room)

* `room` _(String)_
* **Returns** `Socket` for chaining

Sets a modifier for a subsequent event emission that the event will only be
_broadcasted_ to clients that have joined the given `room` (the socket itself
being excluded).

To emit to multiple rooms, you can call `to` several times.

```js
io.on("connection", socket => {
  // to one room
  socket.to("others").emit("an event", { some: "data" });
  // to multiple rooms
  socket
    .to("room1")
    .to("room2")
    .emit("hello");
  // a private message to another socket
  socket.to(/* another socket id */).emit("hey");
});
```

**Note:** acknowledgements are not supported when broadcasting.

#### socket.in(room)

Synonym of [socket.to(room)](#sockettoroom).

#### socket.compress(value)

* `value` _(Boolean)_ whether to following packet will be compressed
* **Returns** `Socket` for chaining

Sets a modifier for a subsequent event emission that the event data will only be
_compressed_ if the value is `true`. Defaults to `true` when you don't call the
method.

```js
io.on("connection", socket => {
  socket.compress(false).emit("uncompressed", "that's rough");
});
```

#### socket.disconnect(close)

* `close` _(Boolean)_ whether to close the underlying connection
* **Returns** `Socket`

Disconnects this client. If value of close is `true`, closes the underlying
connection. Otherwise, it just disconnects the namespace.

```js
io.on("connection", socket => {
  setTimeout(() => socket.disconnect(true), 5000);
});
```

#### Flag: 'broadcast'

Sets a modifier for a subsequent event emission that the event data will only be
_broadcast_ to every sockets but the sender.

```js
io.on("connection", socket => {
  socket.broadcast.emit("an event", { some: "data" }); // everyone gets it but the sender
});
```

#### Flag: 'volatile'

Sets a modifier for a subsequent event emission that the event data may be lost
if the client is not ready to receive messages (because of network slowness or
other issues, or because they’re connected through long polling and is in the
middle of a request-response cycle).

```js
io.on("connection", socket => {
  socket.volatile.emit("an event", { some: "data" }); // the client may or may not receive it
});
```

#### Event: 'disconnect'

* `reason` _(String)_ the reason of the disconnection (either client or
  server-side)

Fired upon disconnection.

```js
io.on("connection", socket => {
  socket.on("disconnect", reason => {
    // ...
  });
});
```

#### Event: 'error'

* `error` _(Object)_ error object

Fired when an error occurs.

```js
io.on("connection", socket => {
  socket.on("error", error => {
    // ...
  });
});
```

#### Event: 'disconnecting'

* `reason` _(String)_ the reason of the disconnection (either client or
  server-side)

Fired when the client is going to be disconnected (but hasn't left its `rooms`
yet).

```js
io.on("connection", socket => {
  socket.on("disconnecting", reason => {
    let rooms = Object.keys(socket.rooms);
    // ...
  });
});
```

These are reserved events (along with `connect`, `newListener` and
`removeListener`) which cannot be used as event names.

### Client

The `Client` class represents an incoming transport (engine.io) connection. A
`Client` can be associated with many multiplexed `Socket`s that belong to
different `Namespace`s.

#### client.conn

* _(engine.Socket)_

A reference to the underlying `engine.io` `Socket` connection.

#### client.request

* _(Request)_

A getter proxy that returns the reference to the `request` that originated the
engine.io connection. Useful for accessing request headers such as `Cookie` or
`User-Agent`.
