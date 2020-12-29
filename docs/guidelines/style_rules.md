<!-- markdownlint-disable MD046 MD038 -->
# Python 风格规范

> 本文档为 [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html) 中 第二章 [Python Style Rules](https://google.github.io/styleguide/pyguide.html#3-python-style-rules) 的译文。
>
> 最后更新时间： 2020-12-09 16:37:00
>
> 如有翻译错误或不准确等任何问题，欢迎提交 PR 。谢谢你。

## 3.1 分号

不要在行尾用分号，也不要使用分号将两个语句放在同一行上。

## 3.2 行长

最大行长为 80 字符。

特例：

- 长的导入声明
- 注释中的 URL 、路径名和长标识
- 不包含空格，不方便跨行的长字符串模块级常量（例如 URL 或路径）
    - Pylint 禁用注释。（例如： `# pylint: disable=invalid-name` ）
  
除非语句需要三个或更多的上下文管理器，否则不要使用反斜杠延续。

利用 Python 的特性： [括号、方括号、大括号内的隐式行连接](https://docs.python.org/3/reference/lexical_analysis.html#implicit-line-joining) 。如果有必要，可以在换行时，周围加一对括号。

!!! success "推荐"

    ```python
    foo_bar(self, width, height, color='black', design=None, x='foo',
            emphasis=None, highlight=0)

    if (width == 0 and height == 0 and
            color == 'red' and emphasis == 'strong'):
    ```

当文本字符串不能放在单行上时，使用括号进行隐式换行。

```python
x = ('This will build a very long long '
     'long long long long long long string')
```

在注释中，如果需要长的 URL 放在一行的话：

!!! success "推荐"
    ```python
    # See details at
    # http://www.example.com/us/developer/documentation/api/content/v2.0/csv_file_name_extension_full_specification.html
    ```

!!! fail "不推荐"
    ```python
    # See details at
    # http://www.example.com/us/developer/documentation/api/content/\
    # v2.0/csv_file_name_extension_full_specification.html
    ```

在定义表达式跨三行或更多行的 `with` 语句上，允许使用反斜杠延续。对于两行表达式，使用嵌套语句：

!!! success "推荐"

    ```python
    with very_long_first_expression_function() as spam, \
         very_long_second_expression_function() as beans, \
         third_thing() as eggs:
        place_order(eggs, beans, spam, beans)
    ```

!!! fail "不推荐"

    ```python
    with VeryLongFirstExpressionFunction() as spam, \
         VeryLongSecondExpressionFunction() as beans:
    PlaceOrder(eggs, beans, spam, beans)
    ```

!!! success "推荐"

    ```python
    with very_long_first_expression_function() as spam:
        with very_long_second_expression_function() as beans:
            place_order(beans, spam)
    ```

在上面的换行示例中，记下元素缩进。参阅 [缩进](#34) 部分了解更多信息。

## 3.3 括号

尽量少用括号。

在元组周围最好少用括号。不要在 return 语句或条件语句中使用，除非在隐式的换行或表示元组时使用括号。

!!! success "推荐"

    ```python
    if foo:
        bar()
    while x:
        x = bar()
    if x and y:
        bar()
    if not x:
        bar()
    # For a 1 item tuple the ()s are more visually obvious than the comma.
    onesie = (foo,)
    return foo
    return spam, beans
    for x, y in dict.items(): ...
    ```

!!! fail "不推荐"

    ```python

    if (x):
        bar()
    if not(x):
        bar()
    return (foo)
    return (spam, beans)
    for (x, y) in dict.items(): ...
    ```

## 3.4 缩进

使用 4 个空格缩进代码块。

- 不要使用制表符或制表符和空格混用。
- 在隐式换行时，应该垂直对齐包装的元素，如[行长](#32)部分的示例
- 当第一行的开括号后面应该没有任何内容，应使用四个空格的悬挂缩进，。

!!! success "推荐"

    ```python
    # Aligned with opening delimiter
    foo = long_function_name(var_one, var_two,
                             var_three, var_four)
    meal = (spam,
            beans)

    # Aligned with opening delimiter in a dictionary
    foo = {
        long_dictionary_key: value1 +
                            value2,
        ...
    }

    # 4-space hanging indent; nothing on first line
    foo = long_function_name(
        var_one, var_two, var_three,
        var_four)
    meal = (
        spam,
        beans)

    # 4-space hanging indent in a dictionary
    foo = {
        long_dictionary_key:
            long_dictionary_value,
        ...
    }
    ```

!!! fail "不推荐"

    ```python
    # Stuff on first line forbidden
    foo = long_function_name(var_one, var_two,
        var_three, var_four)
    meal = (spam,
        beans)

    # 2-space hanging indent forbidden
    foo = long_function_name(
      var_one, var_two, var_three,
      var_four)

    # No hanging indent in a dictionary
    foo = {
        long_dictionary_key:
        long_dictionary_value,
        ...
    }
    ```

### 3.4.1 在序列的条目最后加逗号？

只有在序列结束符 `]` 、 `)` 或 `}` 与最后一个元素不在同一行时才推荐使用。尾随逗号的存在还用作对代码自动格式化程序 [YAPF](https://pypi.org/project/yapf/) 的提示，以指示在最后一个元素之后出现时，自动将项目容器的格式设置为每行一个条目。

!!! success "推荐"

    ```python
    golomb3 = [0, 1, 3]
    ```

    ```python
    golomb4 = [
        0,
        1,
        4,
        6,
    ]
    ```

!!! fail "不推荐"

    ```python
    golomb4 = [
        0,
        1,
        4,
        6
    ]
    ```

## 3.5 空白行

- 顶级定义之间用两个空白行，可以是函数定义，可以是类定义。
- 方法定义间隔、 `class` 所在行于第一个方法之间使用一个空白行。
- 在 `def` 行之后没有空行。
- 函数或方法内部，使用一个空白行。

## 3.6 空格

遵循标点符号周围空格的排版标准。

圆括号、方括号和大括号内没有空格。

!!! success "推荐"

    ```python
    spam(ham[1], {eggs: 2}, [])
    ```

!!! fail "不推荐"

    ```python
    spam( ham[ 1 ], { eggs: 2 }, [ ] )
    ```

逗号、分号或冒号前面没有空格。在逗号、分号或冒号后面使用空格，但行尾除外（没有空格）。

!!! success "推荐"

    ```python
    if x == 4:
        print(x, y)
    x, y = y, x
    ```

!!! fail "不推荐"

    ```python
    if x == 4 :
        print(x , y)
    x , y = y , x
    ```

在参数列表、索引或切片开始的开括号前没有空格。

!!! success "推荐"

    ```python
    spam(1)
    ```

!!! fail "不推荐"

    ```python
    spam (1)
    ```

!!! success "推荐"

    ```python
    dict['key'] = list[index]
    ```

!!! fail "不推荐"

    ```python
    dict ['key'] = list [index]
    ```

行最后没有空格。

用一个单独的空格将二进制操作符包起来，用于赋值（ `=` ）、比较（ `==` 、 `<` 、 `>` 、 `!=` 、 `<>` 、 `<=` 、 `>=` 、 `in` 、 `not in` 、 `is` 、 `is not` ）和布尔操作（ `and` 、 `or` 、 `not` ） 。
在算数运算符（ `+` 、 `-` 、 `*` 、 `/` 、 `//` 、 `%` 、 `**` 、 `@` ）周围插入空格，更易于阅读。

!!! success "推荐"

    ```python
    x == 1
    ```

!!! fail "不推荐"

    ```python
    x<1
    ```

在关键字参数或定义默认参数值时，不要在 `=` 周围使用空格。但有一个例外：当存在类型注释时，在默认参数值的 `=` 周围使用空格。

!!! success "推荐"

    ```python
    def complex(real, imag=0.0): return Magic(r=real, i=imag)
    ```

    ```python
    def complex(real, imag: float = 0.0): return Magic(r=real, i=imag)
    ```

!!! fail "不推荐"

    ```python
    def complex(real, imag = 0.0): return Magic(r = real, i = imag)
    ```

    ```python
    def complex(real, imag: float=0.0): return Magic(r = real, i = imag)
    ```

请勿使用空格在连续的行上垂直对齐，因为这会增加维护负担（使用于 `:` 、 `#` 、 `=` 等）：

!!! success "推荐"

    ```python
    foo = 1000  # comment
    long_name = 2  # comment that should not be aligned

    dictionary = {
        'foo': 1,
        'long_name': 2,
    }
    ```

!!! fail "不推荐"

    ```python
    foo       = 1000  # comment
    long_name = 2     # comment that should not be aligned

    dictionary = {
        'foo'      : 1,
        'long_name': 2,
    }
    ```

## 3.7 [Shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) 行

大多数 `.py` 文件不需要以 `#!` 。程序主文件的开头是 `#!/usr/bin/python` 加上一个可选的单个数字 `2` 或 `3` 结尾的行。参见： [PEP-394](https://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0394/) 。

内核通过这一行来查找 Python 解释器，但是在导入模块时被 Python 忽略。只需要在一个可执行文件上定义。

## 3.8 注释和文档字符串

正确使用模块、函数、方法的文档字符串和内联注释。

### 3.8.1 文档字符串

Python 使用文档字符串记录代码。文档字符串是一个字符串，是包、模块、类或函数中的第一个语句。这些字符串通过对象的 `__doc__` 自动提取，并由 `pydoc` （尝试运行 `pydoc` 看一下结果） 使用。
文档字符串始终使用三个双引号 `"""` （参见： [PEP-257](https://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0257/) ）。文档字符串会合并成一行（一个物理行不超过 80 字符串），以句号、问好和感叹号结尾。当写入更多（鼓励）内容时，后面必须跟一个空行，然后是文档字符串的其余部分，从与第一行第一个引号相同的光标为止开始。后面会后更详细描述。

### 3.8.2 模块

每个文件都应该包含许可证文本。为项目选择合适的许可证文本（例如： `Apache 2.0` 、 `BSD` 、 `LGPL` 、 `GPL` ） 。

文件应该以描述模块内容和用法的文档字符串开始。

```python
"""A one line summary of the module or program, terminated by a period.

Leave one blank line.  The rest of this docstring should contain an
overall description of the module or program.  Optionally, it may also
contain a brief description of exported classes and functions and/or usage
examples.

  Typical usage example:

  foo = ClassFoo()
  bar = foo.FunctionBar()
"""
```

### 3.8.3 函数和方法

在本节中，”函数“指方法、函数或生成器。

一个函数必须有一个文档字符串，除非满足一下所有条件：

- 外部不可见
- 很短
- 见名知意

文档字符串应该提供足够的信息，以便在不阅读代码的情况下调用函数。文档字符串应该是描述性的（ `"""Fetches rows from a Bigtable."""` ） 而不是命令式的（ `"""Fetch rows from a Bigtable."""` ） ，除了 `@property` 装饰的方法，其余都应该使用相同的风格。文本字符串应该描述函数的调用语法和语义，而不是它的实现。对于复杂代码，应在旁边旁边加上注释。

覆盖来自基类的方法时，用一个简单的字符串引导读者查被覆盖方法的文档字符串，如： `"""See base class."""` 。目的是不要重复记录基本方法的文档字符串。但是如果被重写的方法行为发生了改变或者需要提供更详细信息（例如：产生其他影响），并且要在重写方法上记录。

函数的一些方面应参考下面内容做记录。每个部分以标题开始，并以冒号结束。除了标题外的所有部分都应保持两个或四哥空格的垂直悬挂缩进（在文件内保持一致）。如果函数的名称和签名描述很多，可以使用一行文档字符串进行适当描述，也可以省略。

#### `Args:`

按名称列出每个参数。后面接说明，并以冒号分割，后面跟空格或换行符。如果说明太长，超出一行 80 字符，则使用比参数多 2 个或 4 个的悬挂缩进（与文件中其余文档字符串保持一致）。如果代码不包括响应的类型注释，则说明其需要的类型。如果一个函数接受可变长度列表 `*foo` 或/和任意关键字参数 `**bar` ，文档字符串中应写成 `*foo` 和 `**bar` 。

#### `Returns:` 或 `Yields: for generators`

描述返回值的类型和语义。如果函数只返回 `None` ，则可以不需要。如果文档字符串以 `Returns` 或 `Yidles` 开头（例如： `"""Returns row from Bigtable as a tuple of strings."""`）和代码足以描述返回值，那么可以省略

#### `Raises:`

列出所有与接口相关的异常，然后是描述。使用类似的异常名 + 冒号 + 空格或换行符和悬挂缩进样式。如下 `Args:` 所示的悬挂缩进样式。不应记录不按照文档字符串中描述的规范使用 API 引发的异常（因为这会和 API 本身的异常行为自相矛盾）

```python
def fetch_smalltable_rows(table_handle: smalltable.Table,
                          keys: Sequence[Union[bytes, str]],
                          require_all_keys: bool = False,
) -> Mapping[bytes, Tuple[str]]:
    """Fetches rows from a Smalltable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by table_handle.  String keys will be UTF-8 encoded.

    Args:
        table_handle: An open smalltable.Table instance.
        keys: A sequence of strings representing the key of each table
          row to fetch.  String keys will be UTF-8 encoded.
        require_all_keys: Optional; If require_all_keys is True only
          rows with values set for all keys will be returned.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {b'Serak': ('Rigel VII', 'Preparer'),
         b'Zim': ('Irk', 'Invader'),
         b'Lrrr': ('Omicron Persei 8', 'Emperor')}

        Returned keys are always bytes.  If a key from the keys argument is
        missing from the dictionary, then that row was not found in the
        table (and require_all_keys must have been False).

    Raises:
        IOError: An error occurred accessing the smalltable.
    """
```

和下面一样， 使用换行符的 Args 的也是允许的：

```python
def fetch_smalltable_rows(table_handle: smalltable.Table,
                          keys: Sequence[Union[bytes, str]],
                          require_all_keys: bool = False,
) -> Mapping[bytes, Tuple[str]]:
    """Fetches rows from a Smalltable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by table_handle.  String keys will be UTF-8 encoded.

    Args:
      table_handle:
        An open smalltable.Table instance.
      keys:
        A sequence of strings representing the key of each table row to
        fetch.  String keys will be UTF-8 encoded.
      require_all_keys:
        Optional; If require_all_keys is True only rows with values set
        for all keys will be returned.

    Returns:
      A dict mapping keys to the corresponding table row data
      fetched. Each row is represented as a tuple of strings. For
      example:

      {b'Serak': ('Rigel VII', 'Preparer'),
       b'Zim': ('Irk', 'Invader'),
       b'Lrrr': ('Omicron Persei 8', 'Emperor')}

      Returned keys are always bytes.  If a key from the keys argument is
      missing from the dictionary, then that row was not found in the
      table (and require_all_keys must have been False).

    Raises:
      IOError: An error occurred accessing the smalltable.
    """
```

### 3.8.4 类

应该在类定义的下面有一个描述类的文档字符串。如果类具有公共属性，则应该记录在 `Attributes` 中，并遵循于函数的 Args 相同的格式

```python
class SampleClass:
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """

    def __init__(self, likes_spam=False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0

    def public_method(self):
        """Performs operation blah."""
```

#### 3.8.5 块和内联注释

最后一个需要注释的地方是代码中比较棘手的部分。为了便于下次理解代码，推荐编写时就加注释。有些复杂逻辑会友好几行注释。并不是所有注释都在行尾

```python
# We use a weighted dictionary search to find out where i is in
# the array.  We extrapolate position based on the largest num
# in the array and the array size and then do binary search to
# get the exact number.

if i & (i-1) == 0:  # True if i is 0 or a power of 2.
```

为了提高可读性，注释字符 `#` 应该在离代码至少两个空格的地方开始，然后在注释文本本身之前有一个空格。

另一方面，不要描述代码。不要假设阅读代码阅读代码的人比你更了解 Python （即使你不想这么做）。

```python
# BAD COMMENT: Now go through the b array and make sure whenever i occurs
# the next element is i+1
```

### 3.8.6 标点符号、拼写和语法

注意标点符号、拼写和语法。好的注释更容易阅读。

注释应像陈述文本一样可读，并有适当的大写和标吊符号。在大多数情况下，完整的句子比片段更容易阅读。像代码行尾的段注释，可能不是很正式，但是你应该有自己的风格。

虽然被代码审阅者指出标点符号使用不准确（在用分号的地方用了逗号）会很不爽，但保持源代码简洁和可读性却是非常重要的。正确使用标点符号、拼写和语法有主意提高代码质量。

## 3.9 类

类不需要显示的从 `object` 继承（除非为了与 Python 2 兼容）。

!!! note "新式类"

    ```python
    class SampleClass:
        pass


    class OuterClass:

        class InnerClass:
            pass
    ```

!!! note "老式类"

    ```python
    class SampleClass(object):
        pass


    class OuterClass(object):

        class InnerClass(object):
            pass
    ```

## 3.10 字符串

即使参数都是字符串，也要使用 `format` 方法或 `%` 。对于 `+` 、 `%` 和 `format` 自行决定那种方式更好。

!!! success "推荐"

    ```python
    x = a + b
    x = '%s, %s!' % (imperative, expletive)
    x = '{}, {}'.format(first, second)
    x = 'name: %s; score: %d' % (name, n)
    x = 'name: {}; score: {}'.format(name, n)
    x = f'name: {name}; score: {n}'  # Python 3.6+
    ```

!!! fail "不推荐"

    ```python
    x = '%s%s' % (a, b)  # use + in this case
    x = '{}{}'.format(a, b)  # use + in this case
    x = first + ', ' + second
    x = 'name: ' + name + '; score: ' + str(n)
    ```

避免在循环中使用 `+` 和 `+=` 累加字符串。由于字符串是不可变，这会创建不必要的临时对象，并导致二次运行时间而不是线性运行时间。相反将每个字符串加到一个列表中，并在结束后 `''.join` 列表（或者将每个子字符串写入 `io.BytesIO` 缓冲区中）。

!!! success "推荐"

    ```python
    items = ['<table>']
    for last_name, first_name in employee_list:
        items.append('<tr><td>%s, %s</td></tr>' % (last_name, first_name))
    items.append('</table>')
    employee_table = ''.join(items)
    ```

!!! fail "不推荐"

    ```python
    employee_table = '<table>'
    for last_name, first_name in employee_list:
        employee_table += '<tr><td>%s, %s</td></tr>' % (last_name, first_name)
    employee_table += '</table>'
    ```

在一个文件中选择一个字符串引号并保持一致。选择 `'` 或 `"` 并一直使用。在字符串上使用其他引号字符时应避免使用 `\\` 转义。

!!! success "推荐"

    ```python
    Python('Why are you hiding your eyes?')
    Gollum("I'm scared of lint errors.")
    Narrator('"Good!" thought a happy Python reviewer.')
    ```

!!! fail "不推荐"

    ```python
    Python("Why are you hiding your eyes?")
    Gollum('The lint. It burns. It burns us.')
    Gollum("Always the great lint. Watching. Watching.")
    ```

对于多行字符串，推荐使用 `"""` 而不是 `'''` 。当且仅当 `'` 用于常规字符串时，项目中的多行非文档字符串可以使用 `'''` 。但文档字符串必须使用 `"""` 。

多行字符串不会随着程序其他部分的缩进而缩进。如果要避免字符串中嵌套额外空白，可以使用哦串联的单行字符串或带有 [`textwrap.dedent()`](https://docs.python.org/3/library/textwrap.html#textwrap.dedent) 的多行字符串来删除每行的初始空白。

!!! fail "不推荐"

    ```python
    def foo():
        long_string = """This is pretty ugly.
    Don't do this.
    """
    ```

!!! success "推荐"

    ```python
    long_string = """This is fine if your use case can accept
        extraneous leading spaces."""
    ```

    ```python
    long_string = ("And this is fine if you cannot accept\n" +
                   "extraneous leading spaces.")
    ```

    ```python
    long_string = textwrap.dedent("""\
        This is also fine, because textwrap.dedent()
        will collapse common leading spaces in each line.""")
    ```

## 3.11 文件和 Socket

使用文件和 Socket 时要显式关闭。打开不必要的文件、 Socket 或其他类似文件的对象有以下缺点：

- 消耗有限的系统资源，例如文件描述符。如果这些对象在使用后没有立即关闭，那么这种代码可能会把系统资源耗尽。
- 整个程序中的共享文件和 Socket 可能在逻辑关闭后无意之中被读写。如果已经关闭，当尝试读写会抛出异常，从而尽早发现问题。

此外尽管文件和 Socket 的文件对象会在销毁时自动关闭，但这种将文件对象的生命周期与文件的状态绑定在一起的错发是不推荐的：

- 无法保证文件对象何时被销毁。不同的 Python 实现使用不同的内存管理技术，比如延迟的垃圾收集，这可能会任意和无限的增加对象的生存期。
- 对文件的意外引用，例如在全局或异常回溯中，可能使文件保留的时间超出预期。


管理文件的首选方法是使用 [`with` 语句](http://docs.python.org/reference/compound_stmts.html#the-with-statement)：

```python
with open("hello.txt") as hello_file:
    for line in hello_file:
        print(line)
```

对于不支持 `with` 语句的文件对象，请使用 `contextlib.closing()` ：

```python
with contextlib.closing(urllib.urlopen("http://www.python.org/")) as front_page:
    for line in front_page:
        print(line)
```

## 3.12 TODO 注释

对于临时的、短期的解决方案或足够好但不完美的代码，使用 `TODO` 注释。

`TODO` 注释以字符串 `TODO` 开头，全部大写，括号中包括姓名、电子邮件地址或其他标识符，其中包含关于问题的最佳上下文。接下来是解释要做什么。

其目的是有一个一直的 `TODO` 风格，可以通过搜索获取更多的细节信息。引用 `TODO` 的人不一定必须修复这个问题。因此，当创建 `TODO` 时，应该提供你的名字。

```python
# TODO(kl@gmail.com): Use a "*" here for string repetition.
# TODO(Zeke) Change this to use relations.
```

如果 `TODO` 是 “At a future date do something” 这种类型，请确保其中包含一个具体日期（ `Fix by November 2009` ）或者一个具体事件（ `Remove this code when all clients can handle XML responses.` ）。


## 3.13 导入格式

导入应该在单独的行上，[类型导入有例外](#31912-typing)

!!! success "推荐"

    ```python
    import os
    import sys
    from typing import Mapping, Sequence
    ```

!!! fail "不推荐"

    ```python
    import os, sys
    ```

导入总是在文件的顶部，放在任何模块注释和文档字符串后面，放在模块全局变量和常量之前。导入应该从最通用到最不通用分组：

1. Future 导入语句：

    ```python
    from __future__ import absolute_import
    from __future__ import division
    from __future__ import print_function
    ```

2. Python 标准库导入：

    ```python
    import sys
    ```

3. 第三方模块导入：

    ```python
    import tensorflow as tf
    ```

4. 代码库子包导入：

    ```python
    from otherproject.ai import mind
    ```

5. **不推荐：**特定于应用程序的导入，它们属于与此文件相同的顶级子包的一部分。

    ```python
    from myproject.backend.hgwells import time_machine
    ```

    之前的 Google Python 风格可能是这么做的，但现在已经不推荐了。新的代码不要这么做。简单地将特定于应用程序的子包导入与其他子包导入相同对待。

在每个分组中，根据每个模块完整包路径（ `from path import ...` 的 `path` ）导入，顺序按字母排序，忽略大小写。代码可以选择在导入分区之间放一个空行。

```python
import collections
import queue
import sys

from absl import app
from absl import flags
import bs4
import cryptography
import tensorflow as tf

from book.genres import scifi
from myproject.backend import huxley
from myproject.backend.hgwells import time_machine
from myproject.backend.state_machine import main_loop
from otherproject.ai import body
from otherproject.ai import mind
from otherproject.ai import soul

# 之前风格的代码可能会将一些导入放在这里:
#from myproject.backend.hgwells import time_machine
#from myproject.backend.state_machine import main_loop
```

## 2.14 声明

通常每行只有一个语句。

但是只有当真个语句符合一行时，才可以将测试结果放在于测试相同的行上。特别是， `try/except` 永远不要这么做，因为 `try` 和 `except` 不能同时放在同一行上，并且只有 `if` 在没有 `else` 的情况下才能这么做。

!!! success "推荐"

    ```python
    if foo: bar(foo)
    ```

!!! fail "不推荐"

    ```python
    if foo: bar(foo)
    else:   baz(foo)

    try:               bar(foo)
    except ValueError: baz(foo)

    try:
        bar(foo)
    except ValueError: baz(foo)
    ```

## 3.15 访问器

如果访问函数很简单，那么应该使用公共变量而不是使用访问器函数，避免调用函数带来的额外开销。当功能多了，可以使用 `property` 语法。

## 3.16 命名

`module_name` 、 `package_name` 、 `ClassName` 、 `method_name` 、 `ExceptionName` 、 `function_name` 、 `GLOBAL_CONSTANT_NAME` 、 `global_var_name` 、 `instance_var_name` 、 `function_parameter_name` 、
 `local_var_name` 。

函数名、变量名和文件名都应该是描述性的，避免使用缩写，特别是，不要使用让项目外的读者感到含糊不清或不熟悉缩写，也不要通过删除单词中的字母来缩写。

只使用 `.py` 文件扩展名，不要使用破折号。

### 3.16.1 禁用的命名

- 单个字符的名称。除非特别允许的情况。

    - 计数器或迭代器（例如： `i` ， `j` ， `k` ， `v` 等等）
    - 作为 `try/except` 语句的异常标识符 `e` 。
    - 作为 `with` 语句声明的文件对象 `f`

    请注意不要滥用单字符命名。一般来说，描述性应该与名称的可见性范围成正比。例如： `i` 可能是在五行的代码块中是个好名称，但是在多嵌套范围内，可能就太含糊了。

- 任何包/模块中的破折号（ `-` ）
- `__double_leading_and_trailing_underscore__` 前置和后置双下划线命名（ Python 保留名称）
- 不礼貌的用语

### 3.16.2 命名约定

- “内部”是指模块内部的，或者类中受保护或私有的。
- 前置单个下划线（ `_` ）用来保护模块变量。虽然在实例变量或方法中加入双下划线（ `__` ） 可以私有化（使用名称管理），但并不鼓励这么做，因为会影响可读性或可测试性，而且不是真正的私有。
- 将相关类和顶级函数放在一个模块中。不像 Java ，没必要每个模块只有一个类。
- 对于雷鸣使用错峰命名，但对于模块名使用小写加下划线（ `lower_with_under.py` ）。虽然有一些旧的模块名为驼峰命名（ `CapWords.py` ），但现在不推荐这么做，因为模块名和类名相同时，容易混淆。（例如：是应该导入 `import StringIO` 还是 `from StringIO import StringIO` ？）
- 下划线可能出现在但单元测试中，从 `test` 开始用下划线分割逻辑组件的名称，即使这些组件使用驼峰命名。可能格式是： `test<MethodUnderTest>_<state>` ，例如：　`testPop_EmptyStack` 是可行的。对测试方法的命名没有强制规定。

### 3.16.3 文件名

Python 文件名扩展名必须以 `.py` 结尾，而且不能包含破折号（ `-` ）。如果希望可执行文件不需要扩展名就能访问，可以使用连接符号或包含 `exec "$0.py" "$@"` 的脚本包装器。

### 3.16.4 基于 [Guido’s](https://en.wikipedia.org/wiki/Guido_van_Rossum) 推荐的派生准则

| Type                        | Public                    | Internal                        |
| --------------------------- | ------------------------- | ------------------------------- |
| Packages                    | lower_with_under          |                                 |
| Modules                     |lower_with_under           |_lower_with_under                |
| Classes                     |CapWords                   |_CapWords                        |
| Exceptions                  |CapWords                   |                                 |
| Functions                   |lower_with_under()         |_lower_with_under()              |
| Global/Class Constants      |CAPS_WITH_UNDER            |_CAPS_WITH_UNDER                 |
| Global/Class Variables      |lower_with_under           |_lower_with_under                |
| Instance Variables          |lower_with_under           |_lower_with_under (protected)    |
| Method Names                |lower_with_under()         |_lower_with_under() (protected)  |
| Function/Method Parameters  |lower_with_under           |                                 |
| Local Variables             |lower_with_under           |                                 |

## 3.17 Main

在 Python 中， `pydoc` 和单元测都是要求模块是可导入的。如果一个文件打算用作可执行文件，它的主要功能应该在 `main()` 函数中，并且在执行主程序之前，代码应该检查 `if __main__ == '__main__'` ，以便在导入模块时不执行它。

当使用 [`absl`](https://github.com/abseil/abseil-py) 时，使用 `app.run` ：

```python
from absl import app
...

def main(argv):
    # process non-flag arguments
    ...

if __name__ == '__main__':
    app.run(main)
```

或者：

```python
def main():
    ...

if __name__ == '__main__':
    main()
```

导入模块时，将执行顶层的所有代码。注意不要调用函数、创建对象或执行其他对在对文件进行 `pydoc` 时不应该执行的操作。

## 3.18 函数长度

推荐小而集中的功能。风情无法

过长的函数有时是可以接受的，所以对函数的长度并没有硬性限制。如果一个函数超过了 40 行，需要考虑一下在不损害程序结构的情况下将其分解。

即使长函数运行良好，未来有人增加了新功能，可能导致隐藏的 BUG 。保持函数的简洁和简单，便于他人理解和修改。

## 3.19 类型标注

### 3.19.1 一般规则

- 熟悉 [PEP-484](https://www.python.org/dev/peps/pep-0484/)
- 在方法中，如果需要提供类型信息，仅标注 `self` 或 `cls` 。例如：

    ```python
    @classmethod 
    def create(cls: Type[T]) -> T: 
        return cls()
    ```

- 任何类型的变量或者无法表达返回类型，请使用 `Any` 。
- 不需要对模块中的所有函数标注。
    - 至少要标注公共 API
    - 在安全性和清晰性以及灵活性之间找到一个平衡点
    - 容易导致类型相关的错误（之前的错误或复杂性）
    - 对难以理解的代码进行标注
    - 类型标注让代码变得稳定。多数情况下，在收悉的代码中标注所有函数，并不会失去太多灵活性。

### 3.19.2 断行

尽量遵循现有[缩进规则](#34)。

在标注之后，许多函数签名变成每行一个参数。

```python
def my_method(self,
              first_var: int,
              second_var: Foo,
              third_var: Optional[Bar]) -> int:
    ...
```

尽量在变量之间断行，不要在变量名和类型标注之间断行。如果所有内容都在一行上，就不需要管了。

```python
def my_method(self, first_var: int) -> int:
    ...
```

如果函数名、最后一个参数和返回类型的组合太长，则在新航中缩进 4 个字符。

```python
def my_method(
    self, first_var: int) -> Tuple[MyLongType1, MyLongType1]:
    ...
```

当返回类型与最后一个参数不适合在同一行时，推荐的做法是在新行缩进 4 个字符，并将结束括号与 `def` 对齐。

!!! success "推荐"

    ```python
    def my_method(
        self, other_arg: Optional[MyLongType]
    ) -> Dict[OtherLongType, MyLongType]:
    ...
    ```

Pylint 允许将闭括号移动到一个新行，并与开始行对其，但这么做不易于阅读。

!!! fail "不推荐"

    ```python
    def my_method(self,
                other_arg: Optional[MyLongType]
                ) -> Dict[OtherLongType, MyLongType]:
    ...
    ```

有时类型太长，不能放在单独行上（尽量保持子类型不间断）：

```python
def my_method(
    self,
    first_var: Tuple[List[MyLongType1],
                     List[MyLongType2]],
    second_var: List[Dict[
        MyLongType3, MyLongType4]]) -> None:
    ...
```

如果单个名称和类型太长了，就要考虑类型别名。最终的办法是在冒号后面换行，并使用 4 个字符缩进。

!!! success "推荐"

    ```python
    def my_function(
        long_variable_name:
            long_module_name.LongTypeName,
    ) -> None:
        ...
    ```

!!! fail "不推荐"

    ```python
    def my_function(
        long_variable_name: long_module_name.
            LongTypeName,
    ) -> None:
        ...
    ```

### 3.19.3 提前声明

如果需要使用来自同一模块但没有定义的类名，例如：如果你需要在声明类中使用类，或者使用下面定义的类，请使用字符串的类名。

```python
class MyClass:

  def __init__(self,
        stack: List["MyClass"]) -> None:
```

### 2.19.4 默认值

根据 [PEP-008](https://www.python.org/dev/peps/pep-0008/#other-recommendations) ，周围的空格仅用于同事具有类型标注或默认值的参数。

!!! success "推荐"

    ```python
    def func(a: int = 0) -> int:
        ...
    ```

!!! fail "不推荐"

    ```python
    def func(a:int=0) -> int:
        ...
    ```

### 3.19.5 NoneType

在 Python 中， `NoneType` 是“头等”类型，为了方便拼写， `None` 是 `NoneType` 的别名。如果一个参数可以为 `None` ，那么必须被声明。可以使用 `Union` 。但如果只有一个类型，请使用 `Optional` 。

显式使用 `Optional` 替代隐式 `Optional` 。 PEP-484 的早期版本允许将 `Text = None` 解释为 `Optional[Text] = None` ，但现在不推荐这么做。

!!! success "推荐"

    ```python
    def func(a: Optional[Text], b: Optional[Text] = None) -> Text:
        ...
    def multiple_nullable_union(a: Union[None, Text, int]) -> Text
        ...
    ```

!!! fail "不推荐"

    ```python
    def nullable_union(a: Union[None, Text]) -> Text:
        ...
    def implicit_optional(a: Text = None) -> Text:
        ...
    ```

### 3.10.6 类型别名

可以声明复杂类型的别名。别名应该是驼峰命名。如果别名只在这个模块中使用，则应该使用前置下划线（ `_Private` ） 。

例如，如果模块的名称和类型太长：

```python
_ShortName = module_with_long_name.TypeWithLongName
ComplexMap = Mapping[Text, List[Tuple[int, int]]]
```

其他例子包括复杂的嵌套类型和一个函数（作为元组）的多个返回值。

### 3.19.7 忽略类型

可以使用 `# type: ignore.` 禁用检查带有类型标注的行。

`pytype` 有一个针对特定类型的禁用选项（类似与 lint）

```python
# pytype: disable=attribute-error
```

### 3.18.9 输入变量

如果一个内部变量具有难以或不可推断的类型，可以通过两种方式指定其类型。

#### 注释类型标注

在行尾使用 `# type:` 注释标注类型。

```python
a = SomeUndecoratedFunction()  # type: Foo
```

#### 签名标注

在变量名和值之间使用默冒号和类型，如函数参数一样：

```python
a: Foo = SomeUndecoratedFunction()
```

### 3.19.9 元组 VS 列表

列表的类型化只能包含单一类型的对象。元组类型化可以有单个重复类型，也可以有一组不同的元素类型。后者通常作为函数的返回类型。

```python
a = [1, 2, 3]  # type: List[int]
b = (1, 2, 3)  # type: Tuple[int, ...]
c = (1, "2", 3.5)  # type: Tuple[int, Text, float]
```

### 3.19.10 TypeVars

Python 类型系统具有[泛型](https://www.python.org/dev/peps/pep-0484/#generics)，工厂函数 `TypeVar` 是使用泛型常用的方法。

例如：

```python
from typing import List, TypeVar
T = TypeVar("T")
...
def next(l: List[T]) -> T:
return l.pop()
```

可以约束 TypeVar ：

```python
AddableType = TypeVar("AddableType", int, float, Text)
def add(a: AddableType, b: AddableType) -> AddableType:
    return a + b
```

`typing` 模块中一个常见的预定义类型变量是 `AnyStr` 。可以用于标注 `types` 、 `unicode` ，并且必须全部是相同的类型。

```python
from typing import AnyStr
def check_length(x: AnyStr) -> AnyStr:
    if len(x) <= 42:
        return x
    raise ValueError()
```

### 3.19.12 字符串类型

正确的对字符串进行注释，取决于打算将代码用于哪个版本的 Python 。

对于 PYthon 3 代码，推荐用 `str` 。 `Text` 也是可以接受的。使用时保持一致。

对于要兼容 Python 2 的代码，使用 `Text` 。在极少数情况下， `str` 可能会出现问题。通常为了在两个 Python 版本中返回类型不同时增强兼容性。不要使用 `unicode` ，因为在 Python 3 中不存在。

之所以存在这种差异，是因为 `str` 的使用和 Python 版本有关。

!!! fail "不推荐"

    ```python
    def py2_code(x: str) -> unicode:
        ...
    ```

要处理二进制数据的代码，请使用 `types` ：

```python
def deals_with_binary_data(x: bytes) -> bytes:
    ...
```

兼容 Python 2 的代码处理文本数据（ Python 2 中的 `str` 和 `unicode` ， Python 3 中的 `str` ） ，请使用 `Text` 。对于 Python 3 ，只处理文本数据的代码，推荐 `str` 。

```python
from typing import Text
...
def py2_compatible(x: Text) -> Text:
    ...
def py3_only(x: str) -> str:
    ...
```

如果类型是字节或文本，使用 `Unicode` ，并使用适当的文本类型。

```python
from typing import Text, Union
...
def py2_compatible(x: Union[bytes, Text]) -> Union[bytes, Text]:
    ...
def py3_only(x: Union[bytes, str]) -> Union[bytes, str]:
    ...
```

如果函数的所有字符串类型总是相同的，例如，如果返回类型与上面代码中的参数类型相同，则使用 AnyStr。

### 3.19.12 从 Typing 中导入

对于 `typing` 模块中的类，始终导入类本身。显式的允许从 `typing` 模块在一行导入多个特定的类。

```python
from typing import Any, Dict, Optional
```

考虑到这种从 `typing` 中导入到本地命名空间的方式， `typing` 中的任何名称都应该被看成关键字，而不是 Python 中的代码。如果模块中的类型和现有名称冲突，则使用 `import x as y` 导入。

```python
from typing import Any as AnyType
```

### 3.19.13 条件导入

只有为了避免运行时进行类型检查时才使用条件导入。不推荐这种模式。建议使用其他方法，比如重构代码然后顶级导入。

可以将仅用于类型注释的导入放在 `if TYPE_CHECKING:` 代码块中，包含如下情况：

- 条件导入类型需要作为字符串引用，这样，实际计算注释表达式的地方才能向前兼容 Python 3.6 。
- 这里定义的条目仅用于类型注释。其中包括别名。否则将是一个运行时错误，因为模块在运行时不会导入。
- 所有正常导入后的代码块应该是正常的。
- 导入列表中没有空行
- 将导入列表进行常规排序

```python
import typing
if typing.TYPE_CHECKING:
    import sketch
def f(x: "sketch.Sketch"): ...
```

### 3.19.14 循环导入

类型引起的循环导入会造成代码异常。这种代码需要重构。虽然技术上可以保持循环依赖，但构建系统不允许这么做，因为每个模块都必须依赖另一个模块。

将创建循环依赖项导入的模块替换为 `Any`  。设置有意义的[别名](#3106) ，并在此模块中使用实际类型（ Any 的任何属性都是 Any ） 。别名定义应该与上次导入分开一行。

```python
from typing import Any

some_mod = Any  # some_mod.py imports this module.
...

def my_method(self, var: "some_mod.SomeType") -> None:
    ...
```

### 3.19.15 泛型

进行标注时，推荐为泛型类型指定类型参数，否则，会[假定泛型的参数为 `Any`](https://www.python.org/dev/peps/pep-0484/#the-any-type) 。

```python
def get_names(employee_ids: List[int]) -> Dict[int, Any]:
    ...
```

```python
# These are both interpreted as get_names(employee_ids: List[Any]) -> Dict[Any, Any]
def get_names(employee_ids: list) -> Dict:
      ...

def get_names(employee_ids: List) -> Dict:
     ...

```

如果泛型的最佳类型是 `Any` 的话，请明确标注。但在许多情况下 [`TypeVar`](#31910-typevars) 可能更合适。

```python
def get_names(employee_ids: List[Any]) -> Dict[Any, Text]:
  """Returns a mapping from employee ID to employee name for given IDs."""
```

```python
T = TypeVar('T')
def get_names(employee_ids: List[T]) -> Dict[T, Text]:
  """Returns a mapping from employee ID to employee name for given IDs."""
```

<!-- markdownlint-restore -->