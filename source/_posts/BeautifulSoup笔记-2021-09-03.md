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
print(soup_test.contents)
```

