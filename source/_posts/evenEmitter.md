---
title: 深入探究 Node 中的 events.EventEmitter 模块
date: 2017-07-22 23:20:55
tags:
    - node
---

Node 中的许多核心 API 都是通过事件驱动的异步架构实现的，具体来说就是当 `emitters` 发送事件后，相应的响应函数（ `listeners` ）会被执行。例如：`net.Server` 会在每次收到连接时发出事件，`fs.ReadStram` 会在文件打开时发出事件，`stram`会在有数据可读时发出事件。
所有这些对象都是 `EventEmitter` 的实例，它们通过向外暴露的 `eventEmitter.on()` 接口从而让不同的事件响应函数得以执行。

<!--more-->

##  基本的使用
### on 和 emit 方法
events 模块有且只有一个对象 `events.EventEmitter`，它的核心功能就是事件的触发（`emit`）和事件的监听（`on`），一个简单的例子如下：
```JavaScript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('hi', (name) => {
    console.log(`hi, my name is ${name}!`);
});

emitter.emit('hi', 'elvin');
```

在上述的例子中，我们通过 `emitter.on('hi', func)` 的方式注册了 hi 事件的监听函数，通过 `emitter.emit('hi', 'elvin')` 的方式触发了 hi 事件，且会向事件处理函数传递参数 'elvin'，所以最后的执行结果为 `hi, my name is elvin!`。
这里需要说明的时，EventEmitter 还有一个 `addeListener` 的方法，它只不过是 `on` 方法的别名，两者没有任何区别。

### once 方法
有些时候，我们希望某些事件响应函数只被执行一次，这个时候就可以使用 `once()` 方法，它会和 `on()` 一样注册事件的响应函数，不过当响应函数执行一次之后，就会将其移除。
```JavaScript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.once('hi', (name) => {
    console.log(`hi, my name is ${name}!`);
});

emitter.emit('hi', 'elvin');
emitter.emit('hi', 'leonard');
```
上面的例子中只会输出 `hi, my name is elvin!`（leonard 很高冷，不屑于向你打招呼┗( ▔, ▔ )┛）。

### prependListener 方法
当一个事件绑定了多个响应函数时，会按照函数绑定的顺序依次执行，除非响应函数是通过 `prependListener()` 方法绑定的，它使用的方式和 `on()` 类似，不过会将响应函数插到当前该事件处理函数队列的头部，具体的例子如下：
```JavaScript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('hi', (name) => {
    console.log(`my name is ${name}!`);
});

emitter.on('hi', () => {
    console.log('I\'m from Wuhan.');
});

emitter.prependListener('hi', () => {
    console.log('nice to meet you!');
});

emitter.on('hi', () => {
    console.log('What\'s your name?');
});

emitter.emit('hi', 'elvin');

// 输出为：
// nice to meet you!
// my name is elvin.
// I'm from Wuhan.
// What\'s your name?
```

### 响应函数的数量
因为绑定过多的响应函数会消耗大量的内存，所以为了避免内存泄漏，在 `Event.EventEmitter`中一个事件可以绑定的响应函数数量是存在限制的，相关的属性和方法如下：
- **EventEmitter.defaultMaxListeners**： 默认值为10， 表示每个事件的最多可以绑定的响应函数数量。需要注意的是，当修改它时，会影响所有 `EventEmitter` 的实例。
- **emitter.listenerCount(eventName)**：获取事件 `eventName` 已绑定的响应函数个数。
- **emitter.setMaxListeners(n)**：修改 emitter 的每个事件最多可以绑定的响应函数数量，该方法会修改 `emitter._maxListeners` 的值，其优先级大于 `*EventEmitter.defaultMaxListeners` 。
- **emitter.getMaxListeners()**：获取 emitter 每个事件最多可以绑定的响应函数数量。

### 其他相关方法
`EventEmitter` 还有一些其他的方法和属性，这里就不做具体介绍，简要地说一下。
- **emitter.eventNames()**：返回当前已经绑定响应函数的事件名组成的数组。
- **emitter.listeners(eventName)**：返回 `eventName` 事件的响应函数组成的数组。
- **emitter.prependOnceListener(eventName, listener)**：类似于 `once()`，不过会将响应函数插到当前该事件处理函数队列的头部。
- **emitter.removeAllListeners([eventName])**：移除 `eventName` 事件所有的响应函数。当未传入 `eventName` 参数时，所有事件的响应函数都会被移除。
- **emitter.removeListener(eventName, listener)**：移除 `eventName` 事件的响应函数 `listener`。

