---
layout:     post
title:      "WikiText 初识"
category : language-instrument
date:       2019-03-13
author:     "Max"
header-img: "img/post-2019.jpg"
catalog:    true
tags:
    - tw
---

# 1. 概念

与 markdown 类似，WikiText 也是一种轻便的文本标记语言，但是更专注于链接和交互的特性。

# 2. 语法

## 2.1 段落

与 md 相同，单换行符将被忽略，双换行符表明段落的结束。

特别的，被`"""`引起来的与块中，单换行符可以起到预期效果。

## 2.2 引文

### 2.2.1 单行引文

格式与 md 类似，同时支持嵌套使用：
```
> A top quote
>> A subquote
> Another top quote
```

### 2.2.2 多行引文

示例：
```
<<<
This is a block quoted paragraph
written in English
<<<
```

同时，起始`<<<`后可以指定 CSS类添加特效，结束`<<<`后可以添加引语。示例：
```
<<<.tc-big-quote
Operating systems are like a brick wall for our minds
<<< Nobody
```
<blockquote class="tc-quote tc-big-quote"><p>Operating systems are like a brick wall for our minds
</p><cite>Nobody</cite></blockquote>

## 2.3 代码块

语法与 md 相同。需要注意的是，前置文本与代码块之间需要间隔两个连续换行符，示例：
```
    This is an ordinary paragraph
    
    ```
    This will be monospaced
    ```
```

## 2.4 字体

* `''bold''` 粗体
* `//italic//`斜体
* `__underscore__`下划线
* `^^superscript^^`上标
* `,,subscript,,`下标
* `~~strikethrough~~`删除线

## 2.5 HTML 特性

支持 HTML 定义列表，示例：
```
; Term being defined
: Definition of that term
; Another term
: Another definition
```

<dl><dt>Term being defined</dt><dd>Definition of that term</dd><dt>Another term</dt><dd>Another definition</dd></dl>

此外，HTML tags 和 comments 都可直接在 WikiText 中使用。

还可直接使用 CSS 类修饰语块：
```
@@.tc-tiddler-frame
@@width:400px;
Some text
@@
```
<p class="tc-tiddler-frame" style="width:400px;">Some text
</p>

## 2.6 标题

标题以`!`表明：
```
! This is a level 1 heading

!! This is a level 2 heading

!!! This is a level 3 heading
```

CSS 类可以指定到一个标题:
```
!.myStyle This heading has the class `myStyle`
```

## 2.7 图片

添加图片的语法如下：
```
[img[Motovun Jack.jpg]]
[img[https://tiddlywiki.com/favicon.ico]]
```

可以添加提示信息：
```
[img[An explanatory tooltip|Motovun Jack.jpg]]
```

可以指定其他属性，并且这些属性可以使用 transclusions 或 variable references：
```
[img width=32 class="tc-image" [Motovun Jack.jpg]]
[img width={{!!mywidth}} class=<<image-classes>> [Motovun Jack.jpg]]
```

## 2.8 链接

### 2.8.1 内部链接

指向一个 tiddler 的链接的语法为：
```
[[Tiddler Title]]
[[Displayed Link Title|Tiddler Title]]
```

### 2.8.2 外部链接

指向外部资源时的链接语法为：
```
https://tiddlywiki.com/

[[TW5|https://tiddlywiki.com/]]

[[Mail me|mailto:me@where.net]]

[[Open file|file:///c:/users/me/index.html]]
```

为避免相对路径式的外部资源被解析做内部链接，可使用扩展语法：
```
[ext[Open file|index.html]]

[ext[Open file|./index.html]]

[ext[Open file|../README.md]]

[ext[Open file|c:\users\me\index.html]]
```

需注意，驼峰式的 tiddler 标题会自动创建链接，可通过添加前缀`~`取消这种自动创建的连接：
```
* ~HelloThere is not a link
* ~http://google.com/ is not a link
```

## 2.9 列表

### 2.9.1 无序列表

使用 `*`创建无序列表，支持嵌套使用：
```
* First list item
* Second list item
** A subitem
* Third list item
```

注意，与 md 使用缩进嵌套使用多级列表不同，WikiText 次级列表前不可有空格，需要使用`*`补齐。

