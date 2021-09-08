---
title: BeautifulSoup笔记
date: 2021-09-03 07:35:02
categories:
  - - Python
    - 爬虫
tags:
  - 爬虫
  - bs4
mathjax: true
headimg:
  'https://z3.ax1x.com/2021/08/16/fWOlXF.jpg'
---

BeautifulSoup4 学习笔记

<!-- more -->

## 简介

Beautiful Soup是一款可以从HTML以及XML文件中提取数据的python库。

接下来的实验我们都将以一篇经典的案例为基础：

```html
# <html>
#  <head>
#   <title>
#    The Dormouse's story
#   </title>
#  </head>
#  <body>
#   <p class="title">
#    <b>
#     The Dormouse's story
#    </b>
#   </p>
#   <p class="story">
#    Once upon a time there were three little sisters; and their names were
#    <a class="sister" href="http://example.com/elsie" id="link1">
#     Elsie
#    </a>
#    ,
#    <a class="sister" href="http://example.com/lacie" id="link2">
#     Lacie
#    </a>
#    and
#    <a class="sister" href="http://example.com/tillie" id="link2">
#     Tillie
#    </a>
#    ; and they lived at the bottom of a well.
#   </p>
#   <p class="story">
#    ...
#   </p>
#  </body>
# </html>
```



## 开始

想要使用Beautiful Soup解析一段HTML文档，只需要用如下一段代码，得到一个BeautifulSoup对象，即可对改文档进行方便的操作：

```python
frome bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'html.parser')
```

BeautifulSoup对象提供了一些简单的操作，能方便的提取出其中的一些内容：

```python
print(soup.prettify())  # 按照标准缩进格式化输出

print(soup.title)  # <title>The Dormouse's story</title> 该操纵会将第一个title标签下的所后代标签


print(soup.title.name)  # 'title'

print(soup.title.string)  # 'The Dormouse's story'

print(soup.title.parent.name)  # 'head'

print(soup.p)  # <p class="title"><b>The Dormouse's story</b></p>

print(soup.p['class'])  # title

print(soup.fund_all('a'))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

print(soup.get_text())  # 从文档中获取所有文字内容
# The Dormouse's story
#
# The Dormouse's story
#
# Once upon a time there were three little sisters; and their names were
# Elsie,
# Lacie and
# Tillie;
# and they lived at the bottom of a well.
#
# ...
```

## 解析器