### newListener 和 removeListener 事件
当 emitter 被注册响应函数时，会触发 `newListener` 事件；被移除响应函数时，会触发 `removeListener` 事件。两个事件的响应函数会被传入两个参数：注册的事件名和响应的响应函数。具体的例子如下：
```JavaScript
const EventEmitter = require('events');

let emitter = new EventEmitter();

// 注意：此处使用 emitter.on 方法的话会陷入循环调用，导致栈溢出
emitter.once('newListener', (event, listener) => {
    if (event === 'hi') {
        emitter.on('hi', () => {
            console.log('Nice to meet you.');
        });
    }
});

emitter.on('hi', (name) => {
    console.log(`My name is ${name}!`);
});

emitter.emit('hi', 'elvin');
```
运行结果为 `Nice to meet you. My name is elvin.`。实际上， `newListener` 事件被触发时，响应函数还未被注册至 emitter，因而我们就可以在在目标响应函数之前插入其他响应函数，例如上面的例子中 `Nice to meet you.` 就在 `My name is elvin.` 之前进行输出。


## 在 Node 源码中的使用
如在开头所说，`net.Server`、`fs.ReadStram`、`stream` 等 Node 内建对象都是 `EventEmitter` 的实例，它们通过向外暴露的 `eventEmitter.on()` 接口从而让不同的事件响应函数得以执行。这里以 `stream` 的部分源码为例，讲讲 `events.EventEmitter` 在 Node 中的使用。
```JavaScript
// stream 部分源码 2017/02/22 版
// source: https://github.com/nodejs/node/blob/master/lib/internal/streams/legacy.js
const EE = require('events');
const util = require('util'); // Node 的辅助函数库

function Stream() {
  EE.call(this);
}
util.inherits(Stream, EE);

Stream.prototype.pipe = function(dest, options) {
  var source = this;

  function ondata(chunk) {
    if (dest.writable) {
      if (false === dest.write(chunk) && source.pause) {
        source.pause();
      }
    }
  }

  source.on('data', ondata);

  function ondrain() {
    if (source.readable && source.resume) {
      source.resume();
    }
  }

  dest.on('drain', ondrain);

  // If the 'end' option is not supplied, dest.end() will be called when
  // source gets the 'end' or 'close' events.  Only dest.end() once.
  if (!dest._isStdio && (!options || options.end !== false)) {
    source.on('end', onend);
    source.on('close', onclose);
  }

  var didOnEnd = false;
  function onend() {
    if (didOnEnd) return;
    didOnEnd = true;

    dest.end();
  }


  function onclose() {
    if (didOnEnd) return;
    didOnEnd = true;

    if (typeof dest.destroy === 'function') dest.destroy();
  }

  // don't leave dangling pipes when there are errors.
  function onerror(er) {
    cleanup();
    if (EE.listenerCount(this, 'error') === 0) {
      throw er; // Unhandled stream error in pipe.
    }
  }

  source.on('error', onerror);
  dest.on('error', onerror);

  // remove all the event listeners that were added.
  function cleanup() {
    source.removeListener('data', ondata);
    dest.removeListener('drain', ondrain);

    source.removeListener('end', onend);
    source.removeListener('close', onclose);

    source.removeListener('error', onerror);
    dest.removeListener('error', onerror);

    source.removeListener('end', cleanup);
    source.removeListener('close', cleanup);

    dest.removeListener('close', cleanup);
  }

  source.on('end', cleanup);
  source.on('close', cleanup);

  dest.on('close', cleanup);
  dest.emit('pipe', source);

  // Allow for unix-like usage: A.pipe(B).pipe(C)
  return dest;
};

module.exports = Stream;
```
上述代码主要完成了三件事情：
1. 通过在 Stream 的构造函数中调用 `EE.call(this)` 和利用 `util.inherits(Stream, EE);` 调用，让 Sever 继承自 EventEmitter;
2. 通过 on 方法在数据源 sourse 上注册了 data、end、close、error 等事件的响应函数，在数据目的源 dest 上注册了 drain、end、close、error 等事件的响应函数；
3. 在完成初始化和响应函数注册后，向数据目的源发出 pipe 事件。

## ES6 中的使用方式
如上节所示，在 Node 中都是通过 `util.inherits(Stream, EventEmitter);` 来实现继承，但是在 Node 的官方文档中，该种方式已不被推荐。更推荐的做法是通过 ES6 中的 `class` 和 `extends` 来实现继承：
```JavaScript
const EventEmitter = require('events');

class MyStream extends EventEmitter {
  write(data) {
    this.emit('data', data);
  }
}

const stream = new MyStream();

stream.on('data', (data) => {
  console.log(`Received data: "${data}"`);
});
stream.write('With ES6');


// output: Received data: "With ES6"
```

## 写在最后
事件驱动、非异步 I/O 的特点成就了如今的 Node，而 Node 中事件驱动依靠的就是 events.EventEmitter！需要说明的是，events 和 events.EventEmitter 其实指向的都是 EventEmitter，之所以会有 events.EventEmitter 只是为了保持对 Node 0.10.x 版本的兼容。

## 参考链接

1. [Node.js Documentation - evnets](https://nodejs.org/api/events.html)