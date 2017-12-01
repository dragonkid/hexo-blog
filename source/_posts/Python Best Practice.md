---
title: Python Best Practice
---

# Python 编码规范

* [Python Coding Rule](https://wiki.woodpecker.org.cn/moin/PythonCodingRule)
* [PEP8](https://www.python.org/dev/peps/pep-0008/)
* 遵从项目原有代码风格
* 代码整洁优先级最高
* 尽量使用 `key in dict` 或者 `dict.get(key)` 判断 dict 中是否存在 `key` 这个键或获取键 `key` 的 `value`
* `itertools` 包可以应付绝大部分对 list 类型数据的操作场景，遇到特殊的 list 类型数据操作需求时优先考虑使用 `itertools` 包，而非自行实现
* 服务端一律使用 `time.time()` 获取 UTC 时间戳作为时间标识，在需要进行展示的地方使用对应方法转换为本时区的不同格式时间
* 通过截断逻辑避免缩进的出现。以下述代码为例：

```
def save_data_to_mongo(data):
    if data:
        check_data(data)
        format_data(data)
        extend_data(data)
        db.collection.insert(data)
```

更好的写法为：

```
def save_data_to_mongo(data):
    if not data:
        return
    check_data(data)
    format_data(data)
    extend_data(data)
    db.collection.insert(data)
```

* 函数拆分、接口设计原则优先考虑可读性，在出现性能问题时再考虑合并优化
* 定义数值型常量时可以直接写清楚计算过程，解释器会帮忙将常量优化为计算结果

```python
SECOND_ONE_WEEK = 604800

SECOND_ONE_WEEK = 7 * 24 * 3600    # better
```

* `try ... except Exception: ...` 本质上是异常处理，而非仅仅是异常捕获。稳定代码和非稳定代码要做出区分，应严格避免对大段代码的异常捕获，对异常的处理也应该尽可能细致，尽量避免不负责任的匿名 catch 的出现
* Dict upsert

常规实现：

```
dict_uo2data = {}
if ip in dict_uo2data:
    dict_uo2data[ip].append(data)
else:
    dict_uo2data[ip] = [data]
```

更好的写法：

```
dict_uo2data = {}
dict_uo2data.setdefault(ip, []).append(data)
```

* 禁止使用 mutable 对象作为默认参数

错误示例一：

``` python
>>> def f(lst = []):
...     lst.append(1)
...     return lst
...
>>> f()
[1]
>>> f()
[1, 1]
```

错误示例二：

``` python
>>> import time
>>> def report(when=time.time()):
... return when
...
>>> report()
1500113234.487932
>>> report()
1500113234.487932
```

* 正确理解 `=` 和 `+=` 的区别

``` python
>>> x = [1]
>>> print id(x)
4357132800
>>> x = x + [2]	# 指向一个新的对象
>>> print id(x) 
4357132728

>>> x = [1]
>>> print id(x)
4357132800
>>> x += [2]		# 修改原有对象
>>> print id(x)
4357132800
```

* 禁止使用乘法运算生成可变对象序列

```python
In [5]: test = [[]] * 10

In [6]: test
Out[6]: [[], [], [], [], [], [], [], [], [], []]

In [7]: test[0].append(10)

In [8]: test
Out[8]: [[10], [10], [10], [10], [10], [10], [10], [10], [10], [10]]
```

```
In [9]: for i in test:
   ...:     print id(i)
4548762800
4548762800
4548762800
4548762800
4548762800
4548762800
4548762800
4548762800
4548762800
4548762800
```

* 禁止在访问列表的同时对列表中元素进行增删操作

```
In [11]: l = [1, 2, 4, 5]

In [12]: for i, v in enumerate(l):
    ...:     if v % 2 == 0:
    ...:         del l[i]
    ...:

In [13]: l
Out[13]: [1, 4, 5]	# 预期为 [1, 5]
```

* 禁止在 list comprehension 中使用 lambda 表达式

``` python
In [19]: for func in  [lambda x: i * x for i in range(5)]:
    ...:     print func(2)
8
8
8
8
8
```

问题产生根本原因在于 python 的属性查找规则。上述示例中 i 在闭包作用域，而 python 的闭包是 lazy binding，这意味着闭包中的值，是在内部函数被调用时查询到的。

在无法避免使用闭包时，可以通过传递默认参数将 i 置于本地作用域。

```
In [19]: for func in  [lambda x, i=i: i * x for i in range(5)]:
    ...:     print func(2)
```

* 禁止自行定义 `__del__` 函数，可能导致内存泄露。

> Circular references which are garbage are detected when the option cycle detector is enabled (it’s on by default), but can only be cleaned up if there are no Python-level __del__() methods involved.

* 迭代器只能被遍历一次

``` python
In [29]: test = iter([1, 2, 3, 4, 5])

In [30]: for i in range(2):
    ...:     for j in test:
    ...:         print i, j
0 1
0 2
0 3
0 4
0 5

In [31]: test.next()
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-31-0743161fe2fe> in <module>()
----> 1 test.next()

StopIteration:
```

* 尽量使用 `get` 方法获取 `dict` 中对应 key 的 value

```python
test = {
	'list': [1, 2, 3, 4],
	'dict': {
		'key1': 1,
		'key2': 2
	},
	'int': 100,
	'string': 'test',
	'bool': True
}

test.get('list', [])
test.get('dict', {})
test.get('int', 0)
test.get('string', '')
test.get('bool', False)
```

* 函数返回值应为同一类型

错误示例：

```python
In [54]: def func(arg):
    ...:     if arg:
    ...:         return [1, 2, 3, 4]
    ...:     else:
    ...:         return
    ...:
    ...: for i in func(False):
    ...:     print i
    ...:
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-54-eb27d4da633f> in <module>()
      5         return
      6
----> 7 for i in func(False):
      8     print i
      9

TypeError: 'NoneType' object is not iterable
```

正确示例：

```python
In [55]: def func(arg):
    ...:     if arg:
    ...:         return [1, 2, 3, 4]
    ...:     else:
    ...:         return []
    ...:
    ...: for i in func(False):
    ...:     print i
    ...:
```

# Tips

## Alfred

* file saerch
* web search
* clipboard
* snippets
* workflow

## Dash

## autojump

## ssh config 

> The known_hosts file lets the client authenticate the server, to check that it isn't connecting to an impersonator. The authorized_keys file lets the server authenticate the user.

## tmux

* https://github.com/dragonkid/tmux-config
* zoom pane `ctrl + z`

## keychain

```shell
eval $(keychain -Q -q --agents ssh --eval ~/.ssh/id_rsa)
```

## proxychains

## mitmproxy

```shell
mitmproxy -p 9200 -R http://localhost:9201 -b 0.0.0.0 --no-mouse
```

## virtualenvwrapper

## vagrant

* vagrant scp
* vagrant oh-my-zsh plugin

## shell/zsh

* `alt + f/b`: 前进/后退一个单词
* `ctrl + f/b`: 前进/后退一个字母
* `ctrl + u/y`: 清空/恢复当前输入
* `ctrl + e`: 移动到当前输入末尾
* `ctrl + a`: 移动到当前输入开头
* `ctrl + '/"`: 为当前文本增加单/双引号 
* `ctrl + p/n`: 查看上一条/下一条命令
* `ctrl + r`/`ctrl + shift + r`: 查找曾经输入过的命令，按多次查找下一个或上一个
* `ctrl+xx`: Move between the beginning of the line and the current position of the cursor. This allows you to press `ctrl+xx` to return to the start of the line, change something, and then press `ctrl+xx` to go back to your original cursor position. To use this shortcut, hold the `ctrl` key and tap the `x` key twice.
* `ctrl+d or Delete`: Delete the character under the cursor.
* `alt+d`: Delete all characters after the cursor on the current line.
* `alt + h`: 命令输入到一半的时候需要查看 man 手册
* `sudo` 插件: 按两次 `esc` 在命令的开头补齐 sudo
* `colored-man-pages` 插件: man 手册高亮

## Balsamiq

研发快速制作原型图

## Pycharm

* ideavim（~/.ideavimrc 配置方式与 vim 相同）






