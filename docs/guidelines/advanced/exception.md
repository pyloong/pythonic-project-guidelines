# 异常管理

几乎所有编程语言中都有异常。异常可以快速指出程序出现的问题，便于排查。开发人员也可以根据情况抛出自定义异常，
以指示期望的内容和实际不相符。良好的异常设计和使用习惯，可以提高程序的质量。

## 介绍

Python 中的异常分为两类，一是句法错误，一类是异常。

### 句法错误

句法错误是用来指示 Python 编码不符合句法规范的：

```python
>>> while True print('Hello world')
  File "<stdin>", line 1
    while True print('Hello world')
                   ^
SyntaxError: invalid syntax
```

如上所示，使用 `^` 指示错误的位置。

### 异常

即使语句或表达式使用了正确的语法，执行时仍可能触发错误。执行时检测到的错误称为 异常，
异常不一定导致严重的后果：很快我们就能学会如何处理 Python 的异常。大多数异常不会被程序处理，
而是显示下列错误信息：

```python
>>> 10 * (1/0)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: division by zero
>>> 4 + spam*3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'spam' is not defined
>>> '2' + 2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only concatenate str (not "int") to str
```

内置异常结构如下：

```python
BaseException
 +-- SystemExit
 +-- KeyboardInterrupt
 +-- GeneratorExit
 +-- Exception
      +-- StopIteration
      +-- StopAsyncIteration
      +-- ArithmeticError
      |    +-- FloatingPointError
      |    +-- OverflowError
      |    +-- ZeroDivisionError
      +-- AssertionError
      +-- AttributeError
      +-- BufferError
      +-- EOFError
      +-- ImportError
      |    +-- ModuleNotFoundError
      +-- LookupError
      |    +-- IndexError
      |    +-- KeyError
      +-- MemoryError
      +-- NameError
      |    +-- UnboundLocalError
      +-- OSError
      |    +-- BlockingIOError
      |    +-- ChildProcessError
      |    +-- ConnectionError
      |    |    +-- BrokenPipeError
      |    |    +-- ConnectionAbortedError
      |    |    +-- ConnectionRefusedError
      |    |    +-- ConnectionResetError
      |    +-- FileExistsError
      |    +-- FileNotFoundError
      |    +-- InterruptedError
      |    +-- IsADirectoryError
      |    +-- NotADirectoryError
      |    +-- PermissionError
      |    +-- ProcessLookupError
      |    +-- TimeoutError
      +-- ReferenceError
      +-- RuntimeError
      |    +-- NotImplementedError
      |    +-- RecursionError
      +-- SyntaxError
      |    +-- IndentationError
      |         +-- TabError
      +-- SystemError
      +-- TypeError
      +-- ValueError
      |    +-- UnicodeError
      |         +-- UnicodeDecodeError
      |         +-- UnicodeEncodeError
      |         +-- UnicodeTranslateError
      +-- Warning
           +-- DeprecationWarning
           +-- PendingDeprecationWarning
           +-- RuntimeWarning
           +-- SyntaxWarning
           +-- UserWarning
           +-- FutureWarning
           +-- ImportWarning
           +-- UnicodeWarning
           +-- BytesWarning
           +-- EncodingWarning
           +-- ResourceWarning
```

## 使用

### 捕获异常

在逻辑中，可能出现不符合预期的逻辑，会抛出相关异常。此时在编码时，为了逻辑的正常运行，需要对逻辑进行处理：

```python
import sys

try:
    f = open('myfile.txt')
    s = f.readline()
    i = int(s.strip())
except OSError as err:
    print("OS error: {0}".format(err))
except ValueError:
    print("Could not convert data to an integer.")
except BaseException as err:
    print(f"Unexpected {err=}, {type(err)=}")
    raise
```

如上述逻辑，对于已知能判断的情况，可以通过日志输出显示友好信息，避免程序立即停止。当无法判断异常时，则
继续抛出异常。

捕获异常是，使用 `try...except` 代码块包裹需要处理异常的代码。 `expect` 捕获指定的异常类型，如果出现，进入
对应的代码逻辑。对于一些不想处理的，通过 `raise` 抛出异常。

### 异常链

当抛出异常时， `raise` 语句支持 `from` 子句启用链式异常。

```python
>>> def func():
...     raise ConnectionError
...
>>> try:
...     func()
... except ConnectionError as exc:
...     raise RuntimeError('Failed to open database') from exc
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
  File "<stdin>", line 2, in func
ConnectionError

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
RuntimeError: Failed to open database
```

上述示例中，异常信息中含有两次抛出的异常。这对于调试很有帮助。

如果不想抛出链式异常，可以使用 `from None` ：

```python
>>> try:
...     open('database.sqlite')
... except OSError:
...     raise RuntimeError from None
...
Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
RuntimeError
```

### 自定义异常

程序可以通过创建新的异常类命名自己的异常（Python 类的内容详见 类）。不论是以直接还是间接的方式，异常都应从 Exception 类派生。

异常类和其他类一样，可以执行任何操作。但通常会比较简单，只提供让处理异常的程序提取错误信息的一些属性。
创建能触发多个不同错误的模块时，一般只为该模块定义异常基类，然后再根据不同的错误条件，创建指定异常类的子类：

```python
class Error(Exception):
    """Base class for exceptions in this module."""
    pass

class InputError(Error):
    """Exception raised for errors in the input.

    Attributes:
        expression -- input expression in which the error occurred
        message -- explanation of the error
    """

    def __init__(self, expression, message):
        self.expression = expression
        self.message = message

class TransitionError(Error):
    """Raised when an operation attempts a state transition that's not
    allowed.

    Attributes:
        previous -- state at beginning of transition
        next -- attempted new state
        message -- explanation of why the specific transition is not allowed
    """

    def __init__(self, previous, next, message):
        self.previous = previous
        self.next = next
        self.message = message
```

大多数异常命名都以 “Error” 结尾，类似标准异常的命名。

许多标准模块都需要自定义异常，以报告由其定义的函数中出现的错误。

### 异常清理

对于像文件或者连接对象的操作，在打开后，需要在异常最后关闭，就需要用到异常清理。

```python
import sys

try:
    f = open('myfile.txt')
    s = f.readline()
    i = int(s.strip())
except OSError as err:
    print("OS error: {0}".format(err))
    raise
finally:
    f.close()
```

上述逻辑中，使用 `try...expect...finally` 做抛出异常后的清理工作。其中 `finally` 代码块中，关闭了前面
打开的文件对象。

```python
def divide(x, y):
    try:
        result = x / y
    except ZeroDivisionError:
        print("division by zero!")
    else:
        print("result is", result)
    finally:
        print("executing finally clause")
```

上述示例代码通过 `else` 逻辑块执行没有触发异常时的逻辑。

对于一些清理性的工作，推荐使用 [with](https://docs.python.org/zh-cn/3/reference/compound_stmts.html#with) 语句自动管理上下文。

## 实践

开发实践中，异常信息对诊断程序非常重要。所以在使用和处理异常时，请遵循如下几点：

- 需要处理异常时使用 `try...except...finally` 捕获
- 处理异常时，如果没有继续抛出异常，需要输入日志信息。除非你知道不输出任何信息不会造成拍错困难。
- 项目级别，一定要定义一个项目的基类异常。项目中其他自定义异常必须继承该基类异常。这么做的目的是可以在外层逻辑通过捕获基类
    异常来只捕获抛出的自定义异常。
- 项目异常要以 `ERROR` 结尾。和标准异常命名类似。
