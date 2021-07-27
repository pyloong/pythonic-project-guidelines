# 类型提示

Python 作为一个动态类型语言，在编码过程中出现的一些小问题，直到运行时才被发现。相比于静态语言，
像 Java 、 C/C++ 等，在编译期间就能发现并改进代码问题。 所以为了在运行时之前尽可能避免出问题，
在 2014 年 Guido van Rossum 等人为 Python 提出了 [类型提示理论](https://www.python.org/dev/peps/pep-0483/) 。
在 [2015 年的 Pycon](https://www.youtube.com/watch?v=2wDvzy6Hgxg) 做了该主题的演讲。
直到现在，关于静态类型相关的 PEP 有：

- [PEP 484 -- Type Hints](https://www.python.org/dev/peps/pep-0484/)
- [PEP 526 -- Syntax for Variable Annotations](https://www.python.org/dev/peps/pep-0526/)
- [PEP 544 -- Protocols: Structural subtyping (static duck typing)](https://www.python.org/dev/peps/pep-0544/)
- [PEP 586 -- Literal Types](https://www.python.org/dev/peps/pep-0586/)
- [PEP 589 -- TypedDict: Type Hints for Dictionaries with a Fixed Set of Keys](https://www.python.org/dev/peps/pep-0589/)
- [PEP 591 -- Adding a final qualifier to typing](https://www.python.org/dev/peps/pep-0591/)

到现在的 python 3.9 版本，类型提示的支持已经很丰富了。同时与类型提示相关的检测工具，工具在 IDE 上的集成功能也很完善，
在开发体验上有了很大的提升。同时类型检查也成为了 CI 的重要环节，有助于更早更及时的规避 Bug 的出现。

## 1. 初识类型提示

类型提示可以在类、方法或变量上标注相应的类型，在调用的时候通过静态类型检查工具检测调用是否存在问题。

如下面的例子：

```python
def greeting(name: str) -> str:
    return f'Hello {name}'
```

在定义方法 `greeting` 的时候，声明参数 `name` 是 `str` 类型，返回值是 `str` 类型。当调用 `greeting` 函数时，如果传递一个 `int` 类型 的值，
运行类型检查会失败，同时发出警告提示。如果 IDE 已经支持类型检查，则在调用的时候，会提示该方法的参数类型，
如果传递错误类型的参数 IDE 会及时发出警告，提示我们修复。

```python
import logging
from pathlib import Path  # Config root logger

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)


def count(source_file: str, dest_file: str) -> None:
    """
    Count source
    :param source_file:
    :param dest_file:
    :return:
    """
    total = read_from_file(Path(source_file))
    write_to_file(Path(dest_file), total)


def read_from_file(source_file: Path) -> int:
    """
    Read file
    :param source_file:
    :return:
    """
    total_words = 0
    # Read source_file
    logging.debug('Read file: %s', source_file)
    with open(source_file, 'r') as source_obj:
        for line in source_obj.readlines():
            total_words += len(line.split(' '))
    return total_words


def write_to_file(dest_file: Path, total_words: int) -> None:
    """
    Write result to file
    :param dest_file:
    :param total_words:
    :return:
    """
    logging.debug('Count %s words, write to %d', dest_file, total_words)
    with open(dest_file, 'w') as dest_obj:
        dest_obj.write(f'Total count: {total_words}')

```

上述代码中，所有方法的参数和返回值都进行了类型标注。

## 2. 使用类型提示

### 2.1 一般类型提示

在进行类型标注的过程中，一般直接通过标注变量本身的类型就可以了。

例如：

```python
"""Example"""
import logging

logging.basicConfig(level=logging.DEBUG)


class User:
    """User"""

    def __init__(self, name: str):
        self._name = name

    @property
    def name(self) -> str:
        """User's name"""
        return self._name

    def __repr__(self):
        """repr"""
        return f'<User(name="{self.name}")>'


def save(user: User):
    """Mock to save a user"""
    logging.info('Save object: %s', user)


if __name__ == '__main__':
    save(User('Jim'))

```

如上述例子中， `save` 方法传入一个 `User` 类型的参数，直接使用该类标注就可以了。

针对一般数据类型，如 `int` 、 `str` 、 `float` 、 `bytes` 等，可以直接标注。

## 2.2 泛型具象容器

```python
"""Example"""
from typing import Dict, List


def count_words(records: List[str]) -> Dict[str, int]:
    """Count word of all lines."""
    result: Dict[str, int] = {}
    for record in records:
        for word in record.split(' '):
            count = result.get(word, 0)
            result.update({word: count + 1})
    return result

```

`count_words` 方法接收一个内含 `str` 的 `list` 参数，同时返回 `dict`。

这些 `typing.Dict` 、 `typing.List` 、 `typing.Set` 等都是对应基本数据结构的泛型版本。

> 注意： 根据文档 [模块内容](https://docs.python.org/zh-cn/3/library/typing.html#module-contents) 一节描述，
> 在 Python 3.9 已经对一些基本数据 接口做了泛型适配，这和现有 typing 下的泛型类型重复，
> 所以会弃用这些泛型容器类型，具体请参考体包含哪些请参考 [PEP 585](https://www.python.org/dev/peps/pep-0585/) 。
> 如果需要提前使用新特性，在 Python 3.7 开始，可以导入 `from __future__ import annotations` 来使用新的泛型类型。
> 官方会在 Python 3.9 发布五年后的收个 Python 发行版，即2025年10月5日之后的收个发行版会移除
> [PEP 585](https://www.python.org/dev/peps/pep-0585/) 中弃用的泛型容器类型。

### 2.3 特殊类型

```python
"""Example"""
import asyncio
from typing import Callable, Any, Type, Tuple, Dict, Optional
from functools import partial


class BaseTask:
    """base Task"""

    def run(self) -> bool:
        """Run task"""
        raise NotImplementedError

    def stop(self) -> None:
        """Stop task"""
        raise NotImplementedError


class FileTask(BaseTask):
    """File task"""

    def run(self) -> bool:
        pass

    def stop(self) -> None:
        pass


class NetworkTask(BaseTask):
    """Network task"""

    def run(self) -> bool:
        pass

    def stop(self) -> None:
        pass


KwargsType = Dict[str, Any]
ArgsType = Tuple[Any]


async def run_in_executor(
        func: Callable[..., Any],
        args: Optional[ArgsType] = (),
        kwargs: Optional[KwargsType] = None
) -> Any:
    """Wrap a func in a threading executor """
    if kwargs:
        func = partial(func, **kwargs)
    loop = asyncio.get_running_loop()
    return await loop.run_in_executor(None, func, *args)


def task_runner(task_kls: Type[BaseTask]) -> None:
    """Task runner"""
    task = task_kls()
    asyncio.run(run_in_executor(task.run))

```

从上面的例子可以看到，使用了一些新的类型标注方式。

在 `run_in_executor` 方法之前，定义了两个类型，并赋予其别名，方便后面使用。

在 `run_in_executor` 方法中使用了 `typing.Callable` 、 `typing.Optional` 、 `typing.Any` 特殊类型。

在 `task_runner` 中使用 `typing.Type` 类型，表明 `task_kls` 参数是一个 `BaseTask` 类自身， 而不是它的对象，准确说是它的类对象。

如 `a = int` 和 `b = type(a)` 中， `a` 和 `b` 所标注的类型是一样的，都是 `int` 类型。

### 3. 高阶使用

### 3.1 可调对象(Callable)

上一章已经提到了可调对象( `Callable` ) 的使用，这里需要在详细说明一起它的用法。

```python
from typing import Callable, Tuple


def task_a(name: str) -> str:
    return name


def task_sum(a: int, b: int) -> int:
    return a + b


def task_a_executor(func: Callable[[str], str], args: Tuple[str]) -> str:
    return func(*args)


def task_sum_executor(func: Callable[[int, int], int], args: Tuple[int]) -> int:
    return func(*args)

```

针对可调对象中需要传递参数的类型，可以在 `Callable` 中标注。

从上面示例，包括 `Callable` 的使用方法中可以看到，它都是在标注列表参数( `args` ) ，但如果需要标注字典参数 却无法标注。
例如一个方法 `def foo(a: Optional[int] = None, *, b: Optional[int] = None) -> None: ...`， 它在方法定义阶段已经声明了接收 `b` 参数时，
必须为字典类型，也就是说当你不传递 `a` 参数，但又需要传递 `b` 参数的时候，必须这么调用 `foo(b=3)` ，否则传递的值，只会赋值到 `a` 上面。
而这种类型的调用对象却无法使用正常操作的 标注为 `Callable[[int, "b": int], int]` 。

对于这种情况，虽然官方文档中没有对此说明，但可以通过结构子类型定义调用对象的类型。

> 了解 [名义子类型 vs 结构子类型](https://docs.python.org/zh-cn/3/library/typing.html#nominal-vs-structural-subtyping)

所以可以这么定义：

```python
from typing import Optional, Protocol


def foo(
        a: Optional[int],
        *,
        b: Optional[int]
) -> None: ...


class FooCallableType(Protocol):
    def __call__(
            self,
            a: Optional[int] = None,
            *,
            b: Optional[int] = None
    ) -> None: ...


def foo_executor(func: FooCallableType) -> None: ...

```

> 参考： [python typing signature (typing.Callable) for function with kwargs](https://stackoverflow.com/a/57840786/11722440)

### 3.2 异步编程

```python
import asyncio
from typing import Tuple, Any, Awaitable, Union, Callable, AsyncGenerator
from asyncio import iscoroutinefunction


async def func(length: int) -> AsyncGenerator:
    for i in range(length):
        yield i


async def run_in_executor(
        func: Union[Callable[..., Any], Awaitable[..., Any]],
        args: Tuple[...]
) -> Any:
    if iscoroutinefunction(func):
        return await func(*args)
    else:
        loop = asyncio.get_running_loop()
        return await loop.run_in_executor(None, func, *args, )

```

针对异步编程的的所有类型，都已经在 `typing` 下定义了，可以很方便的去使用。
