# 入门 : 聊天应用程序

在这个教程中，我们将创建一个基本聊天应用程序。因为几乎不需要预先了解 NodeJS 或
Socket.IO 的知识点，所以很适合任何新手或者老油条。

## 简介

使用流行的 Web 应用程序技术栈，如 LAMP（PHP ）等编写聊天应用程序非常困难。它涉及
到轮询服务器的变化、时间戳追踪等，因此效率比较低下。

Sockets 一直是大多数实时聊天系统所架构的解决方案，提供客户端与服务器之间的双向通
信信道。

这意味着服务器可以将消息推送到客户端。因此每当你发送一个聊天消息给服务端，它就会
把该消息推送到所有其他连接的客户端。

## Web 框架

首先需要一个简单的 HTML 网页，提供一个表单和一个消息列表。此例子我们将使用
Node.JS Web 框架。

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

为了方便管理依赖，我们使用`npm install --save` 来安装。

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

以上代码的解释：

1. `Express` 将应用程序初始化为可以提供给 `HTTP` 服务器的函数处理程序 ( 如第二行
   所示 )。
2. 然后定义一个路由 `/` 以便在访问首页的时候被调用。
3. 最后在 3000 端口上进行 http 服务监听。输入`node index.js` 运行程序，你将会看
   到如下所示： ![chat-1](https://socket.io/assets/img/chat-1.png)

在浏览器中输入 `http://localhost:3000`即可看到如下页面 :

![chat-2](https://socket.io/assets/img/chat-2.png)

## 提供 HTML

目前为止，在`index.js`中，我们调用`res.send`并传递一个 HTML 字符串。但是如果我们
都是以此字符串的方式提供 HTML，代码看起来会非常混乱。因此我们要创建一
个`index.html`文件并提供给程序进行渲染。

重构路由使用`sendFile`来代替`send`。

```javascript
app.get("/", function(req, res) {
  res.sendFile(__dirname + "/index.html");
});
```

然后添加如下 Html 代码 :

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
      <input id="m" autocomplete="off" /><button>发送</button>
    </form>
  </body>
</html>
```

在命令行中按下 Control+C 结束此前的进程，然后重新运行，刷新页面之后将会看到如下
页面： ![chat-3](https://socket.io/assets/img/chat-3.png)

## 整合 Socket.IO

Socket.IO 由两部分组成 :

* Node.JS 服务端所需要的：`socket.io`
* 浏览器端需要的客户端库：`socket.io-client`

开发过程中`socket.io`会给我们自动提供客户端库，因此我们只需要安装一个模块就可以
：

```bash
npm install --save socket.io
```

该命令安装的同时也会在`package.json`中添加依赖。接下来编辑`index.js`, 添加如下代
码：

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

通过传递 http 服务来初始化一个新的 socket.io 实例。 然后监听 sockets 的连接事件
，并在控制台中输出一个字符串。

编辑`index.html`，将以下代码添加到`</body>`元素前 :

```javascript
<script src="/socket.io/socket.io.js"></script>

<script>
  var socket = io();
</script>
```

这段代码会加载`socket.io-client`客户端脚本，并向外暴露出一个全局的 io，然后使用
此 io 进行连接。

可以看到在调用 io() 时没有指定任何 URL，因为它默认为连接到本机服务器。

现在重启服务器并刷新网站，就可以看到控制台打印的`a user connected`。 尝试多打开
几个标签，你会看到几个消息： ![chat-4](https://socket.io/assets/img/chat-4.png)

每个 socket 都会触发一个断开事件 :

```javascript
io.on("connection", function(socket) {
  console.log("a user connected");
  socket.on("disconnect", function() {
    console.log("user disconnected");
  });
});
```

那么如果你多次刷新同一个标签，你可以看到以下行为：

![chat-5](https://socket.io/assets/img/chat-5.png)

## 发送事件

Socket.IO 的设计思想是你可以在发送和接收任何事件时附加任何数据。 该数据可以为
JSON 对象，也可以是二进制数据。

接下来我们将做这个例子，当用户输入信息时，服务器将其作为一个 chat message event
发送给服务端。

修改 `index.html` 中的脚本，如下所示：

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

打印 chat message event

```javascript
io.on("connection", function(socket) {
  socket.on("chat message", function(msg) {
    console.log("message: " + msg);
  });
});
```

## 广播

接下来让我们从服务器发送事件到其余的用户。

为了方便向所有用户发送事件，Socket.IO 给我们提供了 io.emit：

```javascript
io.emit("some event", { for: "everyone" });
```

如果你想发消息给某个用户之外的所有用户，可以使用`broadcast`来发送。

```javascript
io.on("connection", function(socket) {
  socket.broadcast.emit("hi");
});
```

下面的代码表示给所有人发送消息，包括发送者

```javascript
io.on("connection", function(socket) {
  socket.on("chat message", function(msg) {
    io.emit("chat message", msg);
  });
});
```

在客户端中，我们将会捕获到到 chat message event，然后在页面中进行相应的操作。完
整的代码如下所示：

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

至此我们的聊天小程序就完成了，仅仅只需要 20 行代码。够简单吧。

## 家庭作业

以下是改进应用程序的一些想法：

* 当有人连接或断开连接时，将消息广播给连接的所有用户。
* 支持昵称。
* 不要向发送它的用户发送相同的消息。相反，他一按回车就直接追加信息。
* 加入 “{ 用户 } 正在输入 ...” 功能。
* 显示在线人，并支持发送私聊。
* 分享你的改进成果。

## 获取完整示例代码

Github 地址：

```bash
$ git clone https://github.com/socketio/chat-example.git
```
