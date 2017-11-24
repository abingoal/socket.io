# 入门 : 聊天应用程序

在这个教程中，我们将创建一个基本聊天应用程序。因为几乎不需要预先了解 NodeJS 或
Socket.IO 的知识点，所以很适合任何新手或者老油条。

## 简介

使用流行的 Web 应用程序技术栈，如 LAMP（PHP ）等编写聊天应用程序非常困难。它涉及
到轮询服务器的变化、时间戳追踪，因此效率比较低下。

传统上 Sockets 一直是大多数实时聊天系统所架构的解决方案，提供客户端与服务器之间
的双向通信信道。

这意味着服务器可以将消息推送到客户端。每当你发送一个聊天消息，首先让服务器接收，
并推送到所有其他连接的客户端。

## Web 框架

首先需要一个简单的 HTML 网页，提供一个表单和一个消息列表。为此我们将使用 Node.JS
Web 框架。

然后创建一个描述我们项目的 package.json 清单文件。我建议你把它放在一个专门的空目
录（我把我的例子命名为 chat-example）。

```json
{
  "name": "socket-chat-example",
  "version": "0.0.1",
  "description": "my first socket.io app",
  "dependencies": {}
}
```

为了方便的管理依赖，我们使用 npm install --save 来安装。

```bash
npm install --save express@4.15.2
```

安装完`express`之后，我们需要创建一个`index.js`文件

```javascript
var app = require("express")();
var http = require("http").Server(app);
app.get("/", function(req, res) {
  res.send("<h1>Hello world</h1>");
});
http.listen(3000, function() {
  console.log("listening on *:3000");
});
```

This translates into the following:

1. Express initializes app to be a function handler that you can supply to an
   HTTP server (as seen in line 2).
2. We define a route handler / that gets called when we hit our website home.
3. We make the http server listen on port 3000. If you run `node index.js` you
   should see the following: ![chat-1](https://socket.io/assets/img/chat-1.png)

And if you point your browser to `http://localhost:3000`:

![chat-2](https://socket.io/assets/img/chat-2.png)

## Serving HTML

So far in `index.js` we’re calling `res.send` and pass it a HTML string. Our
code would look very confusing if we just placed our entire application’s HTML
there. Instead, we’re going to create a `index.html` file and serve it.

Let’s refactor our route handler to use `sendFile` instead:

```javascript
app.get("/", function(req, res) {
  res.sendFile(__dirname + "/index.html");
});
```

And populate index.html with the following:

```html
<!doctype html>
<html>
  <head>
    <title>Socket.IO chat</title>
    <style>
      * { margin: 0; padding: 0; box-sizing: border-box; }
      body { font: 13px Helvetica, Arial; }
      form { background: #000; padding: 3px; position: fixed; bottom: 0; width: 100%; }
      form input { border: 0; padding: 10px; width: 90%; margin-right: .5%; }
      form button { width: 9%; background: rgb(130, 224, 255); border: none; padding: 10px; }
      #messages { list-style-type: none; margin: 0; padding: 0; }
      #messages li { padding: 5px 10px; }
      #messages li:nth-child(odd) { background: #eee; }
    </style>
  </head>
  <body>
    <ul id="messages"></ul>
    <form action="">
      <input id="m" autocomplete="off" /><button>Send</button>
    </form>
  </body>
</html>
```

If you restart the process (by hitting Control+C and running node index again)
and refresh the page it should look like this:
![chat-3](https://socket.io/assets/img/chat-3.png)

## Integrating Socket.IO

Socket.IO is composed of two parts:

* A server that integrates with (or mounts on) the Node.JS HTTP Server:
  `socket.io`
* A client library that loads on the browser side: `socket.io-client`

During development, `socket.io` serves the client automatically for us, as we’ll
see, so for now we only have to install one module:

```bash
npm install --save socket.io
```

That will install the module and add the dependency to `package.json`. Now let’s
edit index.js to add it:

```javascript
var app = require("express")();
var http = require("http").Server(app);
var io = require("socket.io")(http);

app.get("/", function(req, res) {
  res.sendFile(__dirname + "/index.html");
});

io.on("connection", function(socket) {
  console.log("a user connected");
});

http.listen(3000, function() {
  console.log("listening on *:3000");
});
```

Notice that I initialize a new instance of socket.io by passing the http (the
HTTP server) object. Then I listen on the connection event for incoming sockets,
and I log it to the console.

Now in index.html I add the following snippet before the `</body>`:

```javascript
<script src="/socket.io/socket.io.js"></script>

<script>
  var socket = io();
</script>
```

That’s all it takes to load the socket.io-client, which exposes a io global, and
then connect.

Notice that I’m not specifying any URL when I call io(), since it defaults to
trying to connect to the host that serves the page.

If you now reload the server and the website you should see the console print “a
user connected”. Try opening several tabs, and you’ll see several messages:
![chat-4](https://socket.io/assets/img/chat-4.png)

Each socket also fires a special disconnect event:

```javascript
io.on("connection", function(socket) {
  console.log("a user connected");
  socket.on("disconnect", function() {
    console.log("user disconnected");
  });
});
```

Then if you refresh a tab several times you can see it in action:

![chat-5](https://socket.io/assets/img/chat-5.png)

## Emitting events

The main idea behind Socket.IO is that you can send and receive any events you
want, with any data you want. Any objects that can be encoded as JSON will do,
and binary data is supported too.

Let’s make it so that when the user types in a message, the server gets it as a
chat message event. The scripts section in index.html should now look as
follows:

<script src="/socket.io/socket.io.js"></script>

<script src="https://code.jquery.com/jquery-1.11.1.js"></script>

<script>
  $(function () {
    var socket = io();
    $('form').submit(function(){
      socket.emit('chat message', $('#m').val());
      $('#m').val('');
      return false;
    });
  });
</script>

And in index.js we print out the chat message event:

```javascript
io.on("connection", function(socket) {
  socket.on("chat message", function(msg) {
    console.log("message: " + msg);
  });
});
```

The result should be like the following video:

## Broadcasting

The next goal is for us to emit the event from the server to the rest of the
users.

In order to send an event to everyone, Socket.IO gives us the io.emit:

```javascript
io.emit("some event", { for: "everyone" });
```

If you want to send a message to everyone except for a certain socket, we have
the broadcast flag:

```javascript
io.on("connection", function(socket) {
  socket.broadcast.emit("hi");
});
```

In this case, for the sake of simplicity we’ll send the message to everyone,
including the sender.

```javascript
io.on("connection", function(socket) {
  socket.on("chat message", function(msg) {
    io.emit("chat message", msg);
  });
});
```

And on the client side when we capture a chat message event we’ll include it in
the page. The total client-side JavaScript code now amounts to:

<script>
  $(function () {
    var socket = io();
    $('form').submit(function(){
      socket.emit('chat message', $('#m').val());
      $('#m').val('');
      return false;
    });
    socket.on('chat message', function(msg){
      $('#messages').append($('<li>').text(msg));
    });
  });
</script>

And that completes our chat application, in about 20 lines of code! This is what
it looks like:

## Homework

Here are some ideas to improve the application:

* Broadcast a message to connected users when someone connects or disconnects
* Add support for nicknames
* Don’t send the same message to the user that sent it himself. Instead, append
  the message directly as soon as he presses enter.
* Add “{user} is typing” functionality
* Show who’s online Add private messaging
* Share your improvements!

## Getting this example

You can find it on GitHub here.

```bash
$ git clone https://github.com/socketio/chat-example.git
```
