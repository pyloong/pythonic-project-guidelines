# 插件

[plug-in](https://en.wikipedia.org/wiki/Plug-in_(computing)) 在维基百科中是这么定义的：“在计算中，插件是软件组件，为现有计算机程序增加一个特定的特征。”
所以插件应该是一个能够灵活配置，并很方便的载入配置中的内容。

由于 Python 本身的动态特性，插件化的实现就更灵活。现有的动态插件都是基于 Python 的命名空间和动态导入功能来查找并导入外部依赖。
具体原理可以查看 [Creating and discovering plugins](https://docs.pytest.org/en/latest/index.html) 。

## 插件框架

### pluggy

[pluggy](https://pluggy.readthedocs.io/en/latest/) 是从 [pytest](https://docs.pytest.org/en/latest/index.html) 中演化出来的一个插件工具。它为 pytest
提供外围插件支持，当开发人员需要扩展 pytest 的功能时，基于 pytest 的规范做出对应的插件然后将其安装到环境中后， pytest 就可以自动识别已有插件。

其具体原理是通过创建一个 `hookspec = pluggy.HookspecMarker("eggsample")` 来标记插件事先的规范，然后使用 `hookimpl = pluggy.HookimplMarker("eggsample")` 标记
插件的实现。

规范：

```python
import pluggy

hookspec = pluggy.HookspecMarker("eggsample")


@hookspec
def eggsample_add_ingredients(ingredients: tuple):
    """Have a look at the ingredients and offer your own.

    :param ingredients: the ingredients, don't touch them!
    :return: a list of ingredients
    """


@hookspec
def eggsample_prep_condiments(condiments: dict):
    """Reorganize the condiments tray to your heart's content.

    :param condiments: some sauces and stuff
    :return: a witty comment about your activity
```

实现：

```python
import pluggy


hookimpl = pluggy.HookimplMarker("eggsample")
"""Marker to be imported and used in plugins (and for own implementations)"""


class ExamplePluggy:

    @hookimpl
    def eggsample_add_ingredients(self):
        spices = ["salt", "pepper"]
        you_can_never_have_enough_eggs = ["egg", "egg"]
        ingredients = spices + you_can_never_have_enough_eggs
        return ingredients

    @hookimpl
    def eggsample_prep_condiments(self, condiments):
        condiments["mint sauce"] = 1
```

然后将插件规范和实现装载到插件管理类中，为了可以找到其他人开发的插件，需要调用 `load_setuptools_entrypoints` 方法从命名空间
查找已经在指定命名空间下的其他插件。

```python
import itertools
import random

import pluggy


def get_plugin_manager():
    pm = pluggy.PluginManager("eggsample")
    pm.add_hookspecs(hookspecs)
    pm.load_setuptools_entrypoints("eggsample")
    pm.register(ExamplePluggy)
    return pm
```

在使用时，调用 `pm.hook.eggsample_add_ingredients` 传递参数即可。

外部开发的插件，只需要遵循插件规范做实现：

```python
import eggsample


@eggsample.hookimpl
def eggsample_add_ingredients(ingredients):
    """Here the caller expects us to return a list."""
    if "egg" in ingredients:
        spam = ["lovely spam", "wonderous spam"]
    else:
        spam = ["splendiferous spam", "magnificent spam"]
    return spam


@eggsample.hookimpl
def eggsample_prep_condiments(condiments):
    """Here the caller passes a mutable object, so we mess with it directly."""
    try:
        del condiments["steak sauce"]
    except KeyError:
        pass
    condiments["spam sauce"] = 42
    return "Now this is what I call a condiments tray!"
```

并在打包信息中标注相同的命名空间：

```python
from setuptools import setup

setup(
    name="eggsample-spam",
    install_requires="eggsample",
    entry_points={"eggsample": ["spam = eggsample_spam"]},
    py_modules=["eggsample_spam"],
)
```

其原理也是通过 Python 的 `from importlib.metadata import entry_points` 找到注册到 Python 解释器 `entry_points` 中的包，并
根据命名空间获取需要的内容。

### stevedore

[stevedore](https://docs.openstack.org/stevedore/latest/) 是 [Openstack](https://www.openstack.org/) 开发和维护的一个插件工具。该
插件为 [Openstack](https://www.openstack.org/) 的 [ceilometer](https://docs.openstack.org/ceilometer/latest/) 提供插件功能。

[stevedore](https://docs.openstack.org/stevedore/latest/) 则是推荐使用继承的方式规范插件接口。

首先创建一个插件基类：

```python
# stevedore/example/base.py
# -*- coding: utf-8 -*-
# Copyright (C) 2020 Red Hat, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
# implied.
# See the License for the specific language governing permissions and
# limitations under the License.
import abc


class FormatterBase(metaclass=abc.ABCMeta):
    """Base class for example plugin used in the tutorial.
    """

    def __init__(self, max_width=60):
        self.max_width = max_width

    @abc.abstractmethod
    def format(self, data):
        """Format the data and return unicode text.

        :param data: A dictionary with string keys and simple types as
                     values.
        :type data: dict(str:?)
        :returns: Iterable producing the formatted text.
        """
```

然后实现一个简单的插件：

```python
# stevedore/example/simple.py
# Copyright (C) 2020 Red Hat, Inc.
#
#  Licensed under the Apache License, Version 2.0 (the "License"); you may
#  not use this file except in compliance with the License. You may obtain
#  a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
#  WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
#  License for the specific language governing permissions and limitations
#  under the License.
from stevedore.example import base


class Simple(base.FormatterBase):
    """A very basic formatter."""

    def format(self, data):
        """Format the data and return unicode text.

        :param data: A dictionary with string keys and simple types as
                     values.
        :type data: dict(str:?)
        """
        for name, value in sorted(data.items()):
            line = '{name} = {value}\n'.format(
                name=name,
                value=value,
            )
            yield line
```

最后打包。打包的时候，将插件注册到 `entry_points` 中。

```python
# stevedore/example/setup.py
# Copyright (C) 2020 Red Hat, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
# implied.
# See the License for the specific language governing permissions and
# limitations under the License.
from setuptools import find_packages
from setuptools import setup

setup(
    name='stevedore-examples',
    version='1.0',

    description='Demonstration package for stevedore',

    author='Doug Hellmann',
    author_email='doug@doughellmann.com',

    url='http://opendev.org/openstack/stevedore',

    classifiers=['Development Status :: 3 - Alpha',
                 'License :: OSI Approved :: Apache Software License',
                 'Programming Language :: Python',
                 'Programming Language :: Python :: 2',
                 'Programming Language :: Python :: 2.7',
                 'Programming Language :: Python :: 3',
                 'Programming Language :: Python :: 3.5',
                 'Intended Audience :: Developers',
                 'Environment :: Console',
                 ],

    platforms=['Any'],

    scripts=[],

    provides=['stevedore.examples',
              ],

    packages=find_packages(),
    include_package_data=True,

    entry_points={
        'stevedore.example.formatter': [
            'simple = stevedore.example.simple:Simple',
            'plain = stevedore.example.simple:Simple',
        ],
    },

    zip_safe=False,
)
```

创建一个插件项目，并实现插件：

```python
# stevedore/example2/fields.py
# -*- coding: utf-8 -*-
# Copyright (C) 2020 Red Hat, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
# implied.
# See the License for the specific language governing permissions and
# limitations under the License.
import textwrap

from stevedore.example import base


class FieldList(base.FormatterBase):
    """Format values as a reStructuredText field list.

    For example::

      : name1 : value
      : name2 : value
      : name3 : a long value
          will be wrapped with
          a hanging indent
    """

    def format(self, data):
        """Format the data and return unicode text.

        :param data: A dictionary with string keys and simple types as
                     values.
        :type data: dict(str:?)
        """
        for name, value in sorted(data.items()):
            full_text = ': {name} : {value}'.format(
                name=name,
                value=value,
            )
            wrapped_text = textwrap.fill(
                full_text,
                initial_indent='',
                subsequent_indent='    ',
                width=self.max_width,
            )
            yield wrapped_text + '\n'
```

同样打包，并配置注册信息，将插件注册到同一个命名空间中：

```python
# stevedore/example2/setup.py
# Copyright (C) 2020 Red Hat, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
# implied.
# See the License for the specific language governing permissions and
# limitations under the License.
from setuptools import find_packages
from setuptools import setup

setup(
    name='stevedore-examples2',
    version='1.0',

    description='Demonstration package for stevedore',

    author='Doug Hellmann',
    author_email='doug@doughellmann.com',

    url='http://opendev.org/openstack/stevedore',

    classifiers=['Development Status :: 3 - Alpha',
                 'License :: OSI Approved :: Apache Software License',
                 'Programming Language :: Python',
                 'Programming Language :: Python :: 2',
                 'Programming Language :: Python :: 2.7',
                 'Programming Language :: Python :: 3',
                 'Programming Language :: Python :: 3.5',
                 'Intended Audience :: Developers',
                 'Environment :: Console',
                 ],

    platforms=['Any'],

    scripts=[],

    provides=['stevedore.examples2',
              ],

    packages=find_packages(),
    include_package_data=True,

    entry_points={
        'stevedore.example.formatter': [
            'field = stevedore.example2.fields:FieldList',
        ],
    },

    zip_safe=False,
)
```

在使用是，可以通过 driver 方式调用：

```python
# stevedore/example/load_as_driver.py
# Copyright (C) 2020 Red Hat, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
# implied.
# See the License for the specific language governing permissions and
# limitations under the License.
import argparse

from stevedore import driver


if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument(
        'format',
        nargs='?',
        default='simple',
        help='the output format',
    )
    parser.add_argument(
        '--width',
        default=60,
        type=int,
        help='maximum output width for text',
    )
    parsed_args = parser.parse_args()

    data = {
        'a': 'A',
        'b': 'B',
        'long': 'word ' * 80,
    }

    mgr = driver.DriverManager(
        namespace='stevedore.example.formatter',
        name=parsed_args.format,
        invoke_on_load=True,
        invoke_args=(parsed_args.width,),
    )
    for chunk in mgr.driver.format(data):
        print(chunk, end='')
```

或者通过 extensions 的方式调用：

```python
# stevedore/example/load_as_extension.py
# Copyright (C) 2020 Red Hat, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
# implied.
# See the License for the specific language governing permissions and
# limitations under the License.
import argparse

from stevedore import extension


if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument(
        '--width',
        default=60,
        type=int,
        help='maximum output width for text',
    )
    parsed_args = parser.parse_args()

    data = {
        'a': 'A',
        'b': 'B',
        'long': 'word ' * 80,
    }

    mgr = extension.ExtensionManager(
        namespace='stevedore.example.formatter',
        invoke_on_load=True,
        invoke_args=(parsed_args.width,),
    )

    def format_data(ext, data):
        return (ext.name, ext.obj.format(data))

    results = mgr.map(format_data, data)

    for name, result in results:
        print('Formatter: {0}'.format(name))
        for chunk in result:
            print(chunk, end='')
        print('')
```

## 实践

通过实际开发体验，推荐使用 [stevedore](https://docs.openstack.org/stevedore/latest/) 。总结优点如下：

- 通过设计基类定义接口规范
- 插件调用更加灵活。
- 使用复杂度上 [stevedore](https://docs.openstack.org/stevedore/latest/) 更胜一筹
