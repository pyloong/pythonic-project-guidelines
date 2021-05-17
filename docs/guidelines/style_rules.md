# Python 风格规范

> 本文档为 [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html) 第二章 [Python Style Rules](https://google.github.io/styleguide/pyguide.html#3-python-style-rules) 的译文。
>
> 最后更新时间： 2021-05-12
>
> 如果有翻译错误或表述不准确的问题，欢迎提交 PR，感谢您的参与。

## 3.1 分号

不要在行尾加分号，也不要用分号将两条命令放在同一行。

## 3.2 行长度

每行不超过80个字符。

例外：

- 长的导入模块语句
- 注释里的 URL 、路径名和长标识
- 不包含空格，不方便跨行拆分的长字符串模块级常量，如 URL 或路径名
    - Pylint 禁用注释。（例如： `# pylint: disable=invalid-name` ）
  
除非 `with` 语句需要三个或更多的上下文管理器，否则不要使用反斜杠连接行。

Python 会将[圆括号、方括号和花括号中的行隐式的连接起来](https://docs.python.org/3/reference/lexical_analysis.html#implicit-line-joining)，你可以利用这个特点 。如果需要，你可以在表达式外围增加一对额外的圆括号。

!!! success "推荐"

    ```python
    foo_bar(self, width, height, color='black', design=None, x='foo',
            emphasis=None, highlight=0)

    if (width == 0 and height == 0 and
            color == 'red' and emphasis == 'strong'):
    ```

如果一个文本字符串在一行放不下，可以使用圆括号来实现隐式行连接。

```python
x = ('This will build a very long long '
     'long long long long long long string')
```

在注释中，如果必要，将长的 URL 放在一行上。

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

在定义表达式跨越三行或更多行的 `with` 语句时，允许使用反斜杠连接。对于两行表达式，请使用嵌套 `with` 语句：

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

注意上面例子中的元素缩进。你可以在本文的[缩进](#34)部分找到解释。

在所有其他情况下，如果一行超过80个字符，并且 [yapf](https://github.com/google/yapf/) 自动格式化程序无法帮助使该行低于限制，则允许该行超过此最大值。建议作者在合理的情况下，根据上述注释手动拆分行。

## 3.3 括号

宁缺毋滥的使用括号。

除非是用于实现行连接，否则不要在返回语句或条件语句中使用括号，不过在元组两边使用括号是可以的。

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

用4个空格来缩进代码。

绝对不要用 `tab`，也不要 `tab` 和空格混用。对于行连接的情况，你应该要么垂直对齐换行的元素（见[行长](#32)部分的示例），或者使用4空格的悬挂式缩进（这时第一行不应该有参数）。

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

### 3.4.1 在序列的末尾是否加逗号？

只有在序列结束符 `]` 、 `)` 或 `}` 与最后一个元素不在同一行时才建议使用。末尾逗号的存在还用作对代码自动格式化程序 [YAPF](https://pypi.org/project/yapf/) 的提示，以引导它在最后一个元素之后出现时，自动将容器中每个条目格式化为一行。

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

## 3.5 空行

顶级定义之间空两行, 方法定义之间空一行

- 顶级定义之间空两行，比如函数或者类定义。
- 方法定义，类定义与第一个方法之间，都应该空一行。
- 在 `def` 函数定义之后不需要空行。
- 函数或方法中，某些地方要是你觉得合适，就空一行。

## 3.6 空格

按照标准的排版规范来使用标点两边的空格。

括号内不要有空格。

!!! success "推荐"

    ```python
    spam(ham[1], {eggs: 2}, [])
    ```

!!! fail "不推荐"

    ```python
    spam( ham[ 1 ], { eggs: 2 }, [ ] )
    ```

不要在逗号，分号，冒号前面加空格。但应该在它们后面加（除了在行尾）。

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

参数列表、索引或切片的左括号前不应加空格。

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

行尾不需要空格。

在二元操作符两边都加上一个空格，比如赋值（`=`）、比较（`==`、`<`、`>`、`!=`、`<>`、`<=`、`>=`、`in`、`not in`、`is`、`is not`），布尔（`and`、`or`、`not`）。
至于算术操作符（`+`、`-`、`*`、`/`、`//`、`%`、`**`、`@`）两边的空格该如何使用，需要你自己好好判断。不过两侧务必要保持一致。

!!! success "推荐"

    ```python
    x == 1
    ```

!!! fail "不推荐"

    ```python
    x<1
    ```

当 `=` 用于指示关键字参数或默认参数值时，不要在其两侧使用空格。但有一个例外：[当存在类型注释时](#3194)，在默认参数值的 `=` 周围使用空格。

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

不要用空格来垂直对齐多行间的标记，因为这会造成维护的负担（适用于 `:`、`#`、`=` 等）：

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

## 3.7 [Shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))

大部分 `.py` 文件不必以 `#!` 作为文件的开始。根据 [PEP-394](https://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0394/)，程序的 `main` 文件应该以 `#!/usr/bin/env python3` （用于支持虚拟环境）或者 `#!/usr/bin/python3` 开始。

内核使用这一行来查找 Python 解释器，但是 Python 在导入模块时会忽略这一行。因此只有在打算直接执行的文件上添加才有必要。

## 3.8 注释和文档字符串

确保对模块, 函数, 方法和行内注释使用正确的风格。

### 3.8.1 文档字符串

Python 有一种独一无二的的注释方式： 使用文档字符串。文档字符串是包、模块、类或函数里的第一个语句。这些字符串可以通过对象的 `__doc__` 成员被自动提取，并且被 `pydoc` 所用（你可以在你的模块上运行 `pydoc` 试一把，看看它长什么样）。
我们对文档字符串的惯例是使用三重双引号 `"""` （参见： [PEP-257](https://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0257/) ）。一个文档字符串应该这样组织（通常一行不超过 80 个字符），先是一行以句号，问号或惊叹号结尾的概述（或者该文档字符串单纯只有一行）。接着是一个空行，接着是文档字符串剩下的部分，它应该与文档字符串的第一行的第一个引号对齐。下面有更多文档字符串的格式化规范。

### 3.8.2 模块

每个文件应该包含一个许可样板。根据项目使用的许可（例如：`Apache 2.0`、`BSD`、`LGPL`、`GPL`），选择合适的样板。

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

下文所指的函数，包括函数，方法，以及生成器。

一个函数必须要有文档字符串，除非它满足以下条件：

- 外部不可见
- 非常短小
- 简单明了

文档字符串应该提供足够的信息，当别人编写代码调用该函数时，他不需要看一行代码，只要看文档字符串就可以了。文档字符串应该是描述性的（ `"""Fetches rows from a Bigtable."""` ） 而不是命令式的（ `"""Fetch rows from a Bigtable."""` ） ，除了 `@property` 数据描述符，它应该使用[与属性相同的样式](#384)。文档字符串应该包含函数做什么，以及输入和输出的详细描述。通常，不应该描述“怎么做”，除非是一些复杂的算法。对于复杂的代码，在代码旁边加注释会比使用文档字符串更有意义。

重写基类中的方法时，用一个简单的文档字符串引导读者查看被覆盖方法的文档字符串，例如： `"""See base class."""` 。这样做的好处是，无需重复基本方法中的文档字符串信息。但是，如果重写方法的行为发生了改变，或者需要提供详细信息（例如：记录额外副作用），那么重写方法至少需要通过文档字符串来描述这些差异。

关于函数的几个方面应该在特定的小节中进行描述记录。这几个方面如下文所述，每节应该以一个标题行开始，标题行以冒号结尾。除标题行外，小节的其他内容应被缩进两个或四个空格（在文件内保持一致）。如果函数的名称和签名具有足够的信息，可以使用单行文档字符串进行适当描述，那就可以省略这些部分。

#### *`Args:`*

列出每个参数的名字，在名字后使用一个冒号和一个空格，分隔对该参数的描述。如果描述太长超过了单行80字符，使用2或者4个空格的悬挂缩进（与文件其他部分保持一致）。描述应该包括所需的类型和含义。如果一个函数接受 `*foo`（可变长度参数列表）或者 `**bar`（任意关键字参数），应该详细列出 `*foo` 和 `**bar` 。

#### *`Returns:`（或者 `Yields:` 用于生成器）*

描述返回值的类型和语义。

- 如果函数返回 `None` ，这一部分可以省略。
- 如果文档字符串以 `Returns` 或 `Yields` 开头（例如： `"""Returns row from Bigtable as a tuple of strings."""`），并且开始的句子足以描述返回值，那么也可以省略。

#### *`Raises:`*

列出与接口有关的所有异常，然后给出说明。使用类似的异常名称 + 冒号 + 空格或换行符，并按 `Args：` 中所述悬挂缩进样式。如果违反了文档字符串中指定的 API，则不应该记录引发的异常（因为这会使违反 API 的行为成为 API 的一部分）。

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

如下所示， Args 中参数换行也是允许的：

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

类应该在其定义下有一个用于描述该类的文档字符串。如果你的类有公共属性（`Attributes`），那么文档中应该有一个属性（`Attributes`）段，并且应该遵守和[函数参数](#383)相同的格式：

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

#### 3.8.5 块注释和行注释

最需要写注释的是代码中那些技巧性的部分。如果你在下次[代码审查](http://en.wikipedia.org/wiki/Code_review)的时候必须解释一下，那么你应该现在就给它写注释。对于复杂的操作，应该在其操作开始前写上若干行注释，对于不是一目了然的代码，应在其行尾添加注释。

```python
# We use a weighted dictionary search to find out where i is in
# the array.  We extrapolate position based on the largest num
# in the array and the array size and then do binary search to
# get the exact number.

if i & (i-1) == 0:  # True if i is 0 or a power of 2.
```

为了提高可读性，注释字符 `#` 应该至少离开代码两个空格，然后在注释本身的文本之前至少有一个空格。

另一方面，绝不要描述代码。假设阅读代码的人比你更懂 Python，他只是不知道你的代码要做什么。

```python
# BAD COMMENT: Now go through the b array and make sure whenever i occurs
# the next element is i+1
```

### 3.8.6 标点符号、拼写和语法

注意标点符号、拼写和语法。好的注释更容易阅读。

注释应该像叙事文本一样可读，有适当的大写和标点符号。在许多情况下，完整的句子比句子片段更具可读性。较短的注释，例如代码行末尾的注释，有时可能不那么正式，但应该与你的风格保持一致。

虽然被代码审阅者指出标点符号使用不准确（在用分号的地方用了逗号）的感觉会很不爽，但源代码保持高度的清晰性和可读性是非常重要的。正确的标点、拼写和语法有助于实现这一目标。

## 3.10 字符串

即使参数都是字符串，也要使用 [f-string](https://docs.python.org/3/reference/lexical_analysis.html#f-strings)， `%` 操作符或者 `format` 方法格式化字符串。不过也不能一概而论，你需要在 `+` 和 `%`（或 `format`）之间好好判定。不要将 `%` 或 `format` 方法用于纯连接。

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

避免在循环中用 `+` 和 `+=` 操作符来累加字符串。

由于字符串是不可变的，这样做会创建不必要的临时对象，且导致二次方而不是线性的运行时间。尽管这种常见的累加可以在 CPython 上进行优化，但这是一个实现细节。应用优化的条件不容易预测，并且可能会改变。作为替代方案，你可以将每个子串加入列表，然后在循环结束后用 `''.join` 连接列表（也可以将每个子串写入一个 `io.StringIO` 缓存中）。

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

在同一个文件中，保持使用字符串引号的一致性。使用单引号 `'` 或者双引号 `"` 之一用以引用字符串，并在同一文件中沿用。在字符串内可以使用另外一种引号，以避免在字符串中使用 `\\` 转义。

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

为多行字符串使用三重双引号 `"""` 而非三重单引号 `'''` 。当且仅当项目中使用单引号 `'` 来引用字符串时，才可能会使用三重 `'''` 为非文档字符串的多行字符串来标识引用。文档字符串必须使用三重双引号 `"""` 。

多行字符串不会随程序其余部分的缩进而缩进。如果要避免在字符串中嵌入额外的空白，可以使用串联的单行字符串或带有 [`textwrap.dedent()`](https://docs.python.org/3/library/textwrap.html#textwrap.dedent) 的多行字符串来删除每行上的初始空白。

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

!!! fail "不推荐"

    ```python
    def foo():
        long_string = """This is pretty ugly.
    Don't do this.
    """
    ```

### 3.10.1 日志

对于期望以模式字符串（带有 `%` - 占位符）作为第一个参数的日志函数：始终使用字符串文本（而不是 `f-string`）作为它们的第一个参数，并使用模式参数（pattern-parameters）作为后续参数。一些日志实现将未展开的模式字符串收集为可查询字段。它还防止花费时间呈现没有配置记录器输出的消息。

!!! success "推荐"

    ```python
    import tensorflow as tf
    logger = tf.get_logger()
    logger.info('TensorFlow Version is: %s', tf.__version__)
    ```

!!! success "推荐"

    ```python
    import os
    from absl import logging

    logging.info('Current $PAGER is: %s', os.getenv('PAGER', default=''))

    homedir = os.getenv('HOME')
    if homedir is None or not os.access(homedir, os.W_OK):
        logging.error('Cannot write to home directory, $HOME=%r', homedir)
    ```

!!! fail "不推荐"

    ```python
    import os
    from absl import logging

    logging.info('Current $PAGER is:')
    logging.info(os.getenv('PAGER', default=''))

    homedir = os.getenv('HOME')
    if homedir is None or not os.access(homedir, os.W_OK):
        logging.error(f'Cannot write to home directory, $HOME={homedir!r}')
    ```

### 3.10.2 错误消息

错误消息（例如：`ValueError` 等异常的消息字符串，或显示给用户的消息）应遵循三个准则：

- 消息需要与实际错误条件精确匹配。
- 插入的片段必须始终能够清楚地识别。
- 它们应该允许简单的自动化处理（例如 `grepping`）。

!!! success "推荐"

    ```python
    if not 0 <= p <= 1:
        raise ValueError(f'Not a probability: {p!r}')

    try:
        os.rmdir(workdir)
    except OSError as error:
        logging.warning('Could not remove directory (reason: %r): %r',
                        error, workdir)
    ```

!!! fail "不推荐"

    ```python
    if p < 0 or p > 1:  # PROBLEM: also false for float('nan')!
        raise ValueError(f'Not a probability: {p!r}')

    try:
        os.rmdir(workdir)
    except OSError:
        # PROBLEM: Message makes an assumption that might not be true:
        # Deletion might have failed for some other reason, misleading
        # whoever has to debug this.
        logging.warning('Directory already was deleted: %s', workdir)

    try:
        os.rmdir(workdir)
    except OSError:
        # PROBLEM: The message is harder to grep for than necessary, and
        # not universally non-confusing for all possible values of `workdir`.
        # Imagine someone calling a library function with such code
        # using a name such as workdir = 'deleted'. The warning would read:
        # "The deleted directory could not be deleted."
        logging.warning('The %s directory could not be deleted.', workdir)
    ```

## 3.11 文件和 Sockets

在文件和 sockets 结束时，显式的关闭它。

除文件外，sockets 或其他类似文件的对象在没有必要的情况下打开，会有许多副作用，例如：

- 它们可能会消耗有限的系统资源。如文件描述符。如果这些资源在使用后没有及时归还系统，那么用于处理这些对象的代码会将资源消耗殆尽。
- 持有文件将会阻止对于文件的其他诸如移动、删除之类的操作。
- 仅仅是从逻辑上关闭文件和 Sockets，那么它们仍然可能会被其共享的程序在无意中进行读或者写操作。只有当它们真正被关闭后，对于它们尝试进行读或者写操作将会抛出异常，并使得问题快速显现出来。

而且，幻想当文件对象析构时，文件和 sockets 会自动关闭， 试图将文件对象的生命周期和文件的状态绑定在一起的想法，都是不现实的。因为有如下原因：

- 没有任何方法可以确保运行环境会真正的执行文件的析构。不同的 Python 实现采用不同的内存管理技术，比如延时垃圾处理机制。延时垃圾处理机制可能会导致对象生命周期被任意无限制的延长。
- 对于文件意外的引用，会导致对于文件的持有时间超出预期（比如对于异常的跟踪，包含有全局变量等）。


管理文件的首选方法是使用 [`with` 语句](http://docs.python.org/reference/compound_stmts.html#the-with-statement)：

```python
with open("hello.txt") as hello_file:
    for line in hello_file:
        print(line)
```

对于不支持 `with` 语句的类文件对象，请使用 `contextlib.closing()` ：

```python
import contextlib

with contextlib.closing(urllib.urlopen("http://www.python.org/")) as front_page:
    for line in front_page:
        print(line)
```

## 3.12 TODO 注释

为临时代码使用 `TODO` 注释，它是一种短期解决方案，不算完美，但够好了。

`TODO` 注释应该在所有开头处包含 `TODO` 字符串，紧跟着是用括号括起来的你的名字，邮箱地址或其它标识符。然后是一个可选的冒号。接着必须有一行注释，解释要做什么。

主要目的是为了有一个统一的 `TODO` 格式，这样添加注释的人就可以搜索到（并可以按需提供更多细节）。写了 `TODO` 注释并不保证写的人会亲自解决问题。当你写了一个 `TODO`，请注上你的名字。

```python
# TODO(kl@gmail.com): Use a "*" here for string repetition.
# TODO(Zeke) Change this to use relations.
```

如果你的 `TODO` 是 “将来做某事” 的形式，那么请确保你包含了一个指定的日期（2009年11月解决）或者一个特定的事件（等到所有的客户都可以处理 XML 请求就移除这些代码）。

## 3.13 导入格式

每个导入应该独占一行，[`typing` 导入是个例外](#31912)。

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

导入总应该放在文件顶部，位于模块注释和文档字符串之后，模块全局变量和常量之前。导入应该按照从最通用到最不通用的顺序分组：

1. Future 导入语句：

    ```python
    from __future__ import absolute_import
    from __future__ import division
    from __future__ import print_function
    ```

    请参阅[上面](https://google.github.io/styleguide/pyguide.html#from-future-imports)的更多信息。

2. 标准库导入：

    ```python
    import sys
    ```

3. [第三方](https://pypi.org/)模块或包导入：

    ```python
    import tensorflow as tf
    ```

4. 代码库子包导入：

    ```python
    from otherproject.ai import mind
    ```

5. **已弃用：** 与此文件属于同一顶级子包的应用程序特定导入。例如：

    ```python
    from myproject.backend.hgwells import time_machine
    ```

    您可能会发现之前的 Google Python 风格是这么做的，但现在已经不推荐了。**新的代码不要这么做**。只需将特定于应用程序的子包导入与其他子包导入一样对待即可。

每种分组中，应该根据每个模块的完整包路径（`from path import ...` 中的 `path`）按字典序排序，忽略大小写。代码可以选择在导入节之间放置一个空行。

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

## 3.14 语句

通常每个语句应该独占一行。

不过，如果测试结果与测试语句在一行放得下，你也可以将它们放在同一行。如果是 `if` 语句，只有在没有 `else` 时才能这样做。特别地，绝不要对 `try/except` 这样做，因为 `try` 和 `except` 不能放在同一行。

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

## 3.15 访问控制

在 Python 中， 对于琐碎又不太重要的访问函数，你应该直接使用公有变量来取代它们，这样可以避免额外的函数调用开销。当添加更多功能时，你可以用属性（`property`）来保持语法的一致性。

另一方面，如果访问更复杂，或者变量的访问开销很显著，那么你应该使用像 `get_foo()` 和 `set_foo()` 这样的函数调用（遵循[命名](#316)准则）。如果之前的代码行为允许通过属性（`property`）访问，那么就不要将新的访问函数与属性绑定。这样，任何试图通过老方法访问变量的代码就没法运行，使用者也就会意识到复杂性发生了变化。

## 3.16 命名

`module_name`、`package_name`、`ClassName`、`method_name`、`ExceptionName`、`function_name`、`GLOBAL_CONSTANT_NAME`、`global_var_name`、`instance_var_name`、`function_parameter_name`、
`local_var_name`。

函数名、变量名和文件名应该都是描述性的，避免使用缩写。特别是，不要使用对项目以外的读者来说模棱两可或不熟悉的缩写，也不要通过删除单词中的字母来缩写。

总是使用 `.py` 文件扩展名，不要使用连字符。

### 3.16.1 应该避免的名称

- 单字符名称，除了以下特殊情况：

    - 计数器和迭代器（例如： `i` ， `j` ， `k` ， `v` 等等）
    - 作为 `try/except` 语句的异常标识符 `e` 。
    - 作为 `with` 语句声明的文件对象 `f`

    注意不要滥用单字符命名。一般来说，描述性应与名称的可见性范围成比例。例如： `i` 可能是五行代码块的好名称，但在多个嵌套范围内，它可能太模糊了。

- 包/模块名中的连字符（`-`）
- `__double_leading_and_trailing_underscore__` 双下划线开头并结尾的名称（Python保留）
- 不礼貌的用语
- 不需要包含变量类型的名称（例如：`id_to_name_dict`）

### 3.16.2 命名约定

- 所谓“内部（`Internal`）”表示仅模块内可用，或者在类内是保护或私有的。
- 用单下划线（`_`）开头表示模块变量或函数是 `protected` 的（使用 `import * from` 时不会包含）。用双下划线（`__`）开头的实例变量或方法表示类内私有，但并不推荐这么做，因为会影响代码的可读性或可测试性，而且也不是真正的私有。
- 将相关的类和顶级函数放在同一个模块里。不像 Java ，没必要限制一个类一个模块。
- 对类名使用大写字母开头的单词（如 `CapWords`，即 Pascal 风格），但是模块名应该用小写加下划线的方式（如 `lower_with_under.py`）。
  尽管已经有很多现存的模块使用类似于 `CapWords.py` 这样的命名，但现在已经不鼓励这样做，因为如果模块名碰巧和类名一致，这会让人困扰。（“想想 - 我应该用 `import StringIO` 还是 `from StringIO import StringIO` ？”）
- 下划线可能会出现在以 `test` 开头的 `unittest` 方法名称中，常用于分隔逻辑组件的名称，即使组件名称使用 `CapWords` 也应如此。一种可用的模式是： `test<MethodUnderTest>_<state>`，例如　`testPop_EmptyStack`。其实对于测试方法的命名并没有强制规定。

### 3.16.3 文件命名

Python 文件名必须以 `.py` 扩展名结尾，并且不要包含连字符（`-`）。这样可以方便导入和单元测试。如果你希望使用没有扩展名的可执行文件，可以使用软连接方式或者包含 `exec "$0.py" "$@"` 的简单包装脚本。

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

### 3.16.5 数学符号

对于偏数学运算的代码，当它们匹配参考论文或算法中已建立的符号时，较短的变量名会违反样式指南。执行此操作时，请在注释或文档字符串中引用所有命名约定的来源，如果来源无法访问，请清楚地记录命名约定。对于公共 API，最好使用符合 PEP8 的描述性名称（`descriptive_names`），这样更容易脱离上下文。

## 3.17 Main

在 Python 中， `pydoc` 以及单元测试要求模块必须是可导入的。如果文件打算作为可执行文件使用，那么它的主要功能应该放在 `main()` 函数中。你的代码应该在执行主程序前总是检查 `if __name__ == '__main__'`，这样当模块被导入时主程序就不会被执行。

当使用 [`absl`](https://github.com/abseil/abseil-py) 时，请使用 `app.run` ：

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

所有的顶级代码在模块导入时都会被执行。要小心不要去调用函数、创建对象或者执行那些不应该在使用 `pydoc` 时执行的操作。

## 3.18 函数长度

喜欢小而美的函数。

长函数有时候是可以接受的，对函数的长度没有硬性限制。如果一个函数超过了40行，就需要思考一下，在不破坏程序结构的情况下是否需要拆分。

即使你的长函数现在运行良好，将来修改它的人也可能会添加新的功能，这可能会导致 BUG 很难查找。保持函数的简短和简单可以使其他人更容易阅读和修改你的代码。

在处理某些代码时，您可能会发现长并且复杂的函数。先不要被修改这些代码所吓倒：如果感到函数使用困难，错误也很难调试，或者想在几个不同的地方使用相同的功能，可以考虑将函数拆分成更小和更易于管理的代码段。

## 3.19 类型标注

### 3.19.1 通用规则

- 熟悉 [PEP-484](https://www.python.org/dev/peps/pep-0484/)
- 在方法中，只有在需要正确的类型信息时才标注 `self` 或 `cls` 。例如：

    ```python
    @classmethod 
    def create(cls: Type[T]) -> T: 
        return cls()
    ```

- 如果无法表示任何其他变量或返回类型，请使用 `Any` 。
- 你不需要标注模块中的所有函数。
    - 至少要标注公共 API。
    - 在安全性和清晰性与灵活性之间找到一个平衡点。
    - 标注那些容易出现类型相关错误（以前的 BUG 或复杂性）的代码
    - 标注那些难以理解的代码。
    - 标注那些从类型的角度来看已经稳定的代码。多数情况下，你可以在稳定的代码中标注所有函数，而不会失去太多灵活性。

### 3.19.2 断行

遵循现有[缩进规则](#34)。

在标注之后，许多函数签名将变成“每行一个参数”。

```python
def my_method(self,
              first_var: int,
              second_var: Foo,
              third_var: Optional[Bar]) -> int:
    ...
```

尽量在变量之间断行，不要在变量名和类型标注之间断行。如果所有内容都在一行上，就不要管了。

```python
def my_method(self, first_var: int) -> int:
    ...
```

如果函数名、最后一个参数和返回类型组合起来太长了，可以新换一行并缩进4个字符。

```python
def my_method(
    self, first_var: int) -> Tuple[MyLongType1, MyLongType1]:
    ...
```

当返回类型与最后一个参数不在同一行时，推荐的做法是在新行中将参数缩进4个字符，并将右括号与 `def` 对齐。

!!! success "推荐"

    ```python
    def my_method(
        self, other_arg: Optional[MyLongType]
    ) -> Dict[OtherLongType, MyLongType]:
    ...
    ```

Pylint 允许您将右括号移到新行，并与左括号对齐，但这么做可读性会比较差。

!!! fail "不推荐"

    ```python
    def my_method(self,
                other_arg: Optional[MyLongType]
                ) -> Dict[OtherLongType, MyLongType]:
    ...
    ```

就像上面的例子一样，我们不希望截断类型。但是，有时候它们放在一行上实在太长了（尽量保持子类型不被截断）：

```python
def my_method(
    self,
    first_var: Tuple[List[MyLongType1],
                     List[MyLongType2]],
    second_var: List[Dict[
        MyLongType3, MyLongType4]]) -> None:
    ...
```

如果单个名称和类型太长，请考虑使用类型的别名。最后一种方法是在冒号后面截断，并缩进4个字符。

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

### 3.19.3 前置声明

如果你需要在同一模块中使用没有定义的类名（例如，需要在声明类中使用类名，或者类似下面的定义），可以使用字符串作为类名。

```python
class MyClass:

  def __init__(self,
               stack: List["MyClass"]) -> None:
```

### 3.19.4 默认值

根据 [PEP-008](https://www.python.org/dev/peps/pep-0008/#other-recommendations) ，仅在同时具有类型标注和默认值参数的 `=` 左右两边使用空格。

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

在 Python 类型系统中， `NoneType` 是“第一类（first class）”类型，而且为了方便拼写，定义了 `None` 作为 `NoneType` 的别名。如果一个参数可以为 `None` ，就必须声明它！你也可以使用 `Union` ，但如果只有一种类型，请使用 `Optional` 。

使用显式 `Optional` 替代隐式 `Optional` 。PEP-484 的早期版本允许将 `a: str = None` 解释为 `a: Optional[str] = None` ，但现在已经不推荐了。

!!! success "推荐"

    ```python
    def func(a: Optional[str], b: Optional[str] = None) -> str:
        ...
    def multiple_nullable_union(a: Union[None, str, int]) -> str
        ...
    ```

!!! fail "不推荐"

    ```python
    def nullable_union(a: Union[None, str]) -> str:
        ...
    def implicit_optional(a: str = None) -> str:
        ...
    ```

### 3.19.6 类型别名

可以为复杂类型声明别名。别名应该是大写的（`CapWorded`）。如果别名仅在模块中使用，那么应该使用前置下划线让其变成私有的（如 `_Private`）。

例如，如果模块名称和类型在一起太长了：

```python
_ShortName = module_with_long_name.TypeWithLongName
ComplexMap = Mapping[Text, List[Tuple[int, int]]]
```

其他例子还有复杂的嵌套类型和函数的多个返回变量（作为元组）。

### 3.19.7 忽略类型

可以在行上使用特殊注释 `# type: ignore` 禁用类型检查。

`pytype` 有一个针对特定错误的禁用选项（类似于 lint）

```python
# pytype: disable=attribute-error
```

### 3.19.8 标注变量

如果内部变量的类型很难推断或者无法推断时，可以通过以下几种方式指定其类型。

*[类型注释](https://google.github.io/styleguide/pyguide.html#type-comments)*

在行末尾使用 `# type:` 类型注释。

```python
a = SomeUndecoratedFunction()  # type: Foo
```

*[赋值标注](https://google.github.io/styleguide/pyguide.html#annotated-assignments)*

在变量名和值之间使用冒号和类型，就像函数参数一样：

```python
a: Foo = SomeUndecoratedFunction()
```

### 3.19.9 元组 vs 列表

类型化列表只能包含单一类型的对象。类型化元组可以具有单个重复类型，也可以具有一组不同类型的元素。后者通常用作函数的返回类型。

```python
a = [1, 2, 3]  # type: List[int]
b = (1, 2, 3)  # type: Tuple[int, ...]
c = (1, "2", 3.5)  # type: Tuple[int, str, float]
```

### 3.19.10 TypeVars

Python 类型系统中有[泛型](https://www.python.org/dev/peps/pep-0484/#generics)，工厂函数 `TypeVar` 是使用它们的常用方法。

例如：

```python
from typing import List, TypeVar
T = TypeVar("T")
...
def next(l: List[T]) -> T:
    return l.pop()
```

TypeVar 可以被约束：

```python
AddableType = TypeVar("AddableType", int, float, str)
def add(a: AddableType, b: AddableType) -> AddableType:
    return a + b
```

`typing` 模块中一个常见的预定义类型变量是 `AnyStr` 。可以用于标注 `bytes` 或 `unicode` ，但是必须是在相同类型中使用。

```python
from typing import AnyStr
def check_length(x: AnyStr) -> AnyStr:
    if len(x) <= 42:
        return x
    raise ValueError()
```

### 3.19.11 字符串类型

想要正确的标注字符串类型，取决于代码将使用哪个版本的 Python 。

- 对于 Python 3 代码，最好使用 `str` 。 `Text` 也是可以接受的。使用时请保持一致性。
- 对于要兼容 Python 2 的代码，可以使用 `Text` 。在极少数情况下，使用 `str` 可能是有意义的；通常，当两个 Python 版本之间的返回类型不同时，可以提高兼容性。
- 避免使用 `unicode` ，因为它在 Python 3 中不存在。

之所以存在这种差异，是因为 `str` 的含义和 Python 版本有关。

!!! fail "不推荐"

    ```python
    def py2_code(x: str) -> unicode:
        ...
    ```

对于处理二进制数据的代码，请使用 `bytes` ：

```python
def deals_with_binary_data(x: bytes) -> bytes:
    ...
```

对于处理文本数据的 python 2 兼容代码（Python 2 中的 `str` 或 `unicode`，Python 3 中的 `str`），请使用 `Text` 。对于仅处理文本数据的 python 3 代码，首选 `str` 。

```python
from typing import Text
...
def py2_compatible(x: Text) -> Text:
    ...
def py3_only(x: str) -> str:
    ...
```

如果类型可以是 `bytes` 或 `text`，请使用 `Unicode` 和适当的文本类型。

```python
from typing import Text, Union
...
def py2_compatible(x: Union[bytes, Text]) -> Union[bytes, Text]:
    ...
def py3_only(x: Union[bytes, str]) -> Union[bytes, str]:
    ...
```

如果函数的所有字符串类型始终相同，例如，如果返回类型与上述代码中的参数类型相同，则使用 AnyStr。

这样编写可以简化将代码移植到 python 3 的过程。

### 3.19.12 类型导入

对于 `typing` 模块中的类，请始终导入类本身。明确允许在一行上从 `typing` 模块导入多个特定的类。例如：

```python
from typing import Any, Dict, Optional
```

既然这种从 `typing` 模块导入的方式会将导入项添加到本地命名空间， 那么 `typing` 中的任何名称都应该类似于关键字，而且不要在你的 Python 代码中去定义（无论是否有类型）。如果模块中的类型和现有名称之间存在冲突，请使用 `import x as y` 导入。

```python
from typing import Any as AnyType
```

### 3.19.13 条件导入

仅在特殊情况下才使用条件导入，在这种情况下，必须在运行时避免类型检查所需的其他导入。不推荐这种方式；应该首选其他方法，比如重构代码以允许顶级导入。

可以将仅用于类型标注的导入放在 `if TYPE_CHECKING:` 代码块中。

- 有条件导入的类型需要作为字符串引用，以便标注表达式实际运行时能向前兼容 Python 3.6。
- 这里只应该定义用于类型标注的实体；包括别名。否则将会有一个运行时错误，因为模块将不会在运行时导入。
- 所有正常导入后的代码块应该是正确的。
- 类型导入列表中不应该有空行。
- 将此列表按照常规导入列表进行排序。

```python
import typing
if typing.TYPE_CHECKING:
    import sketch
def f(x: "sketch.Sketch"): ...
```

### 3.19.14 循环依赖

由类型引起的循环依赖是一种代码味道。这些代码需要进行重构。虽然在技术上可以保持循环依赖关系，但是各种构建系统不允许这样做，因为每个模块都必须依赖于其他模块。

将引起循环依赖导入的模块替换为 `Any`  。设置一个有意义的[别名](#3106) ，并使用此模块中的实际类型名称（Any 的任何属性都是 Any）。别名定义应该与最后导入分开一行。

```python
from typing import Any

some_mod = Any  # some_mod.py imports this module.
...

def my_method(self, var: "some_mod.SomeType") -> None:
    ...
```

### 3.19.15 泛型

进行标注时，最好为泛型类型指定类型参数；否则，[泛型参数将被假定为 `Any`](https://www.python.org/dev/peps/pep-0484/#the-any-type) 。

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

如果泛型的最佳类型参数是 `Any`，请使用显式设置。但请记住，在许多情况下 [`TypeVar`](#31910-typevars) 可能更合适。

```python
def get_names(employee_ids: List[Any]) -> Dict[Any, Text]:
  """Returns a mapping from employee ID to employee name for given IDs."""
```

```python
T = TypeVar('T')
def get_names(employee_ids: List[T]) -> Dict[T, Text]:
  """Returns a mapping from employee ID to employee name for given IDs."""
```
