<!-- markdownlint-disable MD046 MD038 -->

# Python 语言规范

> 本文档为 [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html) 第二章 [Python Language Rules](https://google.github.io/styleguide/pyguide.html#2-python-language-rules) 的译文。
>
> 最后更新时间： 2021-04-28
>
> 如果有翻译错误或表述不准确的问题，欢迎提交 PR，感谢您的参与。

## 1.1 Lint

使用 [pylintrc](https://google.github.io/styleguide/pylintrc) 配置，运行 `pylint` 来检测代码。

### 1.1.1 定义

Pylint 是一个在 Python 源代码中查找 bug 和风格问题的工具。它会发现一些通常是由编译器在动态程度较低的语言(如 C 和 c++)中捕获的问题。由于 Python 的动态特性，一些警告可能是不正确的，不过误判的警告应该也相当少见。

### 1.1.2 优点

捕获容易遗漏的错误，如拼写错误、赋值前使用 var 等。

### 1.1.3 缺点

`pylint` 并不是很完美。要使用它，有时我们需要围绕它进行编写，禁用它的警告或修复它。

### 1.1.4 决定

确保在代码上运行 `pylint`。

如果警告不准确，需要禁用它们，以免出现其他问题。要禁用警告，可以设置行级注释：

```python
dict = 'something awful'  # Bad idea... pylint: disable=redefined-builtin
```

`pylint` 警告均由符号名称 (empty-docstring) 标识， Google 特定的警告以 g- 开头。

如果从符号名称中看不出禁用的原因，可以添加说明。

用这种方式禁用的好处是，我们可以很容易地找到相关禁用，并重新审视它们。

您可以通过执行以下操作获取 `pylint` 警告列表：

```bash
pylint --list-msgs
```

要获取特定信息的详情，可以执行：

```bash
pylint --help-msg=C6409
```

推荐使用 `pylint: disable` 而不是已弃用的 `pylint: disable-msg` 。

未使用参数的警告可以通过删除函数开头的变量来消除。并包含一个注释解释为什么删除它。使用 “Unused.” 注释就足够了。例如：

```python
def viking_cafe_order(spam, beans, eggs=None):
    del beans, eggs  # Unused by vikings.
    return spam + spam + spam
```

禁用这种警告的常见形式还包括使用 “\_" 作为未使用参数的标识符，或在参数名前加上 “unused\_”，或将它们赋值给 “\_"。

上述的这些形式都是允许的，但不再推荐。调用方法时按名称传递的这些参数，实际上并不一定会使用。

## 1.2 导入

只对包和模块使用 import 语句，而不是单独的类或函数。注意： `typing` 模块是个特例。

### 1.2.1 定义

用于从一个模块共享代码到另一个模块的可重用性机制。

### 1.2.2 优点

命名空间的管理约定很简单。每个标识符的来源以一致的方式表示：`x.Obj` 表示对象 `Obj` 是在模块 `x` 中定义的。

### 1.2.3 缺点

模块名仍然可能发生冲突。有些模块名很长，很不方便。

### 1.2.4 决定

- 使用 `import x` 来导入包和模块。
- 使用 `from x import y`，其中 `x` 是包前缀，`y` 是不带前缀的模块名。
- 如果要导入两个名为 `y` 的模块，或者 `y` 是一个不太方便的长名称，则使用 `from x import y as z`。
- 只有当 `z` 是标准缩写(例如，`numpy` 为 `np`)时，才使用 `import y as z`。

例如，可以像下面这样导入模块 `sound.effects.echo`：

```python
from sound.effects import echo
...
echo.EchoFilter(input, output, delay=0.7, atten=4)
```

不要在导入中使用相对名称。即使模块在同一个包中，也要使用完整的包名。这有助于防止无意中导入一个包两次。

从 `typing` 模块和 `six.moves` 模块导入不受此规则约束。

## 1.3 包

使用模块的完整路径名位置导入每个模块。

### 1.3.1 优点

避免模块名称冲突或由于模块搜索路径不符合作者的期望而导致不正确的导入。可以更快速的找到模块。

### 1.3.2 缺点

这使得部署代码更加困难，因为您必须复制包层次结构。但对于当前部署机制来说，这并不是问题。

### 1.3.3 决定

所有新代码都应该按模块的完整包名导入每个模块。

导入应该如下所示：

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

异常使用是允许的，但必须谨慎使用。

### 1.4.1 定义

异常是中断正常控制流以处理错误或其他异常条件的一种方法。

### 1.4.1 优点

正常操作代码的控制流不会受错误处理代码干扰。它还允许控制流在特定条件发生时跳过多个帧，例如，在一个步骤中从 N 个嵌套函数返回，而不是通过检查错误代码。

### 1.4.2 缺点

可能导致控制流混乱。在进行库调用时，容易遗漏错误情况。

### 1.4.4 决定

异常必须遵循某些条件：

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

- 库或包可以定义它们自己的异常。但这些异常必须从现有的异常类继承。异常名应该以 `Error` 结尾，而且避免读起来拗口（`foo.FooError`）。
- 永远不要使用 `expect:` 语句捕获所有异常 ，或者直接捕获 `Exception` 和   `StandardError` 异常。
    除非：
    - 重新抛出异常，或
    - 在程序中创建一个隔离点，其中异常不传播，而是被记录和抑制。例如通过保护线程的最外层来防止线程崩溃。
  
  Python 在这方面非常宽松， `expect:` 可以捕获所有拼写错误的名称， `sys.exit()` 调用， `Ctrl+C` 中断，`unittest` 失败和所有你不想捕获的其他异常。

- 减少 `try / except` 块中的代码量。 `try` 的作用域越大，就越有可能由您不希望引发异常的代码行引发异常。在这些情况下，`try / except` 块将隐藏真正的错误。
- 无论是否在 `try` 块中引发异常，都可以使用 `finally` 子句执行代码。这通常对清理工作很有用，比如关闭文件。

## 1.5 全局变量

避免使用全局变量。

### 1.5.1 定义

在模块级声明或作为类属性声明的变量。

### 1.5.2 优点

偶尔有用。

### 1.5.3 缺点

有可能在导入期间更改模块行为，因为对全局变量的赋值是在模块第一次导入时完成的。

### 1.5.4 决定

避免全局变量。

虽然模块级常量在技术上是变量，但是允许和鼓励使用。例如： `MAX_HOLY_HANDGRENADE_COUNT = 3` 。常量的命名必须使用全大写和下划线。具体请参阅命名规范。

如果需要，应该在模块级别声明全局变量，并通过在名称前加上 `_` 前缀使其成为模块的内部变量。外部访问必须通过公共模块级函数完成。具体请参阅命名规范。

## 1.6 嵌套/局部/内部类和函数

嵌套局部函数或类用于关闭局部变量是没问题的，内部类更好。

### 1.6.1 定义

类可以在方法、函数或类内部定义。函数可以在方法或函数中定义。嵌套函数对定义在封闭作用域中的变量具有只读访问权限。

### 1.6.2 优点

允许定义只在非常有限的范围内使用的实用程序类和函数。非常像 [ADT](https://en.wikipedia.org/wiki/Abstract_data_type)-y 。通常用于实现装饰器。

### 1.6.3 缺点

- 不能直接测试嵌套函数和类。
- 嵌套会使外部函数更长。
- 可读性更差。

### 1.6.4 决定

可以使用，但有一些限制。避免使用嵌套函数或类，除非要关闭局部值。不要仅仅为了对用户隐藏模块的某个函数而进行嵌套。相反，应该在模块级别的名称上加 `_` 前缀，这样方便测试。

## 1.7 推导式和生成表达式

在简单的情况下使用。

### 1.7.1 定义

`List`、`Dict` 和 `Set` 推导式以及生成器表达式提供了一种简洁而有效的方法来创建容器类型和迭代器，而无需使用传统的循环、`map()`、`filter()` 或 `lambda`。

### 1.7.2 优点

简单的推导式比其他的字典、列表或集合创建技术更清晰和简单。生成器表达式可以非常高效，因为它们完全避免了创建列表。

### 1.7.3 缺点

复杂的推导式或生成器表达式可能很难阅读。

### 1.7.4 决定

在简单的情况下使用。每个部分必须适合一行：`mapping` 表达式，`for` 子句，`filter` 表达式。不允许使用多个子句或过滤器表达式。当逻辑变得更复杂时，可以使用循环代替。

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

对于支持默认迭代器和操作符的类型，如列表、字典和文件，请使用默认迭代器和操作符。

### 1.8.1 定义

容器类型，如字典和列表，定义了默认迭代器和成员判断操作符（ `in` 和 `not in` ）

### 1.8.2 优点

默认的迭代器和运算符简单而高效。它们直接表示操作，而不需要额外的方法调用。使用默认操作符的函数是泛型的。它可以与支持该操作的任何类型一起使用。

### 1.8.3 缺点

您不能通过读取方法名称来分辨对象的类型（例如，`has_key()` 表示一个字典）。这是一个特例。

### 1.8.4 决定

对于支持默认迭代器和操作符的类型，如列表、字典和文件，请使用默认迭代器和操作符。内置类型也定义了迭代器方法。与返回列表的方法相比，更推荐使用这些方法，但是在对容器进行迭代时不应该对其进行修改。除非特殊情况，否则不要使用 Python 2 特定的迭代方法 `dict.iter*()` 。

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

根据需要使用生成器。

### 1.9.1 定义

生成器函数返回一个迭代器，每次执行 `yield` 语句都会生成一个值。在生成一个值之后，生成器函数的运行时状态将暂停，直到需要下一个值。

### 1.9.2 优点

更简单的代码，因为每次调用都会保留局部变量和控制流的状态。生成器比一次创建整个值列表的函数使用更少的内存。

### 1.9.3 缺点

无。

### 1.9.4 决定

在生成器函数的文档字符串中使用 `Yields` 而不是 `Returns` 。

## 1.10 Lambda 函数

一种简单的、在同一行中定义函数的方法。

### 1.10.1 定义

`Lambdas` 在表达式中定义匿名函数，而不是在语句中定义。它们通常用于为 `map()` 和 `filter()` 等高阶函数定义回调函数或运算符。

### 1.10.2 优点

简便。

### 1.10.3 缺点

比本地函数更难阅读和调试。缺少名称意味着堆栈跟踪更加难以理解。表达性受到限制，因为函数可能只包含一个表达式。

### 1.10.4 决定

在需要单行的场景下可以使用。如果 `lambda` 函数内部的代码长度超过60-80个字符，最好将其定义为常规嵌套函数。

对于像乘法这样的常见操作，使用 `operator` 模块中的函数而不是 `lambda` 函数。例如，推荐使用 `operator.mul` 而不是 `lambda x, y: x * y` 。

## 1.11 条件表达式

简单情况可以使用。

### 1.11.1 定义

条件表达式（有时称为三元运算符）是一种为 `if` 语句提供更短语法的机制。例如 `x = 1 if cond else 2` 。

### 1.11.2 优点

比 `if` 语句更短、更方便。

### 1.11.2 缺点

可能比 `if` 语句更难理解。如果表达式很长，条件可能很难定位。

### 1.11.4 决定

可以用于简单的情况。每个部分必须放在一行上： `true-expression, if-expression, else-expression` 。复杂情况下应使用完整的 `if` 语句。

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

适用大多数情况。

### 1.12.1 定义

在函数的参数列表的末尾为变量指定值，例如： `def(a, b=0):` 。如果 `foo` 只有一个参数，那么 `b` 被设置为 `0` ，如果调用时有两个参数，则 `b` 的值为第二个参数。

### 1.12.2 优点

通常会有一个使用大量默认值的函数，但在极少数情况下希望覆盖默认值。默认参数值提供了一种简单的方法来实现这一点，而不是必须为特殊情况定义大量的函数。由于 Python 不支持重载的方法或函数，默认参数是“模拟”重载行为的一种简单方法。

### 1.12.3 缺点

默认参数在模块加载时计算一次。如果参数是一个可变对象，如列表或字典，这可能会导致问题。如果函数修改了对象(例如，向列表中添加一个项)，默认值就会被修改。

### 决定

不要在函数或方法定义中使用可变对象作为默认值。

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

## 1.13 属性

在使用简单的、轻量级的访问或设置方法的情况下，可以使用属性来访问或设置数据。

### 1.13.1 定义

要求在轻量级计算时，将获取和设置属性的方法调用包装为标准属性访问的方法。

### 1.13.2 优点

通过消除简单属性访问的显式 `get` 和 `set` 方法调用，提高了可读性。允许延迟加载。考虑用 python 方式维护类的接口。在性能方面，当允许直接访问变量时，不用调用访问方法。这也允许将来在不破坏接口的情况下添加访问器方法。

### 1.13.3 缺点

- 会隐藏类似操作符重载的副作用。
- 对于子类可能会造成混淆。

### 1.13.4 决定

使用新代码中的属性来访问或设置数据，而通常情况下，这些属性是使用轻量级访问器或设置器方法的。属性应该使用 `@property` 装饰器创建。

如果属性本身没有被覆盖，那么属性的继承可能不明显。因此，必须确保间接调用访问器方法，以确保子类中重写的方法被属性调用（可以使用[模板方法设计模式](https://en.wikipedia.org/wiki/Template_method_pattern)）。

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

## 1.14 `True` / `False` 判断

尽可能使用隐式的 `False` 。

### 1.14.1 定义

Python 在布尔上下文中将某些值计算为 `False` 。一个简单的经验法则是，所有空值都被认为是 `False`，所以 `0`, `None`，`[]`，`{}`，`''` 在布尔上下文中都被认为是 `False`。

### 1.14.2 优点

使用 Python 布尔值的条件更容易阅读，而且不容易出错。在大多数情况下，他们也更快。

### 1.14.3 缺点

对于 C / C ++ 开发人员来说可能看起来很奇怪。

### 1.14.4 决定

如果可能，使用隐式的 `false`，例如，`if foo:` 而不是 `if foo !=[]:`。不过，有一些需要注意的事项：

- 总是使用 `if foo is None:` 或 `if foo is not None:` 来检查 `None` 值。例如，当测试一个默认为 `None` 的变量或参数是否被设置为其他值时。另一个值可能是一个在布尔上下文中为 `false` 的值!
- 永远不要使用 `==` 将布尔变量与 `False` 进行比较。使用 `if not x:` 替代。如果需要区分 `False` 和 `None` ，那么将表达式连接起来，比如： `if not x and x is not None:` 。
- 对于序列（字符串、列表、元组）， 空序列就是 `False` 。因此： `if seq:` 和 `if not seq:` 比 `if len(seq):` 和 `if not len(seq)` 要更好。
- 当处理整数时，隐式 `False` 可能会出现更多的问题（例如，意外的将 `None` 处理为 `0` ）。可以将已知为整数的值（不是 `len()` 的结果）与整数 `0` 进行比较。

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

## 1.16 作用域

### 1.16.1 定义

嵌套的 Python 函数可以引用封闭函数中定义的变量，但不能为它们赋值。变量绑定使用作用域（即基于静态程序文本）进行解析。在块中对名称的任何赋值都会导致 Python 将对该名称的所有引用视为局部变量，即使在赋值前使用。如果出现全局声明，则该名称被视为全局变量。

比如这个例子：

```python
def get_adder(summand1):
    """Returns a function that adds numbers to a given number."""
    def adder(summand2):
        return summand1 + summand2

    return adder
```

### 1.16.2 优点

通常会产生更清晰、更优雅的代码。特别适合有经验的 Lisp 和 Scheme （以及 Haskell 和 ML 等）程序员。

### 2.16.3 缺点

可能导致令人困惑的错误。比如，基于 [PEP-0227](http://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0227/) 的例子：

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

`foo([1, 2, 3])` 将打印 `1 2 3 3` 而不是 `1 2 3 4`

### 1.16.4 决定

可以使用。

## 1.17 函数和方法装饰器

当有明显优势时，优先使用装饰器。避免使用 `staticmethod` ，限制使用 `classmethod`。

### 1.17.1 定义

[函数和方法的装饰器](https://docs.python.org/3/glossary.html#term-decorator)（也称为“ `@` 表示法”）。一个常见的装饰器是 `@property` ，用于将普通方法转换为动态计算的属性。但是，装饰器语法也允许用户定义的装饰器。具体来说，对于一些函数 `my_decorator` ，可以这么用：

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

优雅地指定方法上的一些转换；转换可能会消除一些重复的代码，强制不变量等。

### 1.17.3 缺点

装饰器可以对函数的参数或返回值执行任意操作，从而导致令人惊讶的隐式行为。此外，装饰器在导入时执行。装饰器代码中的异常几乎不可能恢复。

### 1.17.4 决定

- 当有明显优势时，优先使用装饰器。
- 装饰器应该遵循与函数相同的导入和命名准则。
- 装饰器 `pydoc` 应该清楚地声明该函数是一个装饰器。
- 为装饰器编写单元测试。

避免装饰器本身的外部依赖（例如，不要依赖于文件、套接字、数据库连接等），因为它们可能在装饰器运行时不可用（在导入时出问题， `pydoc` 问题，或其他工具出现问题）。应该（尽可能）保证使用有效参数调用的装饰器在所有情况下都能成功。

装饰器是顶层代码的一种特殊情况——更多讨论请参见 [main](https://google.github.io/styleguide/pyguide.html#s3.17-main)。

永远不要使用 `staticmethod` ，除非为了与现有库中定义的 API 集成而被迫使用。可以写一个模块级函数代替。

只有在编写命名构造函数或修改必要的全局状态（如进程级缓存）的特定类操作时才使用 `classmethod`。

## 1.18 线程

不要依赖内置类型的原子性。

虽然 Python 的内置数据类型（如字典）似乎具有原子性操作，但也有一些不是原子操作（例如，如果 `__hash__` 或 `__eq__` 是作为 Python 方法实现的），它们的原子性不应该被依赖。您也不应该依赖原子变量赋值（因为这反过来又依赖于字典）。

使用 `Queue` 模块的 `Queue` 数据类型作为线程间数据通信的首选方式。要不然就是使用线程模块以及锁原语。推荐使用条件变量和 `threading.Condition` ，而不是使用底层的锁。

## 1.19 超级特性

避免使用。

### 1.19.1 定义

Python 是一种非常灵活的语言，提供了许多奇特的功能，如自定义元类、访问字节码、动态编译、动态继承、对象重定向、 导入修改、反射（例如 `getattr()` 的一些使用）、修改系统内部等等。

### 1.19.2 优点

这些都是功能强大的语言特性。它们可以使您的代码更紧凑。

### 1.19.3 缺点

在不是绝对必要的时候，轻易引入这些很酷的功能，会让代码更难阅读、理解和调试。一开始（对原始作者来说）似乎不是那样的，但当重新访问代码时，它往往比简单的长代码更难理解。

### 1.19.4 决定

在代码中避免使用这些特性。

内部使用这些特性的标准库模块和类是可以使用的（例如， `abc.ABCMeta` 、 `collections.namedtuple` 、 `dataclasses` 、 和 `enum` ）。

## 1.20 新版 Python ： Python 3 和 `from __future__ imports`

推荐版本就是 Python 3 。虽然不是每个项目都准备使用，但所有编写的代码都应该兼容 Python 3 （并在可能的情况下在 Python 3 下进行测试）。

### 1.20.1 定义

Python 3 是 Python 语言中的一个重大变化。虽然现有的代码通常是按照 Python 2.7 编写的，但是可以通过一些简单的工作让代码更明确地说明其意图，从而更好地准备在 Python 3 下使用而无需修改。

### 1.20.2 优点

当所有依赖都兼容 Python 3 的时候，就可以直接用 Python 3 编写代码了，也更容易在 Python 3 下运行。

### 1.20.3 缺点

- 样板文件不好看。
- 导入实际并不需要的特性模块看起来怪怪的。

### 1.20.4 决定

**`from __future__ imports`**

推荐使用 `from __future__ import ` 语句。所有的新代码应该包含以下内容，现有的代码应该在可能的情况下更新以兼容：

```python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function
```

获取有关这些导入的更多信息，请参阅 [absolute imports](https://www.python.org/dev/peps/pep-0328/) 、 [division behavior](https://google.github.io/styleguide/pyguide.html)
 和 [the print function](https://www.python.org/dev/peps/pep-3105/)。

请不要忽略或删除这些导入，即使它们当前没有在模块中使用，除非代码只在 Python 3 中使用。最好总是在所有文件中有 `future` 的导入，这样当有人开始使用这样的特性时，就不会在以后的编辑中忘记它们。

还有些其他的 `from __future__` 语句。可以在你认为合适的时候使用。推荐中没有包括 `unicode_literals` ，因为在 Python 2.7 中会在许多地方引入隐式的默认编码转化。大多数代码最好在必要时显示使用 `b''` 和 `u''` 字节以及 unicode 字符串文本。

**`six` 、 `future` 和 `past` 库**

当项目需要同时支持 Python 2 和 Python 3 时，可以根据自己的需要使用 [`six`](https://pypi.org/project/six/) 、 [`future`](https://pypi.org/project/future/) 、 [`past`](https://pypi.org/project/past/) 。这些库的存在是为了让代码更清晰和简单。

## 1.21 代码类型标注

您可以根据 [PEP-484](https://www.python.org/dev/peps/pep-0484/) 使用类型标注 Python 3 代码，并在构建时使用类似 [pytype](https://github.com/google/pytype) 的类型检查工具对代码进行类型检查。

类型标注可以在原文件中，也可以在 [stub pyi](https://www.python.org/dev/peps/pep-0484/#stub-files) 文件中。尽可能的将标注放在源代码中。对于第三方或扩展模块使用 `pyi` 文件。

### 1.21.1 定义

类型标注（或类型提示）用于函数或方法参数和返回值

```python
def func(a: int) -> List[int]:
```

您还可以使用类似的 [PEP-526](https://www.python.org/dev/peps/pep-0526/) 语法声明变量的类型：

```python
a: SomeType = some_func()
```

或者在必须支持遗留 Python 版本的代码中使用类型注释。

```python
a = some_func()  # type: SomeType
```

### 1.21.2 优点

类型标注可以提高代码的可读性和可维护性。类型检查器将把许多运行时错误转换为构建时错误，并减少超级特性地使用。

### 1.21.3 缺点

- 必须不断更新类型标注。
- 您可能会看到您认为是有效代码的类型错误。
- 使用类型检查器可能会减少超级特性的使用。
  
### 1.31.4 决定

强烈建议您在更新代码时启用 Python 类型分析。在添加或修改公共 `API` 时，在构建系统中包含类型标注并通过 `pytype` 启用检查。由于静态分析对 Python 来说相对较新，我们承认一些不希望的副作用（比如错误推断的类型）可能会阻止一些项目采用静态分析。在这些情况下，我们鼓励作者添加一个带有 `TODO` 的注释，或者链接到描述当前在 `BUILD` 文件或代码本身中阻止采用类型标注的问题的错误。

<!-- markdownlint-restore -->