Beautiful Soup支持Python标准库中的HTML解析器,还支持一些第三方的解析器,其中一个是 [lxml](http://lxml.de/) 。

另一个可供选择的解析器是纯Python实现的 [html5lib](http://code.google.com/p/html5lib/) ，html5lib的解析方式与浏览器相同。

下面是各种解析器的优缺点：

| 解析器           | 使用方法                                                     | 优势                                                  | 劣势                                            |
| ---------------- | ------------------------------------------------------------ | ----------------------------------------------------- | ----------------------------------------------- |
| Python标准库     | `BeautifulSoup(markup, "html.parser")`                       | Python的内置标准库执行速度适中文档容错能力强          | Python 2.7.3 or 3.2.2)前 的版本中文档容错能力差 |
| lxml HTML 解析器 | `BeautifulSoup(markup, "lxml")`                              | 速度快文档容错能力强                                  | 需要安装C语言库                                 |
| lxml XML 解析器  | `BeautifulSoup(markup, ["lxml-xml"])   BeautifulSoup(markup, "xml")` | 速度快唯一支持XML的解析器                             | 需要安装C语言库                                 |
| html5lib         | `BeautifulSoup(markup, "html5lib")`                          | 最好的容错性以浏览器的方式解析文档生成HTML5格式的文档 | 速度慢不依赖外部扩展                            |

## 对象的种类

Beautiful Soup将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象,所有对象可以归纳为4种: `Tag` , `NavigableString` , `BeautifulSoup` , `Comment` .

### Tag

tag对象与XML或HTML原生文档中的tag相同：

```python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b
type(tag)
# <class 'bs4.element.Tag'>
```

tag具有非常多的属性和方法，其中比较重要的是name和attributes

#### Name

即该tag对象的标签名：

```python
print(tag.name)  # b
```

如果改变了tag的name，将影响所有通过当前BeautifulSoup对象生成的Html文档。

### Attributes

在HTML中，标签往往包含许多属性，而使用BeautifulSoup可以很方便的操作这些属性。在BS4中操作实现和使用python的dict一样：

```python
tag['class']  # blodest
```

也可以直接使用`.`进行访问：

```python
tag.attrs  # {'class':'boldest'}
```

可见访问`tag`的`attrs`将返回一个`dict`

tag属性可以被添加，删除或是修改，和操作字典的方式相同:

```python
tag['class'] = 'verybold'
tag['id'] = 1
tag
# <blockquote class="verybold" id="1">Extremely bold</blockquote>

del tag['class']
del tag['id']
tag
# <blockquote>Extremely bold</blockquote>

tag['class']
# KeyError: 'class'
print(tag.get('class'))
# None
```

#### 多值属性

HTML中允许某一属性有很多值，最典型的就是`class`属性，还有一些比如`rel`，`rev`，`accept-charset`，`headers`，`accesskey`，在BS中，多值属性以`list`的形式返回：

```python
css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.p['class']
# ["body", "strikeout"]

css_soup = BeautifulSoup('<p class="body"></p>')
css_soup.p['class']
# ["body"]
```

但对于一些没有被HTML定义为多值的属性，比如`id`，有时它们的值却看起来像多值，比如一下例子：

```python
id_soup = BeautifulSoup('<p id = 'my id'></p>')
id_soup.p[id]  # 'my id'
```

可见这样的虽然看起来是多值的属性，只要它没有被HTML标准定义为多值属性，结果就会以字符串的形式输出。

如果将文档解析为`XML`格式，那么tag中将不包含多值属性，所有属性均以字符串返回：

```python
xml_soup = BeautifulSoup('<p class="body strikeout"></p>', 'xml')
xml_soup.p['class']
# u'body strikeout'
```

### NavigableString

当我们获取tag中包裹的文本时，BS会用一个`NavigableString`来包装该字符串：

```python
tag.string
# 'Extremely bold'
type(tag.string)
# <class 'bs4.element.NavigableString'>
```

`NavigableString`实际上是封装了一些特性的Unicode字符串，通过`str()`方法可以最直接将`navigableString`对象转化为Unicode String：

```python
unicode_strinng = unicode(tag.string)
print(unicode_string)  # Extremely bold
type(unicode_string)
# <type 'str'>
```

此外，tag中通包含的字符串不能编辑，但是可以进行替换，使用`replace_with()`方法：

```python
tag.string.replace_with("No longer bold")
print(tag)
# <blockquote>No longer bold</blockquote>
```

如果想在Beautiful Soup之外使用 `NavigableString` 对象,需要调用 `unicode()` 方法,将该对象转换成普通的Unicode字符串,否则就算Beautiful Soup已方法已经执行结束，我们的String对象也会包含一个指向整个BeautifulSoup 解析树的引用。这样会浪费内存.

### BeautifulSoup

BeautifulSoup对象代表了整个被解析过的文件，很多时候我们可以将其看作是一个Tag对象。

我们也能将BeautifulSoup对象传入一些修改解析树的函数，例如我们想合并两个结构文档：

```python
doc = BeautifulSoup("<head>INSERT FOOTER HERE</head>", "xml")
foot = BeautifulSoup("<footer>Here is the footer</footer>", "xml")
doc.head.string.replace_with(foot)
print(doc)

# <?xml version="1.0" encoding="utf-8"?>
# <head><footer>Here is the footer</footer></head>
```

而由于`BeautifukSoup`对象并没有真正的指向某个HTML或XML标签，因此，它并不包含明确的标签名和属性集合，但有时访问它的标签名又是必要的，因此该对象被给与了一个特殊的标签名`[document]`：

```python
soup.name
# [document]
soup.attrs
# {}
```

### comment 和特殊字串

使用`Tag`、`NavigableString`和`BeautifulSoup`可以涵盖大部分HTML或XML文件中出现的内容，但还有些并不那么经常出现的特殊内容，例如注释：

```python
markup = "<b><!-- Hey, buddy. Want to buy a used parser? --></b>"
soup = BeautifulSoup(markup, "lxml")
comment = soup.b.string
print(type(comment))
# <class 'bs4.element.Comment'>
```

事实上`Comment`对象就是一种特殊的`NavigableString`：

```python
print(isinstance(comment, NavigableString))
# True
```

输出格式化后的形式如下：

```python
print(soup.prettify())
# <html>
#  <body>
#   <b>
#    <!-- Hey, buddy. Want to buy a used parser? -->
#   </b>
#  </body>
# </html>
```

 BeautifulSoup中也定义了一些类：`Stylesheet`、`Script`、`TemplateString`，分别对应了HTML中的`<style>`标签中的内容，`<script>`标签中的内容以及`<template>`标签中的内容。这些类都是`NavigableString`的子类。做这样的区分是为了更好的找出页面的主要部分。这些类是在BeautifulSoup4.9.0版本新增，html5lib中并不包含这些类。

而在XML中，还有许多特殊标签，比如`CData`、`ProcessingInstruction`、`Declaration`、`Doctype`。这些类和`Comment`一样都是`Navigable String`的子类。

```python
from bs4.element import CData
cdata = CData("A CDATA block")
comment.replace_with(cdata)

print(soup.b.prettify())
# <html>
#  <body>
#   <b>
#    <![CDATA[A CDATA block <]]>
#   </b>
#  </body>
# </html>

```

## 遍历文档树

接下来我们会使用下面的例子进行举例：

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'html.parser')
```

### 向下遍历

BeautifulSoup为我们提供了很多用于遍历的属性，但`NavigableString`是无法使用这些属性的，因为它们不包含子节点。

#### 按标签名遍历

最简单的遍历方式就是通过标签名遍历，但这种遍历方式只会为你找出当前节点下的第一个同名子节点：

```python
soup.a
# <a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>
```

如果想要获得所有同名子标签，那么需要使用`find_all`函数查找所有标签。

```python
soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

