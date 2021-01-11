<!-- markdownlint-disable MD046 MD038 -->

# Python 语言规范

> 本文档为 [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html) 中 第二章 [Python Language Rules](https://google.github.io/styleguide/pyguide.html#2-python-language-rules) 的译文。
>
> 最后更新时间： 2020-12-09 16:37:00
>
> 如有翻译错误或不准确等任何问题，欢迎提交 PR 。谢谢你。

## 1.1 Lint

使用 [pylintrc](https://google.github.io/styleguide/pylintrc) 配置，运行 `pylint` 来检测代码。

### 1.1.1 定义

Pylint 是一个在 Python 源代码中查找 bug 和风格问题的工具。它可以发现一些通常由编译器捕获的问题，这些问题通常是针对 C 和 C++ 等动态程度较低的语言的。由于Python的动态特性，一些警告可能是不正确的；然而，虚假的警告应该很少出现。

### 1.1.2 优点

可以发现经常忽视的错误。例如拼写错误，使用在声明前使用变量，等等。

### 1.1.3 缺点

`pylint` 并不是很完美。要使用它，有时需要：

1. 围绕它写
2. 禁用警告
3. 改进它

2.1.4 决定

确保在代码上运行 `pylint`

如果警告不准确，需要禁用，以免出现其他问题。若要禁止警告，可以设置行级注释：

```python
dict = 'something awful'  # Bad idea... pylint: disable=redefined-builtin
```

`pylint` 警告都是符号名称 (empty-docstring) 标识的， Google 规范的警告是以 `g-` 开头的。

如果从符号名称中看不出禁用的原因，请添加说明。这样做的好处是可以很容易的搜索并重新查看。

要获取警告列表可以执行：

```bash
pylint --list-msgs
```

要获取特定信息的详情，可以执行：

```bash
pylint --help-msg=C6409
```

推荐使用 `pylint: disable` 而不是过时的 `pylint: disable-msg` 。

可以通过删除函数开头的变量来禁用 “未使用参数” 警告。并使用注释解释为什么删除。注释 "Unused" 就行了。例如：

```python
def viking_cafe_order(spam, beans, eggs=None):
    del beans, eggs  # Unused by vikings.
    return spam + spam + spam
```

禁用这种警告的其他常见形态有：

- 使用 `_` 作为未使用参数的标识符
- 使用 `unused_` 作为参数名的前缀
- 将它们分配给 `_`

上述的这些形式是允许的，但不再推荐。中断调用器按名称传递参数并且不强制，参数实际上并没有使用。

## 1.2 Import

只对包和模块使用导入，不对单个类或函数使用导入。注意： `typing` 模块是特例。

### 1.2.1 定义

用于将代码从一个模块共享到另一个模块的可重用机制。

### 1.2.2 优点

- 名称空间管理约定很简单。
- 每个标识符的源以一致的方式表示： `x.Obj` 表示对象 `Obj` 是在模块 `x` 中定义的。

### 1.2.3 缺点

- 模块名称仍然可能发生冲突。
- 有些模块名称太长。

### 1.2.4 决定

- 使用 `import x` 用于导入包和模块。
- 使用 `from x import y` 从包的前缀 `x` 中导入没有前缀的 `y` 模块。
- 使用 `from x import y as z` 导入重名模块，或者这个模块名字很长。
- 只有 `z` 是一个标准缩写时使用 `import y as z` 导入。例如 `np` 表示 `numpy` 。

例如，模块`sound.effects.echo` 可以这么导入：

```python
from sound.effects import echo
...
echo.EchoFilter(input, output, delay=0.7, atten=4)
```

不要在导入中使用相对名称。即使模块在同一个包中，也要使用完整的包名。这有助于防止无意中导入两次。

从 `typing` 模块和 `six.moves` 模块导入不受侧规则约束。

## 1.3 包

使用模块的完整路径名位置导入每个模块。

### 1.3.1 优点

避免模块名称冲突或由于模块搜索路径不符合作者的期望而导致不正确的导入。还可以更快速的找到模块。

### 1.3.2 缺点

因为必须复制包层次结构，所以部署代码更困。但当前的部署机制并没有什么问题。

### 1.3.3 决定

所有新代码都应该导入每个模块的完整包名。

例如：

???+ success "推荐"

    ```python
    # Reference absl.flags in code with the complete name (verbose).
    import absl.flags
    from doctor.who import jodie

    FLAGS = absl.flags.FLAGS
    ```

???+ warning "不推荐"

    ```python
    # Reference flags in code with just the module name (common).
    from absl import flags
    from doctor.who import jodie

    FLAGS = flags.FLAGS
    ```

???+ warning "不推荐 ( 假设 `jodie.py` 文件在 `doctor/who/where` 中 )"

    ```python
    # 不清楚坐着需要导入哪个模块，什么将被导入。
    # 实际上导入行为取决于 sys.patch 控制的外部因素。
    # 作者打算的导入哪个模块的 jodie 呢？
    import jodie
    ```

尽管某些环境中会发生这种情况，但不应假定二进制文件在 `sys.path` 中。这种情况下，代码应该假设 `import jodie` 指的是名为 `jodie` 的第三方包或者顶级包，而不是本地的 `jodie.py` 文件。

## 1.4 异常

异常是允许的，但必须谨慎使用。

### 1.4.1 定义

异常是一种打破常规的代码块控制流以处理错误或其他特殊情况的方法。

### 1.4.1 优点

正常操作代码的控制流不会受错误处理代码的干扰。它还允许控制流在出现某种情况时跳过多个逻辑。例如，在一个步骤中从 N 个嵌套函数返回，而不必执行错误代码。

### 1.4.2 缺点

可能导致控制流混乱。在进行库调用时容易遗漏错误案例。

### 1.4.4 决定

异常必须遵循某些条件：

- 合理使用内置异常。例如，抛出一个 `ValureError` 指示编程错误。比如违反了前置条件（需要一个正数，但传递了一个负数）。不要使用 `assert` 语句验证公共 API 的参数值。
  `assert` 用于确保内部正确性，不得强制使用，也不表示发生了某些意外事件。如果在后一种情况下需要使用异常，请使用 raise 语句。例如：

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

- 库和包可以定义它们的自己的异常。但这些异常必须从现有的异常类继承。异常名应该以 `Error` 结尾，而且避免读起来结巴（`foo.FooError`）。
- 禁止捕获所有异常 `expect:` 语句，或者捕获 `Exception` 、 `StandardError` 。Pyhon 在这方面非常宽松， `expect:` 可以捕获所有拼写错误的名称、 `sys.exit()` 调用、 `Ctrl+C` 中断、单元测试失败以及其他各种您根本不想捕获的异常。
    除非：
    - 重新抛出异常，或者
    - 在程序中创建一个隔离点，在这个隔离点中，异常不会被传播而是被记录或者消除。例如通过保护外部模块来保护线程不崩溃。
- 减少 `try/except` 块中的代码量。 `try` 的作用域越大，这部分代码快中引发代码异常的可能性就越大，而 `try/except` 就可能隐藏一个错误。
- 无论 `try` 块中是否引发异常，都可以使用 `finally` 子句执行代码。这对于清理（例如关闭文件）通常很有用。

## 1.5 全局变量

避免使用全局变量

### 1.5.1 定义

在模块级别或作为类属性声明的变量

### 1.5.2 优点

偶尔有用

### 1.5.3 缺点

有可能在导入期间更改模块行为，因为对全局变量的赋值是在首次导入时完成的。

### 1.5.4 决定

避免全局变量

虽然模块级常量在技术上是变量，但是允许或鼓励使用。例如： `MAX_HOLY_HANDGRENADE_COUNT = 3` 。常量必须使用带下划线的全部大写命名。请参阅命名规范。

如果是模块内部的变量，应在模块级别声明全局变量，并通过在模块名称前加上 `_` 。外部访问必须通过公共模块级函数完成。请参阅命名规范。

## 1.6 嵌套/局部/内部类和函数

使用嵌套局部类或函数来关闭局部变量是可行的。

### 1.6.1 定义

可以在方法、函数、类的内部定义类。函数可以在方法或函数内部定义。嵌套函数对在封闭范围中定义的变量具有只读访问权限。

### 1.6.2 优点

允许定义仅在非常有限的范围内使用的实用程序类和函数。非常 [ADT](https://en.wikipedia.org/wiki/Abstract_data_type)-y 。常用于装饰器。

### 1.6.3 缺点

- 无法对嵌套类或内部类进行 pickle 。
- 不能直接测试嵌套的函数和类。
- 嵌套会使你的外部功能变得更长、更难读。

### 1.6.4 决定

可以使用，但有一些限制。避免使用嵌套函数或类，除非要关闭局部值。不要仅仅为了对用户隐藏模块的某个函数而进行嵌套。相反应该在模块级别的名称上加 `_` 前缀，这样方便测试。

## 1.7 推导式和生成表达式

简单的情况下可以使用。

### 1.7.1 定义

`List` ， `Dict` ， 和 `Set` 推导式以及生成器表达式为创建容器类型和迭代器提供了一种简洁而有效的方法。而不需要使用传统的循环 `map()` 、 `filter()` 或 `lambda` 。

### 1.7.2 优点

简单的逻辑比其他字典、列表和集合创建技术更加清晰和简单。因为避免创建列表，所以生成器表达式可以非常高效。

### 1.7.3 缺点

理解困难，生成器表达式很难阅读。

### 1.7.4 决定

简单的情况下可以使用。每个部分必须是一行： mapping 操作、 `for` 子句 、 filter 操作。不允许使用多个子句或筛选表达式。当逻辑更复杂时，使用循环代替。

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

对支持的类型使用默认迭代器和运算符，如列表、字典和文件。

### 1.8.1 定义

容器类型，如字典和列表，定义默认迭代器和成员操作符（ `in` 和 `not in` ）

### 1.8.2 优点

默认的迭代器和运算符简单而高效。它们直接使用，不需要调用额外的方法。使用默认运算符的函数是泛型的。它们可以与支持任何该操作的类型一起使用。

### 1.8.3 缺点

不能通过读取方法名称来判断对象的类型（例如： `has_key()` 表示一个字典）。这是一个特例。

### 1.8.4 决定

对支持它们的类型使用默认迭代器和运算符，如列表、字典和文件。内置类型也定义了迭代器方法。与返回列表的方法相比，更推荐使用这些方法，但是在对容器进行迭代时不应该对其进行修改。除非特殊情况，否则不要使用 `Python 2` 特定的迭代方法 `dict.iter*()` 。

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

生成器函数返回一个迭代器，该迭代器在每次执行 `yield` 语句时生成一个值。生成值之后，生成器函数的运行时状态将会暂停，直到需要下一个值为止。

### 1.9.2 优点

更简单的代码，因为每次调用都会保留局部变量和控制流的状态。生成器使用的内存比一次创建整个值列表的函数少。

### 1.9.3 缺点

没有。

### 1.9.4 决定

在生成器函数的文档字符串中使用 `Yields` 而不是 `Returns` 。

## 1.10 Lambda 函数

一行程序的情况时使用。

### 1.10.1 定义

Lambdas 在表达式中定义匿名函数，而不是在语句中定义。它们通常用于为 `map()` 和 `filter()` 等高阶函数定义回调函数或运算符。

### 1.10.2 优点

方便。

### 1.10.3 缺点

比本地函数更难阅读和调试。缺少名称意味着堆栈跟踪更难理解。由于函数可能只包含表达式，所以表达性受到限制。

### 1.10.4 决定

可以让程序变成一行。如果 Lambda 函数中代码长度超过 60-80 个字符，那么最好将其定义为常规嵌套函数。

对于像乘法这种常见的操作，使用来自操作符模块的函数而不是使用 Lambda 函数。例如：推荐使用 `operator.mul` 而不是 `lambda x, y: x * y` 。

## 1.11 条件表达式

对于简单情况可以使用

### 1.11.1 定义

条件表达式（有时称为“三元运算符”）是为 `if` 语句提供更短语法的机制。例如 `x = 1 if cond else 2` 。

### 1.11.2 优点

比 `if` 语句更短更方便。

### 1.11.2 缺点

可能比 `if` 语句更难懂。如果表达式很长，可能很难定位。

### 1.11.4 决定

可以用于简单的情况。每个部分必须放在一行上: `true-expression, if-expression, else-expression.` 。复杂情况下应使用完整的语句。

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

在函数的参数列表的末尾为变量指定值，例如： `def(a, b=0):` 。如果 `foo` 只有一个参数，那么 `b` 被设置为 0 ，如果调用时有两个参数，则 `b` 的值为第二个参数。

### 1.12.2 优点

通常会有一个函数使用大量默认值，但在极少数情况下，希望覆盖默认值。默认参数值提供了一种简单的方法来实现这一点，而不是必须为特例定义大量的函数。由于 Python 不支持重载的方法/函数，默认参数是“伪造”重载行为的一种简单方法。

### 1.12.3 缺点

默认参数在模块加载时一次计算。如果参数是可变对象（比如列表或字典），则可能出现问题。如果函数修改对象（例如：在列表中追加一项），会修改默认值。

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

在通常使用简单，轻量级的访问器或设置器方法的地方，使用属性访问或设置数据。

### 1.13.1 定义

是一种包装方法，一个轻量的方式是将获取和设置一个属性作为标准属性访问。

### 1.13.2 优点

不用显式调用 `get` 和 `set` 方法操作数据，可读性变得提高。允许懒加载。认为是 Pythonic 维护类接口的方法。在性能方面，当允许直接访问变量时，不用调用访问方法。这也允许将来在不破坏接口的情况加添加访问器方法。

### 1.13.3 缺点

- 在 Python 2 中，必须从 `object` 继承
- 运算符可能被重载。
- 子类变得难以理解

### 1.13.4 决定

使用新代码中的属性来访问或设置数据，而通常情况下，这些属性本可以使用简单，轻量级的访问器或设置器方法。使用 `@property` 创建属性。

如果不重写属性本身，则对属性的继承可能不太好做。因此必须确保间接调用访问器方法，以确保属性（使用[模板方法设计模式](https://en.wikipedia.org/wiki/Template_method_pattern)）调用子类中重写的方法。

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

尽可能使用“隐式” `False` 。

### 1.14.1 定义

Python 在布尔上下文中将某些值计算为 `False` 。一个快速的“规律”是，所有“空”值都被认为是 `False` 。因此在布尔上下文中 `0` ， `None` ， `[]` ， `{}` ， `''` 都被计算为 `False` 。

### 1.14.2 优点

使用 Python 布尔运算的条件更容易阅读，更不容易出错。

### 1.14.3 缺点

C/C++ 开发人员看起来很怪。

### 1.14.4 决定

如果可能的话使用“隐式” `False` ，例如： `if foo:` 而不是 `if foo != []:` 。不过，有几点需要注意：

- 必须使用 `if foo is None:` 或 `if foo is not None:` 来检查 `None` 值。例如：在测试默认为 `None` 的变量或参数是否被设置为其他值时。另一个值可能在布尔上下文中为 `False` 值！
- 千万不要使用 `==` 来比较布尔变量和 `Flase` 。使用 `if not x:` 替代。如果需要区分 `False` 和 `None` ，那么将表达式连接起来，比如： `if not x and x is not None:` 。
- 对于序列（字符串、列表、元组）， 空序列就是 `False` 。因此： `if seq:` 和 `if not seq:` 比 `if len(seq):` 和 `if not len(seq)` 要更好。
- 当处理整数时，隐式 `False` 可能会出现更多的问题（例如：意外的将 `None` 处理为 0 ）。可以将已知为整数的值（不是 `len()` 的结果）与整数 0 进行比较。

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


## 1.15 不推荐的语言特性

- 尽可能使用字符串方法代替 `string` 模块。
- 使用函数调用语法替代 `apply` 。
- 使用列表推导式和 `for` 替代 `filter` 和 `map` ，因为参数本来就是一个内联的 lambda 。
- 使用 `for` 循环而不是 `reduce`

### 1.15.1 定义

Python 的当前版本提供了人们通常认为更好的替代构造。

### 1.15.2 优点

不使用任何不支持这些特性的 Python 版本，因此没理由不适用新的风格。

???+ success "推荐"

    ```python
    words = foo.split(':')

     [x[1] for x in my_list if x[2] == 5]

     map(math.sqrt, data)    # Ok. No inlined lambda expression.

     fn(*args, **kwargs)
    ```

???+ warning "不推荐"

    ```python
    words = string.split(foo, ':')

     map(lambda x: x[1], filter(lambda x: x[2] == 5, my_list))

     apply(fn, args, kwargs)
    ```

## 1.16 作用域

嵌套的 Python 函数可以引用封闭函数中定义的变量，但不能为它们赋值。变量绑定使用作用域（即基于静态程序文本）进行解析。对块中某个名称的任何赋值都会导致 Python 对该名称的所有引用视为局部变量，即使在赋值前使用。如果发生全局声明，则将该名称视为全局变量。

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

###　2.16.3 缺点

可能导致混乱和 BUG 。比如：基于 PEP-0227 的例子：

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

## 1.17 函数和函数装饰去

当有明显优点时，优先使用装饰器。避免使用 `staticmethod` ，有节制地使用 `classmethod`

### 1.17.1 定义

函数和方法的装饰器（又名为 `@` 标注法）。一个常见的装饰器是 `@property` ，用于将普通方法转换为动态计算的属性。但是装饰器也自定义。具体来说，对于某个函数 `my_decorator` ，可以这么用：

```python
class C:
    @my_decorator
    def method(self):
        # method body ...
```

这相当于：

```python
class C:
    def method(self):
        # method body ...
    method = my_decorator(method)
```

### 1.17.2 优点

优雅的指定了方法上的某些转换，转换可以消除一些重复的代码，加强不变量，等等。

### 1.17.3 缺点

装饰器可以对函数的参数或返回值执行任意操作，从而导致惊人的隐式行为。另外，装饰器在导入时执行。装饰器中的异常几乎不可能修复。

### 1.17.4 决定

- 当有明显优势时，优先使用装饰器。
- 装饰器应该遵循于函数相同导入和命名规则。
- 装饰器文档应清楚的表明该函数是一个装饰器。
- 为装饰器编写单元测试。

避免在装饰器本身中使用外部依赖（例如：不要依赖文件、套接字、数据库连接等），因为装饰器运行时（在导入时出问题， `pydoc` 问题，或其他工具出现问题）可能不可用。应（尽可能）保证在所有情况下使用有效参数调用装饰器都是正常得。

装饰器符合“顶级代码”的一种特殊情况——请参阅 main 以了解更多讨论。

永远不要使用 `staticmethod` ，除非为了现有库中定义的 API 集成而被迫使用。改为编写模块级函数。

仅在编写命名构造函数或修改必要的全局状态（如进程范围缓存）的特定类操作时，才使用 `classmethod` 。

## 1.18 线程

不要依赖内置类型的原子性。

尽管 Python 的内置类型（如字典）看起来具有原子性，但也有一些不是原子操作（例如： 如果 `__hash__` 或 `__eq__` 是作为 Python 方法实现的），而且不应认为是原子操作。也不应该依赖原子变量复制（因为这取决于字典）

使用 `Queue` 模块的 `Queue` 数据类型作为线程之间通信数据的首先方式。要不然就是使用线程模块以及锁原语。推荐使用条件变量和 `threading.Condition` ，而不是使用底层的锁。

## 1.19 强大的特性

避免使用。

### 1.19.1 定义

Python 是一种非常灵活的语言，它提供了许多奇特的特性，例如：定制元类、字节码访问、动态编译、动态继承、对象父类重定义、 hack 导入、反射（例如： `getatter()` 的一些用法）、系统内部修改等。

### 1.19.2 优点

这些都是强大的语言特性，可以使代码更加紧凑。

### 1.19.3 缺点

在不是绝对必要的时候，轻易引入这些“酷”的功能，会让代码更难阅读、理解和调试。在开始（对于原始作者来说）并没有什么感觉，但是再次审查代码时，这种代码会更难理解。

### 1.19.4 决定

在代码中避免使用这些特性。

内部使用的标准模块和类是可以使用这些特性。（例如： `abc.ABCMeta` 、 `collections.namedtuple` 、 `dataclasses` 、 和 `enum` ）。

## 1.20 新版 Python ： Python3 和 `__future__`

当前版本就是 Python 3 。虽然不是每个项目都准备使用，但是所有编写的代码都应该兼容 Python 3（如可能的话，在 Python 3 下进行测试）。

### 1.20.1 定义

Python 3 是 Python 语言中的一个重大变化。虽然现在有的代码通常是 Python 2.7 编写的，但是可以通过一些简单的工作让代码更明确表达出在 Python 3 下不需要更改。

### 1.20.2 优点

当多有依赖都兼容 Python 3 的时候，就可以直接用 Python 3 编写代码了，也更容易在 Python 3 下运行。

### 1.20.3 缺点

- 样板文件不好看。
- 导入实际并不需要的特性模块看起来怪怪的。

### 1.20.4 决定

`from __future__ imports`

推荐使用 `from __future__ import ` 语句。所有新的代码都应包含如下内容，并尽可能更新现有代码以使其兼容

```python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function
```

有关这些导入的更多信息，请参阅 [absolute imports](https://www.python.org/dev/peps/pep-0328/) 、 [division behavior](https://google.github.io/styleguide/pyguide.html)
 和 [the print function](https://www.python.org/dev/peps/pep-3105/)。

还有其他 `from __future__` 语句。在你认为合适的时候使用。推荐中没有包括 `unicode_literals` ，因为在 Python 2.7 中会在许多地方引入隐式的默认编码转化。大多数代码最好在必要时显示使用 `b''` 和 `u''` 字节以及 unicode 字符串文本。

**`six` 、 `future` 和 `past` 库**

当项目需要同时支持 Python 2 和 Python 3 时，可以根据自己的需要使用[`six`](https://pypi.org/project/six/) 、 [`future`](https://pypi.org/project/future/) 、 [`past`](https://pypi.org/project/past/) 。这些库的存在是为了让代码更清晰和简单。

## 1.21 代码类型标注

在 Python 3 中，可以根据 [PEP-484](https://www.python.org/dev/peps/pep-0484/) 使用类型标注，并在构建时使用类型检查工具（如： pytype） 检查代码。

类型标注可以在原文件中，可以在 [pyi 存根](https://www.python.org/dev/peps/pep-0484/#stub-files)文件中。尽可能的将标注放在源代码中。对于第三方或扩展模块使用 pyi 文件。

### 1.21.1 定义

类型标注（或类型提示）适用于函数或方法参数和返回值。

```python
def func(a: int) -> List[int]:
```

还可以使用 [PEP-526](https://www.python.org/dev/peps/pep-0526/) 语法声明变量类型。

```python
a: SomeType = some_func()
```

或者对于老版本 Python 必须使用注释类型标注

```python
a = some_func()  # type: SomeType
```

### 1.21.2 优点

类型标注提高了代码的可读性和可维护性。类型检查会在构建时发现许多将会出现在运行时的错误，并减少新特性地使用。

### 1.21.3 缺点

- 必须不断更新类型标注。
- 本该正确的代码可能出现类型错误。
- 类型检查可能会减少使用新特性的使用。
  
### 1.31.4 决定

强烈建议在更新代码时启用类型分析。在添加或修改公共 API 时，在构建过程中使用 pytype 检查类型标注。由于静态分析刚兴起，某些项目可能会因为负面效果（例如：类型推断错误）而不使用。在这种情况下，鼓励坐着添加一个带有 TODO 的注释，或者连接到有一个错误，该错误描述了在构建文件或者代码本身使用类型标注出现的问题。

<!-- markdownlint-restore -->
