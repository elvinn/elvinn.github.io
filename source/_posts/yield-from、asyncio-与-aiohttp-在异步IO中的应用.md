---
title: yield from、asyncio 与 aiohttp 在异步IO中的应用
tags:
  - python
date: 2016-11-17 22:43:34
---

Python3.3 中引入了 `yield from` 用于定义生成器;Python3.4 中引入了 `asyncio`标准库， 从而直接内置了对异步IO的支持。
<!--more-->

## `yield from`的简单实例
```python
def generator1():
    for i in range(10):
        yield i
def generator2():
    for i in range(10, 20):
        yield i
def generator():
    yield from generator1()
    yield from generator2()
    
for i in generator():
    print(i)
```

在上面的例子中， 最终会输出0~19。通过`yield from`的使用，我们可以将不同的生成器组合在一起，形成新的生成器，就如同通过不同函数的调用形成新德函数一样。

## `next` && `send`

*   `next` 返回生成器的下一个值， 然后生成器挂起。
*   `send` 对于一个生成器对象g，如果调用`g.send(None)`， 那么效果和`next(g)`一样；如果调用`g.send(value)`，就相当于yield的返回值。

## `asyncio`的简单实例
```python
import asyncio
@asyncio.coroutine
def hello():
    print('Hello World!')
    yield from asyncio.sleep(1)
    print('GoodBye!')
loop = asyncio.get_event_loop()
tasks = [hello(), hello(), hello()]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

观察运行结果：

```
Hello World!
Hello World!
Hello World!
(暂停约1秒）
GoodBye!
GoodBye!
GoodBye!
```

上述例子的运行流程如下：

1.  `@asyncio.coroutine`把一个生成器（此处是hello())标记为coroutine类型，然后我们把这个coroutine扔到EventLoop中执行。
2.  通过`asyncio.get_event_loop()`创建消息循环的列表。
3.  `loop.run_until_complete(asyncio.wait(tasks))`异步执行消息列表中的事件。
4.  在`hello()`函数中，`yield from`语法可以让我们调用另一个generator。由于`asyncio.sleep()`也是一个`coroutine`，所以线程不会等待`asyncio.sleep()`，而是直接中断并执行下一个消息循环。当`asyncio.sleep()`返回时，线程就可以从`yield from`拿到返回值（此处是None），然后接着执行下一行语句。
5.  最后`loop.close()`结束消息循环的列表。

## `aiohttp`的简单实例

`asyncio`实现了TCP、UDP、SSL等协议，`aiohttp`则是基于asyncio实现的HTTP框架。

```python
# Simple HTTP Server
# coding=utf-8
import asyncio
from aiohttp import web
def index(request):
    return web.Response(body=b'<h1>Index</h1>')
def hello(request):
    yield from asyncio.sleep(0.5)
    text = '<h1>hello, %s!</h1>' % request.match_info['name']
    return web.Response(body=text.encode('utf-8'))
@asyncio.coroutine
def init(loop):
    app = web.Application(loop=loop)
    app.router.add_route('GET', '/', index)
    app.router.add_route('GET', '/hello/{name}', hello)
    srv = yield from loop.create_server(app.make_handler(), '127.0.0.1', 8000)
    print('Server started at http://127.0.0.1:8000 ...')
    return srv
loop = asyncio.get_event_loop()
loop.run_until_complete(init(loop))
loop.run_forever()
```