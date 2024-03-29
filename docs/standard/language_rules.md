# Python 语言规范

> 本文档为 [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)
> 第二章 [Python Language Rules](https://google.github.io/styleguide/pyguide.html#2-python-language-rules) 的译文。
>
> 最后更新时间： 2023-06-26
>
> 如果有翻译错误或表述不准确的问题，欢迎提交 PR，感谢您的参与。

## 1.1 Lint

使用 [pylintrc](https://google.github.io/styleguide/pylintrc) 配置，对你的代码运行 `pylint`。

### 1.1.1 定义

`Pylint` 是一个在 Python 源代码中查找 bug 和风格问题的工具。对于 C 和 C++ 这样的不那么动态的语言，这些问题通常由编译器来捕获。由于
Python 的动态特性，有些警告可能不对。不过伪告警应该很少。

### 1.1.2 优点

可以捕获容易忽视的错误，例如输入错误，使用未赋值的变量等。

### 1.1.3 缺点

`pylint` 不完美。要利用其优势，我们有时侯需要：围绕着它来写代码、抑制其告警、改进它或者忽略它。

### 1.1.4 结论

确保对你的代码运行 `pylint`。

抑制不准确的警告，以便能够将其他警告暴露出来。你可以通过设置一个行注释来抑制告警：

```python
def do_PUT(self):  # WSGI name, so pylint: disable=invalid-name
    ...
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
pylint --help-msg=invalid-name
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

只对包和模块使用 `import` 语句，而不是单独的类或函数。

### 1.2.1 定义

模块间共享代码的重用机制。

### 1.2.2 优点

命名空间管理约定十分简单。每个标识符的来源都用一种一致的方式指示：`x.Obj` 表示 `Obj` 对象定义在模块 `x` 中。

### 1.2.3 缺点

模块名仍可能冲突。有些模块名太长, 不太方便。

### 1.2.4 结论

- 使用 `import x` 来导入包和模块。
- 使用 `from x import y`，其中 `x` 是包前缀，`y` 是不带前缀的模块名。
- 在以下任一情况下使用 `from x import y as z`：

    - 名字都为 `y` 的模块。
    - `y` 与当前模块中顶级名称冲突。
    - `y` 与作为公共 API 一部分的公共参数名称（例如功能）冲突。
    - `y` 是一个长名称，使用不太方便。
    - `y` 在代码上下文中过于通用（例如：`from storage.file_system import options as fs_options`）

- 只有当 `z` 是标准缩写（例如，`numpy` 为 `np`）时，才使用 `import y as z`。

例如，模块 `sound.effects.echo` 可以用如下方式导入：

```python
from sound.effects import echo

...
echo.EchoFilter(input, output, delay=0.7, atten=4)
```

导入时不要使用相对名称。即使模块在同一个包中，也要使用完整包名。这能帮助你避免无意间导入一个包两次。

以下规则不受约束：

- 用于支持静态分析和类型检查：

    - [typing](https://google.github.io/styleguide/pyguide.html#typing-imports)
    - [collections.abc](https://google.github.io/styleguide/pyguide.html#typing-imports)
    - [typing_extensions](https://github.com/python/typing_extensions/blob/main/README.md)  

- 重定向模块 [Six.moves](https://six.readthedocs.io/#module-six.moves)

## 1.3 包

使用模块的全路径名来导入每个模块。

### 1.3.1 优点

避免模块名称冲突或因模块搜索路径与作者预期不符而导致的错误导入。查找包更容易。

### 1.3.2 缺点

因为要复制包层次结构，所以在部署代码时会更加困难。但是对于现代的部署机制来说，这并不是真正的问题。

### 1.3.3 结论

所有的新代码都应该用完整包名来导入每个模块。

应该像下面这样导入：

!!! success "推荐"

    ```python
    # Reference absl.flags in code with the complete name (verbose).
    import absl.flags
    from doctor.who import jodie

    _FOO = absl.flags.DEFINE_string(...)
    ```

!!! success "推荐"

    ```python
    # Reference flags in code with just the module name (common).
    from absl import flags
    from doctor.who import jodie

    _FOO = flags.DEFINE_string(...)
    ```

!!! fail "不推荐 ( 假设 `jodie.py` 文件在 `doctor/who/` 中 )"

    ```python
    # Unclear what module the author wanted and what will be imported.  The actual
    # import behavior depends on external factors controlling sys.path.
    # Which possible jodie module did the author intend to import?
    import jodie
    ```

尽管在某些环境中会发生这种情况，但不应假定主二进制文件所在的目录位于 `sys.path` 中。在这种情况下，代码应假定 `import jodie`
引用了名为 `jodie` 的第三方或顶级程序包，而不是本地的 `jodie.py`。

## 1.4 异常

允许使用异常，但必须小心。

### 1.4.1 定义

异常是一种跳出代码块的正常控制流来处理错误或者其它异常条件的方式。

### 1.4.1 优点

正常操作代码的控制流不会和错误处理代码混在一起。当某种条件发生时，它也允许控制流跳过多个框架。例如，一步跳出 N
个嵌套的函数，而不必继续执行错误的代码。

### 1.4.2 缺点

可能会导致让人困惑的控制流。调用库时容易错过错误情况。

### 1.4.4 结论

异常必须遵守特定条件：

- 如果有必要，请使用内置异常类。例如，抛出 `ValureError`
  来指示编程错误。比如违反了前置条件（需要一个正数，但传递了一个负数）。不要使用 `assert` 语句验证公共 API 的参数值。`assert`
  用于确保内部正确性，不得强制使用，也不表示发生了某些意外事件。如果在后一种情况下需要使用异常，请使用 raise 语句。例如：  

    !!! success "推荐"

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

    !!! fail "不推荐"

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

- 模块或包应该定义自己的特定域的异常基类。这个基类应该从内建的 `Exception` 类继承。异常名称应该以 `Error`
  结尾，而且不应该难以理解（`foo.FooError`）。
- 永远不要使用 `expect:` 语句来捕获所有异常，也不要捕获 `Exception` 或者   `StandardError`，除非：

    - 重新触发该异常，或
    - 在程序中创建一个隔离点，其中异常不会传播，而是被记录和抑制，例如通过保护线程的最外层块来防止程序崩溃。

  在异常这方面, Python 非常宽容， `expect:` 可以捕获所有拼写错误的名称， `sys.exit()` 调用， `Ctrl+C` 中断，`unittest`
  失败和所有你不想捕获的其他异常。

- 尽量减少 `try/except` 块中的代码量。 `try` 块的体积越大，期望之外的异常就越容易被触发。在这些情况下，`try/except`
  块将隐藏真正的错误。
- 使用 `finally` 子句来执行那些无论 `try` 块中有没有异常都应该被执行的代码。这对于清理资源常常很有用，例如关闭文件。

## 1.5 全局变量

避免全局变量。

### 1.5.1 定义

定义在模块级的变量。

### 1.5.2 优点

偶尔有用。

### 1.5.3 缺点

- 破坏封装：这种设计可能会让有效目标的实现变得困难。例如使用全局状态来管理数据库连接，则同时连接两个不同的数据库变得困难（例如再迁移期间
计算差异）。全局注册表也容易出现类似的问题。
- 导入时可能改变模块行为，因为导入模块时会对模块级变量赋值。

### 1.5.4 结论

避免使用全局变量。

如果需要，全局变量应该仅在模块内部可用，并通过在名称前加上 `_` 前缀使其成为模块的内部变量。外部访问必须通过模块级的公共函数来访问。具体请参阅命名规范。请在注释或与注释相关的文档中说明使用可变全局状态的设计原因。

模块级常量是允许和鼓励使用的。例如： `_MAX_HOLY_HANDGRENADE_COUNT = 3` (对内部使用常量)或 `SIR_LANCELOTS_FAVORITE_COLOR = "blue"`(对公共 API 常量)。常量的命名必须使用全大写和下划线。具体请参阅命名规范。

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

可以使用，但有一些限制。避免使用嵌套函数或类，除非要关闭局部值。不要仅仅为了对用户隐藏模块的某个函数而进行嵌套。相反，应该在模块级别的名称上加 `_`
前缀，这样方便测试。

## 1.7 推导式和生成表达式

可以在简单情况下使用。

### 1.7.1 定义

`List`、`Dict` 和 `Set` 推导式与生成器表达式提供了一种简洁而有效的方法来创建列表和迭代器，而不必借助传统的循环、`map()`
、`filter()` 或者 `lambda`。

### 1.7.2 优点

简单的推导式可以比其他的字典、列表或集合创建技术更加清晰简单。生成器表达式可以十分高效，因为它们避免了创建整个列表。

### 1.7.3 缺点

复杂的推导式或者生成器表达式可能难以阅读。

### 1.7.4 结论

适用于简单情况。每个部分应该单独置于一行：`mapping` 表达式，`for` 子句，`filter` 表达式。禁止多重 `for`
语句或过滤器表达式。复杂情况下还是使用循环。

!!! success "推荐"

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

!!! fail "不推荐"

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

您无法通过读取方法名称来判断对象的类型（除非变量具有类型注释）。这也是一个优点。

### 1.8.4 结论

如果类型支持，就使用默认迭代器和操作符，例如列表、字典和文件。内建类型也定义了迭代器方法。优先考虑这些方法，而不是那些返回列表的方法。当然，这样遍历容器时，你将不能修改容器。

!!! success "推荐"

    ```python
    for key in adict: ...
    if obj in alist: ...
    for line in afile: ...
    for k, v in adict.items(): ...
    ```

!!! fail "不推荐"

    ```python
    for key in adict.keys(): ...
    for line in afile.readlines(): ...
    ```

## 1.9 生成器

按需使用生成器。

### 1.9.1 定义

所谓生成器函数，就是每当它执行一次生成 `yield` 语句，它就返回一个迭代器，这个迭代器生成一个值。
生成值后，生成器函数的运行状态将被挂起，直到需要下一次值为止。

### 1.9.2 优点

简化代码，因为每次调用时，局部变量和控制流的状态都会被保存。比起一次创建一系列值的函数，生成器使用的内存更少。

### 1.9.3 缺点

生成器中的局部变量不会被垃圾收集，直到生成器耗尽或本身被垃圾收集。

### 1.9.4 结论

鼓励使用。注意在生成器函数的文档字符串中使用“`Yields:`”而不是“`Returns:`”。

如果生成器管理着一个昂贵的资源，请确保强制进行清理。

一个很好的清理方式是使用上下文管理器 [PEP-0533](https://peps.python.org/pep-0533/) 来包装生成器。

## 1.10 Lambda 函数

适用于单行函数。常用于为 `map()` 和 `filter()` 之类的高阶函数定义回调函数或者操作符。

### 1.10.1 定义

`Lambdas` 在一个表达式中定义匿名函数，而不是在语句中。

### 1.10.2 优点

方便。

### 1.10.3 缺点

比本地函数更难阅读和调试。没有函数名意味着堆栈跟踪更难理解。由于 `lambda` 函数通常只包含一个表达式，因此其表达能力有限。

### 1.10.4 结论

适用于单行函数。如果代码超过60-80个字符，最好还是定义成常规（嵌套）函数。

对于常见的操作符，例如乘法操作符，使用 `operator` 模块中的函数以代替 `lambda` 函数。例如，推荐使用 `operator.mul`
而不是 `lambda x, y: x * y` 。

## 1.11 条件表达式

适用于单行函数。

### 1.11.1 定义

条件表达式是对于 `if` 语句的一种更为简短的句法规则。例如 `x = 1 if cond else 2` 。

### 1.11.2 优点

比 `if` 语句更加简短和方便。

### 1.11.2 缺点

比 `if` 语句难于阅读。如果表达式很长，难于定位条件。

### 1.11.4 结论

适用于单行函数。每个部分必须放在一行上： `true-expression, if-expression, else-expression`
。在其他情况下，推荐使用完整的 `if` 语句。

!!! success "推荐"

    ```python
    one_line = 'yes' if predicate(value) else 'no'
    slightly_split = ('yes' if predicate(value) else 'no, nein, nyet')
    the_longest_ternary_style_that_can_be_done = (
        'yes, true, affirmative, confirmed, correct'
        if predicate(value) else 'no, false, negative, nay')
    ```

!!! fail "不推荐"

    ```python
    bad_line_breaking = ('yes' if predicate(value) else 'no')
    portion_too_long = ('yes' if some_long_module.some_long_predicate_function(
                            really_long_variable_name) else 'no, false, negative, nay')
    ```

## 1.12 默认参数值

适用于大部分情况。

### 1.12.1 定义

你可以在函数参数列表的最后指定变量的值，例如， `def(a, b=0):` 。如果调用 `foo` 时只带一个参数，则 `b` 被设为 `0`，如果带两个参数，则 `b` 的值等于第二个参数。

### 1.12.2 优点

你经常会碰到一些使用大量默认值的函数，但偶尔（比较少见）你想要覆盖这些默认值。默认参数值提供了一种简单的方法来完成这件事，你不需要为这些罕见的例外定义大量函数。同时，
Python 也不支持重载方法和函数，默认参数是一种“模拟”重载行为的简单方式。

### 1.12.3 缺点

默认参数只在模块加载时求值一次。如果参数是列表或字典之类的可变类型，这可能会导致问题。如果函数修改了对象（例如，向列表追加项），默认值就被修改了。

### 1.12.4 结论

鼓励使用，不要在函数或方法定义中使用可变对象作为默认值。

!!! success "推荐"

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

!!! fail "不推荐"

    ```python
    def foo(a, b=[]):
         ...
    ```

    ```python
    def foo(a, b=time.time()):  # The time the module was loaded???
         ...
    ```

    ```python
    from absl import flags
    _FOO = flags.DEFINE_string(...)

    def foo(a, b=_FOO.value):  # sys.argv has not yet been parsed...
         ...
    ```

    ```python
    def foo(a, b: Mapping = {}):  # Could still get passed to unchecked code
         ...
    ```

## 1.13 属性（properties）

属性（properties）可以用于控制需要进行简单计算或逻辑的属性的获取或设置。其实现必须符合常规属性访问的一般期望：它们应该是简单，直接，且易于理解。

### 1.13.1 定义

一种用于包装方法调用的方式。当运算量不大，它是获取和设置属性的标准方式。

### 1.13.2 优点

- 允许进行属性访问和赋值的 API，而不是调用 `getter` 和 `setter` 方法。
- 可使得属性为只读。
- 允许延迟加载。
- 提供了一种方式，在类的内部与外部独立演化时，仍然能够保持类的公共接口不变。

### 1.13.3 缺点

- 会隐藏类似操作符重载的副作用。
- 对于子类可能会造成混淆。

### 1.13.4 结论

与运算符重载一样，在必要的情况下可以使用属性，并且应符合典型属性访问的预期；否则，请遵循 `getters` 和 `setters` 规则进行操作。

例如，使用属性同时获取和设置内部属性是不允许的：因为没有进行任何计算，所以属性是不必要的（可以将属性公开而不用使用属性）。相比之下，使用属性来控制属性访问或计算一个很容易得出的值是被允许的：因为逻辑简单，且易于理解。

应该使用 `@property` [装饰器](https://google.github.io/styleguide/pyguide.html#s2.17-function-and-method-decorators)创建属性。手动实现属性描述符被认为是威力过大的特性。

属性的继承可能是不明显的。不要使用属性来实现子类可能想要重写和扩展的计算。

## 1.14 `True` / `False` 的求值

尽可能使用隐式 `False` 。

### 1.14.1 定义

Python 在布尔上下文中会将某些值求值为 `False` 。按简单的直觉来讲，就是所有的空值都被认为是 `False`，因此 `0`, `None`，`[]`
，`{}`，`''` 都被认为是 `False`。

### 1.14.2 优点

使用 Python 布尔值的条件语句更易读也更不易犯错。大部分情况下，也更快。

### 1.14.3 缺点

对于 C / C ++ 开发人员来说，可能看起来有点怪。

### 1.14.4 结论

尽可能使用隐式的 `false`，例如：使用 `if foo:` 而不是 `if foo !=[]:`。不过还是有一些注意事项需要你铭记在心：

- 总是使用 `if foo is None:` 或 `if foo is not None:` 来检查 `None` 值。例如，当你要测试一个默认值是 `None`
  的变量或参数是否被设为其它值。这个值在布尔语义下可能是 `false`!
- 永远不要用 `==` 将一个布尔量与 `False` 相比较。使用 `if not x:` 代替。如果你需要区分 `False` 和 `None`
  ，你应该用像 `if not x and x is not None:` 这样的语句。
- 对于序列（字符串、列表、元组）， 要注意空序列是 `False` 。因此： `if seq:` 或者 `if not seq:` 比 `if len(seq):`
  或 `if not len(seq)` 要更好。
- 处理整数时，使用隐式 `False` 可能会得不偿失（即不小心将 `None` 当做 `0` 来处理）。你可以将一个已知是整型（且不是 `len()`
  的返回结果）的值与 `0` 比较。

    !!! success "推荐"

        ```python
        if not users:
            print('no users')

        if i % 10 == 0:
            self.handle_multiple_of_ten()

        def f(x=None):
            if x is None:
                x = []
        ```

    !!! fail "不推荐"

        ```python
        if len(users) == 0:
            print('no users')

        if not i % 10:
            self.handle_multiple_of_ten()

        def f(x=None):
            x = x or []
        ```

- 注意： `'0'` （即： `0` 作为字符串）的计算结果是 `True` 。
- 注意： Numpy 数组可能会在隐式布尔上下文中引发异常。测试一组 `np.array` 为空首选 `.size` 属性 （例如 `if not users.size`）。

## 1.16 词法作用域（Lexical Scoping）

推荐使用

### 1.16.1 定义

嵌套的 Python 函数可以引用外层函数中定义的变量，但是不能够对它们赋值。变量绑定的解析是使用词法作用域，也就是基于静态的程序文本。
对一个块中的某个名称的任何赋值都会导致Python 将对该名称的全部引用当做局部变量，甚至是赋值前的处理。
如果碰到 `global` 声明，该名称就会被视作全局变量。

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

可能导致让人迷惑的
bug。例如下面这个依据 [PEP-0227](http://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0227/) 的例子：

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

当有明显优势时，就明智而谨慎的使用装饰器。避免使用 `staticmethod` ，限制使用 `classmethod`。

### 1.17.1 定义

用于[函数及方法的装饰器](https://docs.python.org/3/glossary.html#term-decorator)（也就是 `@`标记）。
最常见的装饰器是 `@property`，用于将普通方法转换为动态运算的属性。不过，装饰器语法也允许用户自定义装饰器。
特别地，对于某个函数 `my_decorator`，下面的两段代码是等效的：

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

装饰器可以在函数的参数或返回值上执行任何操作，这可能导致让人惊异的隐藏行为。此外，装饰器在对象定义时执行。
对于模块级别的对象（类、模块函数等），此过程发生在导入时。从装饰器代码的失败中恢复更加不可能。

### 1.17.4 结论

- 如果好处很显然，就明智而谨慎的使用装饰器。
- 装饰器应该遵守和函数一样的导入和命名规则。
- 装饰器的 Python 文档应该清晰的说明该函数是一个装饰器。
- 请为装饰器编写单元测试。

避免装饰器自身对外界的依赖（即不要依赖于文件，`socket`，数据库连接等），因为装饰器运行时这些资源可能不可用（由 `pydoc`
或其它工具导入）。应该保证一个用有效参数调用的装饰器在所有情况下都是成功的。

装饰器是一种特殊形式的“顶级代码”。参考 [Main](https://google.github.io/styleguide/pyguide.html#s3.17-main) 的话题。

永远不要使用 `staticmethod` ，除非为了与现有库中定义的 API 集成而被迫使用。可以写一个模块级函数代替。

只有在编写命名构造函数或修改必要的全局状态（如进程级缓存）的特定类操作时才使用 `classmethod`。

## 1.18 线程

不要依赖内建类型的原子性。

虽然 Python 的内建类型例如字典看上去拥有原子操作，但是在某些情形下它们仍然不是原子的（即，如果 `__hash__` 或 `__eq__` 被实现为
Python 方法）且它们的原子性是靠不住的。你也不能指望原子变量赋值（因为这个反过来依赖字典）。

优先使用 `Queue` 模块的 `Queue` 数据类型作为线程间的数据通信方式。另外，使用 `threading`
模块及其锁原语（`locking primitives`）。了解条件变量的合适使用方式，这样你就可以使用 `threading.Condition` 来取代低级别的锁了。

## 1.19 威力过大的特性

避免使用这些特性。

### 1.19.1 定义

Python 是一种异常灵活的语言，它为你提供了很多花哨的特性，诸如元类（`metaclasses`）、字节码访问、
任意编译（`on-the-fly compilation`）、动态继承、对象父类重定义（`object reparenting`）、导入修改（`import hacks`）、
反射（例如 `getattr()` 的一些使用）、系统内修改（`modification of system internals`）、方法实现自定义清理（`__del__`）等等。

### 1.19.2 优点

强大的语言特性，能让你的代码更紧凑。

### 1.19.3 缺点

使用这些很“酷”的特性十分诱人，但不是绝对必要。使用奇技淫巧的代码将更加难以阅读和调试。开始可能还好（对原作者而言）, 但当你回顾代码,
它们可能会比那些稍长一点但是很直接的代码更加难以理解.

### 1.19.4 结论

在你的代码中避免这些特性。

内部需要使用这些特性的标准库模块和类可以使用（例如， `abc.ABCMeta` 、 `dataclasses` 和 `enum`）。

## 1.20 新版 Python:`from __future__ imports`

可以使用导入 future 这种特殊操在老版本中使用新版本的语法特性。

### 1.20.1 定义

使用 `from __future__ import` 语句可以在老版本中启用新版本的功能。

### 1.20.2 优点

经验证明，在声明兼容性并防止这些文件中的回归的同时，对每个文件进行更改，可以使运行时版本升级更加平滑。
现代代码更易于维护，因为它不太可能在将来的运行时升级期间积累技术债务。

### 1.20.3 缺点

- 没有引入所需的 future 语句时，这些代码可能无法在老版本的解释器版本上运行。
- 在支持各种环境的项目中，这种需求更为常见。

### 1.20.4 结论

**`from __future__ imports`**

推荐使用 `from __future__ import` 语句。所有的新代码都应该包含以下内容，现有的代码也应该在有条件的情况下进行兼容更新。

在 3.5 或更早的版本（而不是 >= 3.7）上执行的代码中，导入：

```python
from __future__ import generator_stop
```

有关更多信息，请阅读 [Python future](https://docs.python.org/3/library/__future__.html) 语句定义文档。

不要删除这些导入，除非您确信代码在当前环境运行没有问题。即使您现在没有使用当前代码中特定的 future 导入启用的特性，
保留这些导入便于以后修改代码时直接使用。

还有一些其他的 `from __future__` 语句，可以在需要的时候使用。

## 1.21 代码类型标注

可以根据 [PEP-484](https://www.python.org/dev/peps/pep-0484/)
使用类型标注，并使用类似 [pytype](https://github.com/google/pytype) 的类型检查工具在构建时对代码进行检查。

类型标注可以在源码中，也可以在 [stub pyi](https://www.python.org/dev/peps/pep-0484/#stub-files)文件中。
尽可能在源代码中进行标注，对于第三方库或扩展模块可以使用 `pyi` 文件。

### 1.21.1 定义

类型标注（或类型提示）可以用于函数或方法的参数和返回值

```python
def func(a: int) -> List[int]:
```

还可以使用类似 [PEP-526](https://www.python.org/dev/peps/pep-0526/) 的语法声明变量的类型：

```python
a: SomeType = some_func()
```

### 1.21.2 优点

类型标注可以提高代码的可读性和可维护性。类型检查器可以把许多运行时错误转换为构建时错误，并减少威力过大特性地使用。

### 1.21.3 缺点

- 必须保持类型标注更新。
- 您可能会看到您认为是正确代码的错误信息。
- 使用类型检查器可能会减少威力过大特性地使用。

### 1.21.4 结论

强烈建议您在更改代码时启用 Python 类型分析。当添加或修改公共 API 时，请包含类型标注，并在构建系统中启用 `pytype`进行检查。
由于静态分析对 Python 来说相对较新，我们承认会有一些副作用（比如错误的类型推断）可能会阻止一些项目采用。
因此，我们鼓励作者添加一个带有 `TODO`的注释，或者在 `BUILD` 文件或代码本身中通过 bug 链接描述当前不采用类型标注的问题。