单项中若需包含多行文本，可以使用 transclude 方式，或使用 HTML "div" 元素：
```
# Step 1
# Step 2<div>

Here is the first of several paragraphs. Note that the double linebreak preceding this paragraph is significant.

And here is the second of several paragraphs.
</div>
# Step 3
```

此外，可以使用 CSS 类修饰任意项：
```
* List One
*.MyClass List Two
* List Three
```

### 2.9.2 有序列表

使用 `#`创建有序列表，其它特性与`*`完全相同：
```
* To do today
*# Eat
* To get someone else to do
*# This
*# That
*## And the other
```

## 2.10 表格

### 2.10.1 基础语法

`|`代表单元格分隔符，`!`表明表头：
```
|!Cell1 |!Cell2 |
|Cell3 |Cell3 |
```

### 2.10.2 单元格水平对齐

文本紧邻单元格边框代表对齐方式，还可指定填充方式：
```
|Left aligned content |
| Right aligned content|
| Centred content |
|+++ a very wide column so we can see the alignment +++|
```

### 2.10.3 单元格垂直对齐

单元格分隔符后第一个字符`^`、`,`指定单元格垂直对齐方式：
```
|^top left |^ top center |^ top right|
|middle left | middle center | middle right|
|,bottom left |, bottom center |, bottom right|
```

对于水平左对齐的单元格，其文本的首字符`^`、`,`可分别使用 HTML 的转义字符`&#94;`、`&#44;`代替。

### 2.10.4 单元格合并

`~`代表向上合并，`<`代表向左合并，`>`代表向右合并：
```
|Cell1 |Cell2 |Cell3 |Cell4 |
|Cell5 |Cell6 |Cell7 |<|
|Cell5 |~|Cell7 |Cell8 |
|>|Cell9 |Cell10 |Cell11 |
```

### 2.10.5 设定表格属性

支持使用如下方式设定表格的 CSS 类、表头、表尾、提示：
```
|myclass anotherClass|k
|This is a caption |c
|Cell1 |Cell2 |
|Cell3 |Cell3 |
|Header|Header|h
|Footer|Footer|f
```

<table class="myclass anotherClass"><caption>This is a caption </caption><tbody><tr class="evenRow"><td align="left">Cell1</td><td align="left">Cell2</td></tr><tr class="oddRow"><td align="left">Cell3</td><td align="left">Cell3</td></tr></tbody><thead><tr class="evenRow"><td>Header</td><td>Header</td></tr></thead><tfoot><tr class="oddRow"><td>Footer</td><td>Footer</td></tr></tfoot></table>

## 2.11 嵌入引用 Transclusion

可以使用如下 Transclusion 标记语法引用其它 tiddler 的内容： 

* `{{MyTiddler}}` 嵌入引用单个 tiddler
* `{{MyTiddler||TemplateTitle}}` 通过一个指定的模板展示 tiddler
* `{{||TemplateTitle}}` 不修改当前 tiddler 的情况下展示指定模板 tiddler

可以指定 tiddler 的部分内容进行引用：
* `{{MyTiddler!!field}}` 嵌入引用指定 tiddler 的指定字段
* `{{!!field}}` 嵌入引用当前 tiddler 的指定字段
* `{{MyTiddler##index}}` 嵌入引用指定 tiddler 的指定索引号的属性
* `{{##index}}` 嵌入引用当前 tiddler 的指定索引号的属性

可以可用相似语法引用满足指定过滤器的多个 tiddler：
```
{{{ [tag[mechanism]] }}}
{{{ [tag[mechanism]] ||TemplateTitle}}}
```

## 2.12 变量 Variable

支持使用  $set 组件定义变量:
```
<$set name=animal value=zebra>
<<animal>>
</$set>
```

宏 Macros 是包含占位符的特殊变量，占位符在宏使用时填充:
```
\define tags-of-current-tiddler() {{!!tags}}

The tags are: <<tags-of-current-tiddler>>
```

变量可用作过滤器参数：
```
<<list-links filter:"[<currentTiddler>backlinks[]]">>
```
$list 组件内置了变量 currentTiddler 变量，上述语句利用 backlinks 操作可列出链向当前 tiddler 的全部 tiddler。

## 2.13 组件 Widget

组件语法与 HTML 元素相同，但标记名以 `$` 开始。组件可以为 WikiText 提供丰富的功能。

# 3. 参考文档

1. [WikiText](https://tiddlywiki.com/static/WikiText.html)