#### contents和children

`contents`是`tag`对象所包含的一个`list`，该属性包含了该标签下的所有**直系**子标签

```python
soup_test = BeautifulSoup(html_doc_test, 'lxml')
tag = soup_test.body
print(tag.contents)
# ['\n', <p class="title"><b>The Dormouse's story</b></p>, '\n', <p class="story">Once upon a time there were three little sisters; and their names were
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
# <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
# and they lived at the bottom of a well.</p>, '\n', <p class="story">...</p>, '\n']

print(tag.contents[1].contents[0].contents)
# ["The Dormouse's story"]
```

BeautifulSoup对象本身也包含子节点，我们认为BeautifulSoup对象只包含一个`<html>`子标签：

```python
print(len(soup_test.contents))
# 1
print(soup_test.contents[0].name)
# html
```

需要注意的是`NavigableString`对象并不包含这两个属性。

`contents`是返回`list`对象，而`children`则是返回一个包含所有`直系`子节点的`generator`：

```python
for child in soup_test.body.children:
    print(child)
```

#### descendants

相比于`contents`和`children`返回直系子节点，`descendants`则是返回所有子节点。

例如，对于如下标签的子节点：

```pythob
headTag = soup_test.head
print(headTag.contents)
# [<title>The Dormouse's story</title>]
```

该节点的子标签只有`<title>`但是，从BeautifulSoup对象的角度来看，标签`<title>`还有一个子节点`NavigableString`，但我们是用`.children`进行遍历时，无法遍历到该`NavigableString`

此时我们尝试使用`decendants`进行遍历：

```py
for child in headTag.descendants:
    print(child)
# <title>The Dormouse's story</title>
# The Dormouse's story
```

对于`BeautifulSoup`对象而言，它的`children`只包含一个子节点，而`descendants`则包含整个html解析树中的所有结点：

```python
print(len(soup_test.contents)) # 1
print(len(list(soup_test.descendants))) # 26
```

#### string

当某一标签仅包含一个子节点，且该子节点为`NavigableString`对象时，可以调用该节点的`String`属性，如果该标签不仅包含一个`NavigableString`对象节点，则访问该对象的`String`属性时将会返回`None`

```python
print(soup_test.string)
print(headTag.title.string)
# None
# The Dormouse's story
```

#### string & stripped_strings

当某个标签之内不仅包含一个`NavigableString`对象时，可以使用`strings`遍历该标签下的所有`NavigableString`对象。该属性返回一个包含所有子`NavigableString`的`生成器`

```python
for string in soup_test.strings:
    print(string)
# The Dormouse's story
# '\n'
# Once upon a time there were three little sisters; and their names were
# '\n'
# Elsie
# ,
# '\n'
# Lacie
#  and
# '/n'
# Tillie
# ;
# and they lived at the bottom of a well.
# '/n'
# ...
# '/n'
```

