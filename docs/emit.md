## Emit 备忘单

```javascript
io.on('connect', onConnect);

function onConnect(socket){

  // 发送给客户端
  socket.emit('hello', 'can you hear me?', 1, 2, 'abc');

  // 发送给所有客户端(除了发送者)
  socket.broadcast.emit('broadcast', 'hello friends!');

  // 发送给所有在'game'房间中的客户端(除了发送者)
  socket.to('game').emit('nice game', "let's play a game");

  // 发送给所有在'game1'和'game2'的客户端(除了发送者)
  socket.to('game1').to('game2').emit('nice game', "let's play a game (too)");

  // 发送给所有在'game'房间中的客户端(包括发送者)
  io.in('game').emit('big-announcement', 'the game will start soon');

  // 发送给所有在命名空间'myNamespace'中的客户端(包括发送者)
  io.of('myNamespace').emit('bigger-announcement', 'the tournament will start soon');

  // 发送给特定命名空间中的特定房间(包括发送者)
  io.of('myNamespace').to('room').emit('event', 'message');

  // 发送给指定客户端 (私信)
  socket.to(<socketid>).emit('hey', 'I just met you');

  // 发送带有回执的消息
  socket.emit('question', 'do you think so?', function (answer) {});

  // 不压缩消息
  socket.compress(false).emit('uncompressed', "that's rough");

  // 如果客户端不准备接收，则发送可能被丢弃的消息
  socket.volatile.emit('maybe', 'do you really need it?');

  // 发送到节点上的所有客户端(使用多节点情况下)
  io.local.emit('hi', 'my lovely babies');

  // 发送给所有连接的客户端
  io.emit('an event sent to all connected clients');
};
```

**Note:** 以下事件名为保留字，不能以此作为自定义事件名

* `error`
* `connect`
* `disconnect`
* `disconnecting`
* `newListener`
* `removeListener`
* `ping`
* `pong`
