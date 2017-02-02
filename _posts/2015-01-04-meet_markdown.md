---
layout: post
title: markdown初识
category : 语言与工具
tagline:
tags : [md]
---
{% include JB/setup %}

***
# 基本语法

* **语块**

  用空行来区分语块


* **标题**

  用`#`的个数（行首）区别标题级数，最多六级标题


* **引用**

  用`>`来表明之后的内容为引用


* **字体**

  用  `**`  或 `__` 括起来的文字 **加粗**

  用 `*` 或 `_` 括起来的文字 _倾斜_

  嵌套使用时用不同的符号更易于区分，如`_You **can** combine them_`

* **列表**

  符号要和文字之间加上一个字符的空格，行首空2格可以降为二级列表，同理类推。

  * 无序列表

    每行行首加一个  `*`  或  `-`

  * 有序列表

    每行行首加数字接着一个英文句点，符号要和文字之间加上一个字符的空格


* **超链接**

  在`[]`中写要添加超链接的文字或说明，紧接着在`()`中写入超链接地址。如`[Visit GitHub!](www.github.com)`


* **图片**

  格式仿照超链接，不过要在首部加一个`!`。如`![GitHub Logo](/images/logo.png)`

  插入图片的地址需要图床，比较好的图床工具有[Droplr](https://droplr.com/hello)、[Cloudapp](https://www.getcloudapp.com/)、[ezShare for Mac](https://itunes.apple.com/cn/app/yi-xiang/id672522335?mt=12&ign-mpt=uo%3D4)。


* **表格**

  示例：
```
First Header | Second Header
------------ | -------------
Content from cell 1 | Content from cell 2
Content in the first column | Content in the second column
```

First Header | Second Header
------------ | -------------
Content from cell 1 | Content from cell 2
Content in the first column | Content in the second column


* **代码**

  一行中，用 ' ` ' (键盘上与 ' ~ ' 共用一个键位，在 ' 1 \ ! ' 键之前） 括起来的字段被识别为代码

  多行代码， 需用 " ``` " （占一行）括起来


* **分割线**

  `***`单独成行


***

# GitHub中的特色

* **语法高亮**

  示例：
```javascript
function fancyAlert(arg) {
  if(arg) {
    $.facebox({div:'#foo'})
  }
}
```

* **任务列表**

  示例：
```
- [x] @mentions, #refs, [links](), **formatting**, and <del>tags</del> supported
- [x] list syntax required (any unordered or ordered list supported)
- [x] this is a complete item
- [ ] this is an incomplete item
```

- [x] @mentions, #refs, [links](), **formatting**, and <del>tags</del> supported
- [x] list syntax required (any unordered or ordered list supported)
- [x] this is a complete item
- [ ] this is an incomplete item


* **自动转换**

  一些特殊的字符串（如URL网址、提交的版本号）会自动转换为链接

  在 HTML 文件中，有两个字符需要特殊处理： `<` 和 `&` 。 `<` 符号用于起始标签，`&` 符号则用于标记 HTML 实体，如果你只是想要显示这些字符的原型，你必须要使用实体的形式，像是 `&lt;` 和 `&amp;`。Markdown 中的这两个字符可以自动识别转换。


* **表情**

  GitHub 支持 emoji! :sparkles: :camel: :boom:

  表情详情可以查看[Emoji Cheat Sheet](http://www.emoji-cheat-sheet.com/)

***