可见使用该方法访问时，回车也会被遍历到，如果不想单行的回车也被视为`NavigableString`对象，则需要使用`stripped_strings`，该属性将返回除`\n`以外的所有`NavigableString`对象

```python
for string in soup_test.stripped_strings:
    print(string)
# The Dormouse's story
# The Dormouse's story
# Once upon a time there were three little sisters; and their names were
# Elsie
# ,
# Lacie
# and
# Tillie
# ;
# and they lived at the bottom of a well.
# ...
```

### 向上遍历

#### parent

对于每个节点，可以通过`parent`属性访问该节点上一层的节点，例如`<title>`的上层节点为`<head>`。

需要注意的是几种特殊节点的`parent`：

1. `NavigableString`对象的`parent`上层节点为包裹该字串的标签
2. 类似`<html>`的顶层标签的上层节点为`BeautifulSoup`对象。
3. 而`BeautifulSoup`对象的上层节点则为`None`

```python
print('--------')
title_tag = soup_test.head.title
print(title_tag)
print(title_tag.parent)
print(title_tag.string.parent)
html_tag = soup_test.html
print(type(html_tag.parent))
print(soup_test.parent

# <title>The Dormouse's story</title>
# <head><title>The Dormouse's story</title></head>
# <title>The Dormouse's story</title>
# <class 'bs4.BeautifulSoup'>
# None
```

#### parents

`parent`属性只能向上访问一级，而`parents`则可以按层序访问所有的**祖先节点**

```python
a_tag = soup_test.a
print(a_tag)
for parent in a_tag.parents:
    print(parent.name)

# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
# p
# body
# html
# [document]
```

### 同级遍历

有时候某一标签包含多个子标签，这些子标签处于同一级别，我们成为**siblings**，当我们身处某一节点，却想要访问该节点的兄弟节点时，这样从操作称为同级遍历.

我们用如下文档进行举例：

```python
sibling_soup = BeautifulSoup("<a><b>text1</b><c>text2</c></b></a>", 'html.parser')
print(sibling_soup.prettify())
#   <a>
#    <b>
#     text1
#    </b>
#    <c>
#     text2
#    </c>
#   </a>
```

#### next_sibling & previous_sibling

BS4为我们提供了两种用于在同级标签之间切换的属性`next_sibling`和`previous_sibling`，这两个函数分别返回下一个标签的tag对象以及上一个标签的tag对象。如果基准元素没有下一个或上一个标签，则返回`None`：

```python
sibling_soup.b.next_sibling
# <c>text2</c>

sibling_soup.c.previous_sibling
# <b>text1</b>

print(sibling_soup.b.previous_sibling)
# None
print(sibling_soup.c.next_sibling)
# None

sibling_soup.b.string
# 'text1'

print(sibling_soup.b.string.next_sibling)
# None
```

但注意到一个问题，使用`strings`遍历时会发现，其中包含很多`/n`，这些字符也被视为`NavigableString`对象，因此使用这两个属性进行同级访问时也会访问到他们，例如下面这个例子：

```python
# <a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>
# <a href="http://example.com/lacie" class="sister" id="link2">Lacie</a>
# <a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>

link = soup.a
link
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

link.next_sibling
# ',\n '

link.next_sibling.next_sibling
# <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
```

#### next_slibings & previous_sliblings

如果需要访问与当前节点同级的所有标签，则可以使用`next_slibings`和`previous_siblings`两个属性。这两个属性分别返回一个包含所有当前节点之后的所有同级节点的生成器，和之前的所有同级节点的生成器：

```python
for sibling in soup.a.next_siblings:
    print(repr(sibling))
# ',\n'
# <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
# ' and\n'
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
# '; and they lived at the bottom of a well.'

for sibling in soup.find(id="link3").previous_siblings:
    print(repr(sibling))
# ' and\n'
# <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
# ',\n'
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
# 'Once upon a time there were three little sisters; and their names were\n'

```

#### next_element & previous_element

该属性用于寻找当前节点的**按文件解析顺序**的下一个节点。

```python
last_a_tag = soup_test.find("a",id = 'link3')
print(last_a_tag)
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
print(last_a_tag.next_sibling)
# ;
# and they lived at the bottom of a well.
for child in last_a_tag.children:
    print(child)
    # Tillie
print(last_a_tag.next_element)
# Tillie
```

