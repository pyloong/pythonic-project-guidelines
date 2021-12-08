# 信号解耦

这里所说的信号并不是操作系统的信号，而是 [事件驱动架构](https://en.wikipedia.org/wiki/Event-driven_architecture) 的一种简单实现。

事件驱动架构可以基于发布/订阅模型或者事件流模型。

后面谈到的都是基于发布/订阅模型实现的。

## 历史

Python 中的信号解耦机制可以通过 [pydispatcher](http://pydispatcher.sourceforge.net/) 实现。而且 Django Web 框架中的信号机制也是
基于这个项目衍生的。

该项目的核心逻辑 ---- 弱引用，也在后来引入到 Python 官方库中。此后该项目也在 2015 年不再更新。

而之后社区也出现一些信号框架，和在底层实现类似于 [pydispatcher](http://pydispatcher.sourceforge.net/) 功能的逻辑。

## 信号框架

### pydispatcher

[pydispatcher](http://pydispatcher.sourceforge.net/)  提供多生产者-多消费者信号注册和路由基础设施，以在多个上下文中使用。

#### 使用示例

```python
from pydispatch import dispatcher

start_process = 'process'


def audit(name):
    print(f'{name} processing ......')


dispatcher.connect(audit, signal=start_process, sender=dispatcher.Any)


class ETL:
    name = 'foo'

    def process(self):
        """"""
        dispatcher.send(signal=start_process, sender=self, name=self.name)


if __name__ == '__main__':
    ETL().process()

```

上述示例中 `start_process` 订阅了 `audit` 事件，然后在执行 `ETL.process` 的时候，通过 `dispatcher.send` 一条记录，
同时触发该事件执行。

[pydispatcher](http://pydispatcher.sourceforge.net/) 支持指定特定信号，和发送者或匿名。信号可以是特定或者匿名。对象由 Python 解释器
解释器管理，如果对象被回收，则不会在触发。

### blinker

[blinker](https://pythonhosted.org/blinker/) 为Python对象提供快速和简单的对象和广播信号。其内部逻辑依然使用的是弱引用。使用起来和
[pydispatcher](http://pydispatcher.sourceforge.net/) 类似。

#### 使用示例

```python
from blinker import Signal



class AltProcessor:

   on_ready = Signal()
   on_complete = Signal()

   def __init__(self, name):
       self.name = name

   def go(self):
       self.on_ready.send(self)
       print("Alternate processing.")
       self.on_complete.send(self)

   def __repr__(self):
       return '<AltProcessor %s>' % self.name


apc = AltProcessor('c')


@apc.on_complete.connect
def completed(sender):
    print "AltProcessor %s completed!" % sender.name


if __name__ == '__main__':
    apc.go()

```

[blinker](https://pythonhosted.org/blinker/) 同样支持匿名信号，底层的弱引用机制可以减少对象的引用。它有一个好处是支持
装饰器订阅事件，使用起来比较方便。

### aiosignal

[aiosignal](https://github.com/aio-libs/aiosignal) 是从 [aiohttp](https://github.com/aio-libs/aiohttp) 中独立出来的异步信号框架。
它和上述两个信号框架区别有：一，它是一个异步信号框架，可以订阅异步事件；二，在订阅事件时，属于强引用。

#### 使用示例

```python
import asyncio

from aiosignal import Signal

signal = Signal('signal')


async def receiver(message: str):
    print(f'I receive message: {message}')


signal.append(receiver)
signal.freeze()


async def main():
    await signal.send('I am god!')


if __name__ == '__main__':
    asyncio.run(main())

```

在底层， `Signal` 是继承了 `MutableSequence` 类，使用 `Signal.append` 方法将订阅的事件保存在对象的属性中。
当调用 `Signal.send` 方法时，会遍历订阅的事件列表，然后执行。

### 实现自定义的异步信号

[aio-pydispatch](https://pypi.org/project/aio-pydispatch/)

#### 源代码

`aio_signal.signal.py`

```python
"""
Asyncio pydispatch (Signal Manager)
This is based on [pyDispatcher](http://pydispatcher.sourceforge.net/) reference
[scrapy SignalManager](https://docs.scrapy.org/en/latest/topics/signals.html) implementation on
[Asyncio](https://docs.python.org/3/library/asyncio.html)
"""

import asyncio
import functools
import logging
import threading
import weakref
from typing import (Any, Awaitable, Callable, List, Optional, Tuple, TypeVar,
                    Union)

from aio_pydispatch.utils import id_maker, safe_ref

T = TypeVar('T')    # pylint: disable=invalid-name

logger = logging.getLogger(__name__)


class _IgnoredException(Exception):
    """Ignore exception"""


class Signal:
    """
    The signal manager, you can register functions to a signal,
    and store in it.
    Then you can touch off all function that registered on the
    signal where you want.
    example:
        import asyncio
        from aio_pydispatch import Signal
        server_start = Signal('server_start')
        server_stop = Signal('server_stop')
        def ppp(value: str) -> None:
            print(value)
        async def main():
            server_start.connect(ppp)
            server_stop.connect(ppp)
            await server_start.send('server start')
            await asyncio.sleep(1)
            await server_stop.send('server stop')
        if __name__ == '__main__':
            asyncio.run(main())
    """

    def __init__(
            self,
            name: Optional[str] = None,
            doc: Optional[str] = None,
    ):
        self._name = name
        self._doc = doc

        self.__lock = threading.Lock()

        self.__should_clear_receiver = False

        self._receivers = {}

    @property
    def receivers(self):
        """Receivers"""
        return self._receivers

    def connect(
            self,
            receiver: Callable[..., Union[T, Awaitable]],
    ) -> Callable[..., Union[T, Awaitable]]:
        """
        Connect a receiver on this signal.
        :param receiver:
        :return:
        """
        assert callable(receiver), "Signal receivers must be callable."

        referenced_receiver = safe_ref(receiver, self._set_should_clear_receiver, value=True)

        lookup_key = id_maker(receiver)

        with self.__lock:
            self._clear_dead_receivers()

            if lookup_key not in self._receivers:
                self._receivers.setdefault(lookup_key, referenced_receiver)
            self._set_should_clear_receiver(False)
        return receiver

    async def send(self, *args, **kwargs) -> List[Tuple[Any, Any]]:
        """Send signal, touch off all registered function."""
        _dont_log = kwargs.pop('_ignored_exception', _IgnoredException)
        responses = []
        loop = asyncio.get_running_loop()
        for receiver in self.live_receivers:
            func = functools.partial(
                receiver,
                *args,
                **kwargs
            )
            try:
                if asyncio.iscoroutinefunction(receiver):
                    response = await func()
                else:
                    response = await loop.run_in_executor(None, func)
            except _dont_log as ex:
                response = ex
            except Exception as ex:  # pylint: disable=broad-except
                response = ex
                logger.error('Caught an error on %s', receiver, exc_info=True)
            responses.append((receiver, response))

        return responses

    def sync_send(self, *args, **kwargs) -> List[Tuple[Any, Any]]:
        """
        Can only trigger sync func. If func is coroutine function,
        it will return a awaitable object.
        :param args:
        :param kwargs:
        :return:
        """
        _dont_log = kwargs.pop('_ignored_exception', _IgnoredException)
        responses = []
        for receiver in self.live_receivers:
            try:
                if asyncio.iscoroutinefunction(receiver):
                    logger.warning('%s is coroutine, but it not awaited', receiver)
                response = receiver(*args, **kwargs)
            except _dont_log as ex:
                response = ex
            except Exception as ex:  # pylint: disable=broad-except
                response = ex
                logger.error('Caught an error on %s', receiver, exc_info=True)
            responses.append((receiver, response))

        return responses

    @property
    def live_receivers(self) -> List[Callable[..., Union[T, Awaitable]]]:
        """Get all live receiver."""
        with self.__lock:
            receivers = []
            _receiver = self._receivers.copy()
            for lookup_key, receiver in _receiver.items():
                if isinstance(receiver, weakref.ReferenceType):
                    real_receiver = receiver()
                    if real_receiver is None:
                        self._receivers.pop(lookup_key)
                    else:
                        receivers.append(real_receiver)
            return receivers

    def _set_should_clear_receiver(self, value: bool) -> None:
        """Register to the receiver weakerf finalize callback"""
        self.__should_clear_receiver = value

    def _clear_dead_receivers(self) -> None:
        if self.__should_clear_receiver:
            _receiver = self._receivers.copy()
            for lookup_key, receiver in _receiver.items():
                if isinstance(receiver, weakref.ReferenceType) and receiver() is not None:
                    continue
                self._receivers.pop(lookup_key)

    def disconnect(self, receiver) -> None:
        """clean a receiver"""
        self._receivers.pop(id_maker(receiver))

    def disconnect_all(self) -> None:
        """Clean all receiver."""
        self._receivers.clear()
```

`aio_signal.utils.py`

```python
"""Utils"""
import weakref
from typing import Any, Callable, Tuple
from weakref import ReferenceType, WeakMethod


def ref_adapter(receiver: Any) -> Tuple[Any, ReferenceType]:
    """
    Adapt a receiver to ref object.
    :param receiver:
    :return:
    """
    ref = weakref.ref
    receiver_obj = receiver

    # Check if receiver is a ref.
    if hasattr(receiver, '__self__') and hasattr(receiver, '__func__'):
        ref = WeakMethod
        receiver_obj = receiver.__self__
    referenced_receiver = ref(receiver)

    return receiver_obj, referenced_receiver


def safe_ref(receiver: Any, callback: Callable[..., None], *args, **kwargs) -> ReferenceType:
    """
    Save ref a receiver.
    Register a callback function to the object finalizer
    :param receiver:    A ref object
    :param callback:    Register the callback function to the object finalizer
    :param args:    Callback args
    :param kwargs:  Callback kwargs
    :return:
    """
    receiver_obj, receiver = ref_adapter(receiver)
    weakref.finalize(receiver_obj, callback, *args, **kwargs)
    return receiver


def id_maker(receiver: Any) -> int:
    """
    Get receiver id.
    If receiver is ref object, will return a ref object id.
    :param receiver:
    :return: Any
    """
    if not isinstance(receiver, ReferenceType):
        _, receiver = ref_adapter(receiver)
    return id(receiver)

```

自定义的 `aio_signal` 支持订阅同步事件和异步事件。发布时支持同步发布和异步发布。异步发布支持触发同步事件和
异步事件，同步发布只支持触发同步事件。

#### 使用

```python
import asyncio

from aio_signal import Signal

server_start = Signal('server_start')
server_stop = Signal('server_stop')


def ppp(value: str) -> None:
    print(value)


async def main():
    server_start.connect(ppp)
    server_stop.connect(ppp)
    await server_start.send('server start')
    await asyncio.sleep(1)
    await server_stop.send('server stop')


if __name__ == '__main__':
    asyncio.run(main())

```

## 实践

在开发实践中，推荐使用带有弱引用的信号库。这样可以避免资源占用。
