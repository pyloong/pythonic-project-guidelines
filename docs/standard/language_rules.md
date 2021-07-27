# Python 语言规范

> 本文档为 [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html) 第二章 [Python Language Rules](https://google.github.io/styleguide/pyguide.html#2-python-language-rules) 的译文。
>
> 最后更新时间： 2021-04-28
>
> 如果有翻译错误或表述不准确的问题，欢迎提交 PR，感谢您的参与。

## 1.1 Lint

使用 [pylintrc](https://google.github.io/styleguide/pylintrc) 配置，对你的代码运行 `pylint`。

### 1.1.1 定义

`Pylint` 是一个在 Python 源代码中查找 bug 和风格问题的工具。对于 C 和 C++ 这样的不那么动态的语言，这些问题通常由编译器来捕获。由于 Python 的动态特性，有些警告可能不对。不过伪告警应该很少。

### 1.1.2 优点

可以捕获容易忽视的错误，例如输入错误，使用未赋值的变量等。

### 1.1.3 缺点

`pylint` 不完美。要利用其优势，我们有时侯需要：围绕着它来写代码、抑制其告警、改进它或者忽略它。

### 1.1.4 结论

确保对你的代码运行 `pylint`。抑制不准确的警告，以便能够将其他警告暴露出来。

你可以通过设置一个行注释来抑制告警。例如：

```python
dict = 'something awful'  # Bad idea... pylint: disable=redefined-builtin
```

`pylint` 警告是以一个符号名 (如 `empty-docstring`) 来标识的，Google 特定的警告以 g- 开头。

如果从符号名称中看不出禁用的原因，那么请对其增加一个详细解释。

采用这种抑制方式的好处是我们可以轻松查找抑制并回顾它们。

您可以通过执行以下操作来获取 `pylint` 警告列表：

```bash
pylint --list-msgs
```

获取关于特定消息的更多信息，可以执行：

```bash
pylint --help-msg=C6409
```

相比较于之前使用的 `pylint: disable-msg`，本文推荐使用 `pylint: disable`。

未使用参数的警告可以通过删除函数开头的变量来消除。并包含一个注释解释为什么删除它。使用 “Unused.” 注释就足够了。例如：

```python
def viking_cafe_order(spam, beans, eggs=None):
    del beans, eggs  # Unused by vikings.
    return spam + spam + spam
```

要抑制这种警告的常见形式还包括使用 “\_" 作为未使用参数的标识符，或在参数名前加上 “unused\_”，或将它们赋值给 “\_"。

上述的这些形式都是允许的，但不再推荐。调用方法时按名称传递的这些参数，实际上并不一定会使用。

## 1.2 导入

只对包和模块使用 import 语句，而不是单独的类或函数。注意： `typing` 模块是个特例。

### 1.2.1 定义

模块间共享代码的重用机制。

### 1.2.2 优点

命名空间管理约定十分简单。每个标识符的来源都用一种一致的方式指示：`x.Obj` 表示 `Obj` 对象定义在模块 `x` 中。

### 1.2.3 缺点

模块名仍可能冲突。有些模块名太长, 不太方便。

### 1.2.4 结论

- 使用 `import x` 来导入包和模块。
- 使用 `from x import y`，其中 `x` 是包前缀，`y` 是不带前缀的模块名。
- 使用 `from x import y as z`，如果两个要导入的模块都叫做 `z` 或者 `y` 太长了。
- 只有当 `z` 是标准缩写（例如，`numpy` 为 `np`）时，才使用 `import y as z`。

例如，模块 `sound.effects.echo` 可以用如下方式导入：

```python
from sound.effects import echo
...
echo.EchoFilter(input, output, delay=0.7, atten=4)
```

导入时不要使用相对名称。即使模块在同一个包中，也要使用完整包名。这能帮助你避免无意间导入一个包两次。

从 `typing` 模块和 `six.moves` 模块导入不受此规则约束。

## 1.3 包

使用模块的全路径名来导入每个模块

### 1.3.1 优点

避免模块名冲突。查找包更容易。

### 1.3.2 缺点

部署代码变难，因为你必须复制包层次。

### 1.3.3 结论

所有的新代码都应该用完整包名来导入每个模块。

应该像下面这样导入：

???+ success "推荐"

    ```python
    # Reference absl.flags in code with the complete name (verbose).
    import absl.flags
    from doctor.who import jodie

    FLAGS = absl.flags.FLAGS
    ```

???+ success "推荐"

    ```python
    # Reference flags in code with just the module name (common).
    from absl import flags
    from doctor.who import jodie

    FLAGS = flags.FLAGS
    ```

???+ warning "不推荐 ( 假设 `jodie.py` 文件在 `doctor/who/where` 中 )"

    ```python
    # Unclear what module the author wanted and what will be imported.  The actual
    # import behavior depends on external factors controlling sys.path.
    # Which possible jodie module did the author intend to import?
    import jodie
    ```

尽管在某些环境中会发生这种情况，但不应假定主二进制文件所在的目录位于 `sys.path` 中。在这种情况下，代码应假定 `import jodie` 引用了名为 `jodie` 的第三方或顶级程序包，而不是本地的 `jodie.py`。

## 1.4 异常

允许使用异常，但必须小心。

### 1.4.1 定义

异常是一种跳出代码块的正常控制流来处理错误或者其它异常条件的方式。

### 1.4.1 优点

正常操作代码的控制流不会和错误处理代码混在一起。当某种条件发生时，它也允许控制流跳过多个框架。例如，一步跳出 N 个嵌套的函数，而不必继续执行错误的代码。

### 1.4.2 缺点

可能会导致让人困惑的控制流。调用库时容易错过错误情况。

### 1.4.4 结论

异常必须遵守特定条件：

- 如果有必要，请使用内置异常类。例如，抛出 `ValureError` 来指示编程错误。比如违反了前置条件（需要一个正数，但传递了一个负数）。不要使用 `assert` 语句验证公共 API 的参数值。`assert` 用于确保内部正确性，不得强制使用，也不表示发生了某些意外事件。如果在后一种情况下需要使用异常，请使用 raise 语句。例如：

    ???+ success "推荐"

        ```python
        def connect_to_next_port(self, minimum):
            """Connects to the next available port.

            Args:
                minimum: A port value greater or equal to 1024.
            
            Returns:
                The new minimum port.
            
            Raises:
                ConnectionError: If no available port is found.
            """
            if minimum < 1024:
                # Note that this raising of ValueError is not mentioned in the doc
                # string's "Raises:" section because it is not appropriate to
                # guarantee this specific behavioral reaction to API misuse.
                raise ValueError(f'Min. port must be at least 1024, not {minimum}.')
            port = self._find_next_open_port(minimum)
            if not port:
                raise ConnectionError(
                    f'Could not connect to service on port {minimum} or higher.')
            assert port >= minimum, (
                f'Unexpected port {port} when minimum was {minimum}.')
            return port
        ```

    ???+ warning "不推荐"

        ```python
        def connect_to_next_port(self, minimum):
            """Connects to the next available port.

            Args:
                minimum: A port value greater or equal to 1024.
            
            Returns:
                The new minimum port.
            """
            assert minimum >= 1024, 'Minimum port must be at least 1024.'
            port = self._find_next_open_port(minimum)
            assert port is not None
            return port
        ```

- 模块或包应该定义自己的特定域的异常基类。这个基类应该从内建的 `Exception` 类继承。异常名称应该以 `Error` 结尾，而且不应该难以理解（`foo.FooError`）。
- 永远不要使用 `expect:` 语句来捕获所有异常，也不要捕获 `Exception` 或者   `StandardError`，除非：

    - 重新触发该异常，或
    - 你已经在当前线程的最外层（记得还是要打印一条错误消息）
  
  在异常这方面, Python 非常宽容， `expect:` 可以捕获所有拼写错误的名称， `sys.exit()` 调用， `Ctrl+C` 中断，`unittest` 失败和所有你不想捕获的其他异常。

- 尽量减少 `try/except` 块中的代码量。 `try` 块的体积越大，期望之外的异常就越容易被触发。在这些情况下，`try/except` 块将隐藏真正的错误。
- 使用 `finally` 子句来执行那些无论 `try` 块中有没有异常都应该被执行的代码。这对于清理资源常常很有用，例如关闭文件。

## 1.5 全局变量

避免全局变量。

### 1.5.1 定义

定义在模块级的变量。

### 1.5.2 优点

偶尔有用。

### 1.5.3 缺点

导入时可能改变模块行为，因为导入模块时会对模块级变量赋值。

### 1.5.4 结论

避免使用全局变量。

虽然模块级常量在技术上是变量，但是允许和鼓励使用。例如： `MAX_HOLY_HANDGRENADE_COUNT = 3` 。常量的命名必须使用全大写和下划线。具体请参阅命名规范。

如果需要，全局变量应该仅在模块内部可用，并通过在名称前加上 `_` 前缀使其成为模块的内部变量。外部访问必须通过模块级的公共函数来访问。具体请参阅命名规范。

## 1.6 嵌套/局部/内部类或函数

在需要关闭局部变量时鼓励使用嵌套本地内部类或函数，嵌套类更好。

### 1.6.1 定义

类可以定义在方法、函数或者类中。函数可以定义在方法或函数中。封闭区间中定义的变量对嵌套函数是只读的。

### 1.6.2 优点

允许定义仅用于有效范围的工具类和函数。非常像 [ADT](https://en.wikipedia.org/wiki/Abstract_data_type)-y 。通常用于实现装饰器。

### 1.6.3 缺点

- 不能直接测试嵌套函数和类。
- 嵌套会使外部函数更长。
- 可读性更差。

### 1.6.4 结论

可以使用，但有一些限制。避免使用嵌套函数或类，除非要关闭局部值。不要仅仅为了对用户隐藏模块的某个函数而进行嵌套。相反，应该在模块级别的名称上加 `_` 前缀，这样方便测试。

## 1.7 推导式和生成表达式

可以在简单情况下使用。

### 1.7.1 定义

`List`、`Dict` 和 `Set` 推导式与生成器表达式提供了一种简洁而有效的方法来创建列表和迭代器，而不必借助传统的循环、`map()`、`filter()` 或者 `lambda`。

### 1.7.2 优点

简单的推导式可以比其他的字典、列表或集合创建技术更加清晰简单。生成器表达式可以十分高效，因为它们避免了创建整个列表。

### 1.7.3 缺点

复杂的推导式或者生成器表达式可能难以阅读。

### 1.7.4 结论

适用于简单情况。每个部分应该单独置于一行：`mapping` 表达式，`for` 子句，`filter` 表达式。禁止多重 `for` 语句或过滤器表达式。复杂情况下还是使用循环。

???+ success "推荐"

    ```python
    result = [mapping_expr for value in iterable if filter_expr]

    result = [{'key': value} for value in iterable
              if a_long_filter_expression(value)]

    result = [complicated_transform(x)
              for x in iterable if predicate(x)]

    descriptive_name = [
        transform({'key': key, 'value': value}, color='black')
        for key, value in generate_iterable(some_input)
        if complicated_condition_is_met(key, value)
    ]

    result = []
    for x in range(10):
        for y in range(5):
            if x * y > 10:
                result.append((x, y))

    return {x: complicated_transform(x)
            for x in long_generator_function(parameter)
            if x is not None}

    squares_generator = (x**2 for x in range(10))

    unique_names = {user.name for user in users if user is not None}

    eat(jelly_bean for jelly_bean in jelly_beans
        if jelly_bean.color == 'black')
    ```

???+ warning "不推荐"

    ```python
    result = [complicated_transform(
              x, some_argument=x+1)
          for x in iterable if predicate(x)]

    result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]

    return ((x, y, z)
            for x in range(5)
            for y in range(5)
            if x != y
            for z in range(5)
            if y != z)
    ```

## 1.8 默认迭代器和操作符

如果类型支持，就使用默认迭代器和操作符。比如列表，字典及文件等。

### 1.8.1 定义

容器类型，像字典和列表，定义了默认的迭代器和关系测试操作符（ `in` 和 `not in` ）

### 1.8.2 优点

默认操作符和迭代器简单高效，它们直接表达了操作，没有额外的方法调用。使用默认操作符的函数是通用的。它可以用于支持该操作的任何类型。

### 1.8.3 缺点

你没法通过阅读方法名来区分对象的类型（例如，`has_key()` 意味着字典）。不过这也是优点。

### 1.8.4 结论

如果类型支持，就使用默认迭代器和操作符，例如列表、字典和文件。内建类型也定义了迭代器方法。优先考虑这些方法，而不是那些返回列表的方法。当然，这样遍历容器时，你将不能修改容器。除非特殊情况，否则不要使用 Python 2 特定的迭代方法 `dict.iter*()` 。

???+ success "推荐"

    ```python
    for key in adict: ...
    if key not in adict: ...
    if obj in alist: ...
    for line in afile: ...
    for k, v in adict.items(): ...
    for k, v in six.iteritems(adict): ...
    ```
  
???+ warning "不推荐"

    ```python
    for key in adict.keys(): ...
    if not adict.has_key(key): ...
    for line in afile.readlines(): ...
    for k, v in dict.iteritems(): ...
    ```

## 1.9 生成器

按需使用生成器。

### 1.9.1 定义

所谓生成器函数，就是每当它执行一次生成 `yield` 语句，它就返回一个迭代器，这个迭代器生成一个值。生成值后，生成器函数的运行状态将被挂起，直到下一次生成。

### 1.9.2 优点

简化代码，因为每次调用时，局部变量和控制流的状态都会被保存。比起一次创建一系列值的函数，生成器使用的内存更少。

### 1.9.3 缺点

没有。

### 1.9.4 结论

鼓励使用。注意在生成器函数的文档字符串中使用“`Yields:`”而不是“`Returns:`”。

## 1.10 Lambda 函数

适用于单行函数。

### 1.10.1 定义

与语句相反，`Lambdas` 在一个表达式中定义匿名函数。常用于为 `map()` 和 `filter()` 之类的高阶函数定义回调函数或者操作符。

### 1.10.2 优点

方便。

### 1.10.3 缺点

比本地函数更难阅读和调试。没有函数名意味着堆栈跟踪更难理解。由于 `lambda` 函数通常只包含一个表达式，因此其表达能力有限。

### 1.10.4 结论

适用于单行函数。如果代码超过60-80个字符，最好还是定义成常规（嵌套）函数。

对于常见的操作符，例如乘法操作符，使用 `operator` 模块中的函数以代替 `lambda` 函数。例如，推荐使用 `operator.mul` 而不是 `lambda x, y: x * y` 。

## 1.11 条件表达式

适用于单行函数。

### 1.11.1 定义

条件表达式是对于 `if` 语句的一种更为简短的句法规则。例如 `x = 1 if cond else 2` 。

### 1.11.2 优点

比 `if` 语句更加简短和方便。

### 1.11.2 缺点

比 `if` 语句难于阅读。如果表达式很长，难于定位条件。

### 1.11.4 结论

适用于单行函数。每个部分必须放在一行上： `true-expression, if-expression, else-expression` 。在其他情况下，推荐使用完整的 `if` 语句。

```python
one_line = 'yes' if predicate(value) else 'no'
slightly_split = ('yes' if predicate(value)
                  else 'no, nein, nyet')
the_longest_ternary_style_that_can_be_done = (
    'yes, true, affirmative, confirmed, correct'
    if predicate(value)
    else 'no, false, negative, nay')
```


```python
bad_line_breaking = ('yes' if predicate(value) else
                     'no')
portion_too_long = ('yes'
                    if some_long_module.some_long_predicate_function(
                        really_long_variable_name)
                    else 'no, false, negative, nay')
```


## 1.12 默认参数值

适用于大部分情况。

### 1.12.1 定义

你可以在函数参数列表的最后指定变量的值，例如， `def(a, b=0):` 。如果调用 `foo` 时只带一个参数，则 `b` 被设为 `0` ，如果带两个参数，则 `b` 的值等于第二个参数。

### 1.12.2 优点

你经常会碰到一些使用大量默认值的函数，但偶尔（比较少见）你想要覆盖这些默认值。默认参数值提供了一种简单的方法来完成这件事，你不需要为这些罕见的例外定义大量函数。同时， Python 也不支持重载方法和函数，默认参数是一种“模拟”重载行为的简单方式。

### 1.12.3 缺点

默认参数只在模块加载时求值一次。如果参数是列表或字典之类的可变类型，这可能会导致问题。如果函数修改了对象（例如，向列表追加项），默认值就被修改了。

### 1.12.4 结论

鼓励使用，不要在函数或方法定义中使用可变对象作为默认值。

???+ success "推荐"

    ```python
    def foo(a, b=None):
        if b is None:
            b = []
    ```

    ```python
    def foo(a, b: Optional[Sequence] = None):
        if b is None:
            b = []
    ```

    ```python
    def foo(a, b: Sequence = ()):  # Empty tuple OK since tuples are immutable
        ...
    ```

???+ warning "不推荐"

    ```python
    def foo(a, b=[]):
         ...
    ```

    ```python
    def foo(a, b=time.time()):  # The time the module was loaded???
         ...
    ```

    ```python
    def foo(a, b=FLAGS.my_thing):  # sys.argv has not yet been parsed...
         ...
    ```

    ```python
    def foo(a, b: Mapping = {}):  # Could still get passed to unchecked code
         ...
    ```

## 1.13 属性（properties）

访问和设置数据成员时，你通常会使用简单，轻量级的访问和设置函数。建议用属性（properties）来代替它们。

### 1.13.1 定义

一种用于包装方法调用的方式。当运算量不大，它是获取和设置属性的标准方式。

### 1.13.2 优点

通过消除简单的属性访问时显式的 `get` 和 `set` 方法调用，可读性提高了。允许延迟加载。用 `Pythonic` 的方式来维护类的接口。就性能而言，当直接访问变量是合理的，添加访问方法就显得琐碎而无意义。使用属性可以绕过这个问题。将来也可以在不破坏接口的情况下将访问方法加上。

### 1.13.3 缺点

- 会隐藏类似操作符重载的副作用。
- 对于子类可能会造成混淆。

### 1.13.4 结论

你通常习惯于使用访问或设置方法来访问或设置数据，它们简单而轻量。不过我们建议你在新的代码中使用属性。只读属性应该用 `@property` 装饰器来创建。

如果子类没有覆盖属性，那么属性的继承可能看上去不明显。因此使用者必须确保访问方法间接被调用，以保证子类中的重载方法被属性调用（使用[模板方法设计模式](https://en.wikipedia.org/wiki/Template_method_pattern)）。

???+ success "推荐"

    ```python
    import math

    class Square:
        """A square with two properties: a writable area and a read-only perimeter.

        To use:
        >>> sq = Square(3)
        >>> sq.area
        9
        >>> sq.perimeter
        12
        >>> sq.area = 16
        >>> sq.side
        4
        >>> sq.perimeter
        16
        """

        def __init__(self, side):
            self.side = side

        @property
        def area(self):
            """Area of the square."""
            return self._get_area()

        @area.setter
        def area(self, area):
            return self._set_area(area)

        def _get_area(self):
            """Indirect accessor to calculate the 'area' property."""
            return self.side ** 2

        def _set_area(self, area):
            """Indirect setter to set the 'area' property."""
            self.side = math.sqrt(area)

        @property
        def perimeter(self):
            return self.side * 4
    ```

## 1.14 `True` / `False` 的求值

尽可能使用隐式 `False` 。

### 1.14.1 定义

Python 在布尔上下文中会将某些值求值为 `False` 。按简单的直觉来讲，就是所有的空值都被认为是 `False`，因此 `0`, `None`，`[]`，`{}`，`''` 都被认为是 `False`。

### 1.14.2 优点

使用 Python 布尔值的条件语句更易读也更不易犯错。大部分情况下，也更快。

### 1.14.3 缺点

对于 C / C ++ 开发人员来说，可能看起来有点怪。

### 1.14.4 结论

尽可能使用隐式的 `false`，例如：使用 `if foo:` 而不是 `if foo !=[]:`。不过还是有一些注意事项需要你铭记在心：

- 总是使用 `if foo is None:` 或 `if foo is not None:` 来检查 `None` 值。例如，当你要测试一个默认值是 `None` 的变量或参数是否被设为其它值。这个值在布尔语义下可能是 `false`!
- 永远不要用 `==` 将一个布尔量与 `False` 相比较。使用 `if not x:` 代替。如果你需要区分 `False` 和 `None` ，你应该用像 `if not x and x is not None:` 这样的语句。
- 对于序列（字符串、列表、元组）， 要注意空序列是 `False` 。因此： `if seq:` 或者 `if not seq:` 比 `if len(seq):` 或 `if not len(seq)` 要更好。
- 处理整数时，使用隐式 `False` 可能会得不偿失（即不小心将 `None` 当做 `0` 来处理）。你可以将一个已知是整型（且不是 `len()` 的返回结果）的值与 `0` 比较。

    ???+ success "推荐"

        ```python
        if not users:
            print('no users')

        if foo == 0:
            self.handle_zero()

        if i % 10 == 0:
            self.handle_multiple_of_ten()

        def f(x=None):
            if x is None:
                x = []
        ```

    ???+ warning "不推荐"

        ```python
        if len(users) == 0:
            print('no users')

        if foo is not None and not foo:
            self.handle_zero()

        if not i % 10:
            self.handle_multiple_of_ten()

        def f(x=None):
            x = x or []
        ```

- 注意： `'0'` （即： `0` 作为字符串）的计算结果是 `True` 。

## 1.16 词法作用域（Lexical Scoping）

推荐使用

### 1.16.1 定义

嵌套的 Python 函数可以引用外层函数中定义的变量，但是不能够对它们赋值。变量绑定的解析是使用词法作用域，也就是基于静态的程序文本。对一个块中的某个名称的任何赋值都会导致 Python 将对该名称的全部引用当做局部变量，甚至是赋值前的处理。如果碰到 `global` 声明，该名称就会被视作全局变量。

一个使用这个特性的例子：

```python
def get_adder(summand1):
    """Returns a function that adds numbers to a given number."""
    def adder(summand2):
        return summand1 + summand2

    return adder
```

### 1.16.2 优点

通常可以带来更加清晰，优雅的代码。尤其会让有经验的 Lisp 和 Scheme （还有 Haskell， ML 等）程序员感到欣慰。

### 1.16.3 缺点

可能导致让人迷惑的 bug。例如下面这个依据 [PEP-0227](http://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0227/) 的例子：

```python
i = 4
def foo(x):
    def bar():
        print(i, end='')
    # ...
    # A bunch of code here
    # ...
    for i in x:  # Ah, i *is* local to foo, so this is what bar sees
        print(i, end='')
    bar()
```

因此 `foo([1, 2, 3])` 会打印 `1 2 3 3` 而不是 `1 2 3 4`

### 1.16.4 结论

鼓励使用。

## 1.17 函数与方法装饰器

如果好处很显然，就明智而谨慎的使用装饰器。避免使用 `staticmethod` ，限制使用 `classmethod`。

### 1.17.1 定义

用于[函数及方法的装饰器](https://docs.python.org/3/glossary.html#term-decorator)（也就是 `@` 标记）。最常见的装饰器是 `@property` ，用于将普通方法转换为动态运算的属性。不过，装饰器语法也允许用户自定义装饰器。特别地，对于某个函数 `my_decorator`，下面的两段代码是等效的：

```python
class C:
    @my_decorator
    def method(self):
        # method body ...
```

相当于：

```python
class C:
    def method(self):
        # method body ...
    method = my_decorator(method)
```

### 1.17.2 优点

优雅的在函数上指定一些转换。该转换可能减少一些重复代码，保持已有函数不变（enforce invariants)）等。

### 1.17.3 缺点

装饰器可以在函数的参数或返回值上执行任何操作，这可能导致让人惊异的隐藏行为。而且，饰器在导入时执行。从装饰器代码的失败中恢复更加不可能。

### 1.17.4 结论

- 如果好处很显然，就明智而谨慎的使用装饰器。
- 装饰器应该遵守和函数一样的导入和命名规则。
- 装饰器的 Python 文档应该清晰的说明该函数是一个装饰器。
- 请为装饰器编写单元测试。

避免装饰器自身对外界的依赖（即不要依赖于文件，`socket`，数据库连接等），因为装饰器运行时这些资源可能不可用（由 `pydoc` 或其它工具导入）。应该保证一个用有效参数调用的装饰器在所有情况下都是成功的。

装饰器是一种特殊形式的“顶级代码”。参考后面关于 [Main](https://google.github.io/styleguide/pyguide.html#s3.17-main) 的话题。

永远不要使用 `staticmethod` ，除非为了与现有库中定义的 API 集成而被迫使用。可以写一个模块级函数代替。

只有在编写命名构造函数或修改必要的全局状态（如进程级缓存）的特定类操作时才使用 `classmethod`。

## 1.18 线程

不要依赖内建类型的原子性。

虽然 Python 的内建类型例如字典看上去拥有原子操作，但是在某些情形下它们仍然不是原子的（即，如果 `__hash__` 或 `__eq__` 被实现为 Python 方法）且它们的原子性是靠不住的。你也不能指望原子变量赋值（因为这个反过来依赖字典）。

优先使用 `Queue` 模块的 `Queue` 数据类型作为线程间的数据通信方式。另外，使用 `threading` 模块及其锁原语（`locking primitives`）。了解条件变量的合适使用方式，这样你就可以使用 `threading.Condition` 来取代低级别的锁了。

## 1.19 威力过大的特性

避免使用这些特性。

### 1.19.1 定义

Python 是一种异常灵活的语言，它为你提供了很多花哨的特性，诸如元类（`metaclasses`）、字节码访问、任意编译（`on-the-fly compilation`）、动态继承、对象父类重定义（`object reparenting`）、导入修改（`import hacks`）、反射（例如 `getattr()` 的一些使用）、系统内修改（`modification of system internals`）等等。

### 1.19.2 优点

强大的语言特性，能让你的代码更紧凑。

### 1.19.3 缺点

使用这些很“酷”的特性十分诱人，但不是绝对必要。使用奇技淫巧的代码将更加难以阅读和调试。开始可能还好（对原作者而言）, 但当你回顾代码, 它们可能会比那些稍长一点但是很直接的代码更加难以理解.

### 1.19.4 结论

在你的代码中避免这些特性。

内部需要使用这些特性的标准库模块和类可以使用（例如， `abc.ABCMeta` 、 `dataclasses` 和 `enum`）。

## 1.20 新版 Python ： Python 3 和 `from __future__ imports`

当前推荐 Python 3 。虽然不是每个项目都必须使用，但所有编写的代码都应该兼容 Python 3 （并尽可能的通过 Python 3 的测试）。

### 1.20.1 定义

Python 3 是 Python 语言的重大变化。虽然现有的代码通常是考虑用 Python 2.7 编写的，但是可以做一些简单的事情来使代码更明确地表达其意图，从而无需修改就能更好地在 Python 3 下使用。

### 1.20.2 优点

当所有项目依赖都兼容 Python 3 的时候，就可以直接编写 Python 3 的代码了，这样也更容易在 Python 3 中运行。

### 1.20.3 缺点

- 觉得这些额外的样板文件很难看。
- 导入实际并不需要的特性模块看起来怪怪的。

### 1.20.4 结论

**`from __future__ imports`**

推荐使用 `from __future__ import` 语句。所有的新代码都应该包含以下内容，现有的代码也应该在有条件的情况下进行兼容更新：

```python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function
```

获取有关这些导入的更多信息，请参阅 [absolute imports](https://www.python.org/dev/peps/pep-0328/) 、 [division behavior](https://google.github.io/styleguide/pyguide.html)
 和 [the print function](https://www.python.org/dev/peps/pep-3105/)。

即使当前在模块中没有使用这些导入，也请不要忽略或删除它们，除非代码仅适用于 Python 3 版本。最好始终在所有文件中包含 `future` 的导入，以便在有人开始使用这些特性时，不会在以后的编辑中忘记它们。

还有一些其他的 `from __future__` 语句，可以在需要的时候使用。我们在推荐中并没有包括 `unicode_literals` ，那是因为只有在 Python 2.7 中才会引入许多隐式的默认编码转换。大多数代码最好根据需要明确使用 `b''` 和 `u''` 字符以及 `unicode` 字符串。

**`six` 、 `future` 和 `past`**

当项目同时需要支持 Python 2 和 3 版本时，可以使用 [`six`](https://pypi.org/project/six/) 、 [`future`](https://pypi.org/project/future/) 和 [`past`](https://pypi.org/project/past/) 。这些库就是为了让代码实现更清晰简单而存在的。

## 1.21 代码类型标注

Python 3 的代码可以根据 [PEP-484](https://www.python.org/dev/peps/pep-0484/) 使用类型标注，并使用类似 [pytype](https://github.com/google/pytype) 的类型检查工具在构建时对代码进行检查。

类型标注可以在原文件中，也可以在 [stub pyi](https://www.python.org/dev/peps/pep-0484/#stub-files) 文件中。尽可能在源代码中进行标注，对于第三方库或扩展模块可以使用 `pyi` 文件。

### 1.21.1 定义

类型标注（或类型提示）可以用于函数或方法的参数和返回值

```python
def func(a: int) -> List[int]:
```

还可以使用类似 [PEP-526](https://www.python.org/dev/peps/pep-0526/) 的语法声明变量的类型：

```python
a: SomeType = some_func()
```

或者在必须支持旧版 Python 版本的代码中使用类型注释。

```python
a = some_func()  # type: SomeType
```

### 1.21.2 优点

类型标注可以提高代码的可读性和可维护性。类型检查器可以把许多运行时错误转换为构建时错误，并减少威力过大特性地使用。

### 1.21.3 缺点

- 必须保持类型标注更新。
- 您可能会看到您认为是正确代码的错误信息。
- 使用类型检查器可能会减少威力过大特性地使用。
  
### 1.21.4 结论

强烈建议您在更改代码时启用 Python 类型分析。当添加或修改公共 API 时，请包含类型标注，并在构建系统中启用 `pytype` 进行检查。由于静态分析对 Python 来说相对较新，我们承认会有一些副作用（比如错误的类型推断）可能会阻止一些项目采用。因此，我们鼓励作者添加一个带有 `TODO` 的注释，或者在 `BUILD` 文件或代码本身中通过 bug 链接描述当前不采用类型标注的问题。