`previous_element`原理相似但结果是返回在当前标签之前被解析的标签。

#### next_elements & previous_elements

这两个属性则是返回当前标签后的解析和当前标签前需要解析的对象，下面我们使用`repr`函数将这些对象转换为供python解释器读取的形式以便观察：

```python
for element in last_a_tag.next_elements:
    print(repr(element))
# 'Tillie'
# ';\nand they lived at the bottom of a well.'
# '\n'
# <p class="story">...</p>
# '...'
# '\n'
```

## 搜索文档树

BeautifulSoup中提供了大量用于搜索的函数，但大部分函数的参数比较相似，这里主要讨论`find()`和`find_all()`两个方法。下面主要讲解`find_all`方法

### find_all

`find_all()`函数， 它会将所有符合条件的内容以列表形式返回。它的构造方法如下：

```python
find_all(name, attrs, recursive, text, **kwargs )
```

name 参数可以有多种写法：

- （1）节点名

```python
print(soup.find_all('p'))
# 输出结果如下：
[<p class="title" name="dromouse"><b>The Dormouse's story</b></p>, <p class="story">Once upon a time there were three little sisters; and their names were</p>]
```

- （2）正则表达式

```python
print(soup.find_all(re.compile('^p')))
# 输出结果如下：
[<p class="title" name="dromouse"><b>The Dormouse's story</b></p>, <p class="story">Once upon a time there were three little sisters; and their names were</p>]
```

- （3）列表
  如果参数为列表，过滤标准为列表中的所有元素。看下具体代码，你就会一目了然了。

```python
print(soup.find_all(['p', 'a']))
# 输出结果如下：
[<p class="title" name="dromouse"><b>The Dormouse's story</b></p>,  <p class="story">Once upon a time there were three little sisters; and their names were</p>,  <a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>]
```

另外 attrs 参数可以也作为过滤条件来获取内容，而 limit 参数是限制返回的条数。

### 利用CSS选择器

除了只用find函数意外，CSS选择器也是一个非常方便的搜索手段，以 CSS 语法为匹配标准找到 Tag。同样也是使用到一个函数，该函数为`select()`，返回类型也是 list。它的具体用法如下, 同样以 prettify() 打印的结果为前提：

- （1）通过 tag 标签查找

```python
print(soup.select(head))
# 输出结果如下：
# [<head><title>The Dormouse's story</title></head>]
```

- （2）通过 id 查找

```python
print(soup.select('#link1'))
# 输出结果如下：
# [<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>]
```

- （3）通过 class 查找

```python
print(soup.select('.sister'))
# 输出结果如下：
# [<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>]
```

- （4）通过属性查找

```python
print(soup.select('p[name=dromouse]'))
# 输出结果如下：
# [<p class="title" name="dromouse"><b>The Dormouse's story</b></p>]
```



```python
print(soup.select('p[class=title]'))
# 输出结果如下：
# [<p class="title" name="dromouse"><b>The Dormouse's story</b></p>]
```

- （5）组合查找

```python
print(soup.select("body p"))
# 输出结果如下：
# [<p class="title" name="dromouse"><b>The Dormouse's story</b></p>,
# <p class="story">Once upon a time there were three little sisters; and their names were</p>]
```



```python
print(soup.select("p > a"))
# 输出结果如下：
# [<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>]
```



```python
print(soup.select("p > .sister"))
# 输出结果如下：
# [<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>]
```

## 总结

至此已经可以利用BeautifulSoup来进行一些简单的HTML解析工作了。剩下的一些属性以及函数，可以在应用中学习。

在本次学习的过程中，我意识到这样看官方文档并做笔记的学习方式，效率比较低。

而且我也没办法做到将官方文档概况的面面俱到，某次和队友聊天的过程中偶然提到了目前的学习状况。队友直截了当的指出了我学习上的不足：“BS完全不值得你去做笔记学习！”

确实，现阶段我已经进入了研究生的学习阶段，我的学习也不再是通过一些相关技术来了解本专业的过程了。而应该转化为通过所学的一些技术对当前所学领域进行一些研究，解决或是发现一些领域内比较含糊的问题。

工具终究是工具，研究才是现阶段应该关注的。我认为比起依赖于笔记，作为一名合格的硕士研究生，应该养成看API文档的习惯。

## 参考文档

[Beautiful Soup Documentation — Beautiful Soup 4.9.0 documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)

