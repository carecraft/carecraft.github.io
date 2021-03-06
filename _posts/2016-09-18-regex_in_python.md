---
layout: post
title: 在python中使用正则表达式
category : language-instrument
author: Max
tags : [python, regular_expressions]
---


<item>
<title>在python中使用正则表达式</title>
<description></description>
<content:encoded>

<h2>一、 查找首个匹配串</h2>

<h3>1.1 简单示例</h3>

<pre><code class="python">str = 'an example word:cat!!'
match = re.search(r'word:\w\w\w', str)
# The 'r' at the start of the pattern string designates a python "raw" string which passes through backslashes without change
# If-statement after search() tests if it succeeded
if match:                      
print 'found', match.group() ## 'found word:cat'
else:
print 'did not find'
</code></pre>

<h3>1.2单一字符</h3>

<ul>
<li><code>a</code>, <code>X</code>, <code>9</code>, <code>&lt;</code> -- 一般字符匹配自身。</li>
<li><code>.</code> -- 匹配除换行符<code>\n</code>外的任意单一字符。</li>
<li><code>\w</code> -- 匹配任意一个单词字符<code>[a-zA-Z0-9_]</code>。</li>
<li><code>\W</code> -- 匹配任意一个非单词字符。</li>
<li><code>\b</code> -- 单词字符和非单词字符的边界。一个零宽界定符，只用以匹配单词的词首和词尾。</li>
<li><code>\B</code> -- 单词字符和单词字符的连接。另一个零宽界定符，在当前位置不在单词边界时匹配。</li>
<li><code>\s</code> -- 匹配任意一个空字符<code>[ \n\r\t\f\v]</code>。</li>
<li><code>\S</code> -- 匹配任意一个非空字符。</li>
<li><code>\t</code>,<code>\n</code>,<code>\r</code>,<code>\f</code>,<code>\v</code> -- 制表符，换行符，回车符，换页符，垂直制表符。</li>
<li><code>\d</code> -- 匹配任意一个非数字字符<code>[0-9]</code> 。</li>
<li><code>\D</code> -- 匹配任意一个非数字字符 。</li>
<li><code>^</code> = start, <code>$</code> = end -- 分别匹配字符串首尾</li>
<li><code>\</code> -- 取消一个字符的特殊性，使右边的任意字符按照一般字符进行匹配。</li>
</ul>

注：在 Python 字符串里，<code>\b</code>是反斜杠字符，ASCII值是8。如果没有使用 raw 字符串时，那么 Python 将会把 <code>\b</code> 转换成一个回退符，RE 将无法像希望的那样匹配它了。

<h3>1.3 方括号</h3>

<ul>
<li><code>[]</code> -- 表示可能出现字符的集合，可填充上述任意字符。需注意，在方括号中：

<ul>
<li><code>.</code>仅代表一般字符<code>.</code>；</li>
<li><code>-</code>位于末尾代表一般字符<code>-</code>，否则代表一个区间；</li>
<li><code>^</code>位于起始代表“非”，匹配所有不在集合中的字符；</li>
</ul></li>
</ul>

<h3>1.4 重复</h3>

<ul>
<li><code>+</code> -- 表示左边的字符重复出现一次或更多次。</li>
<li><code>*</code> -- 表示左边的字符重复出现零次或更多次。</li>
<li><code>?</code> -- 表示左边的字符重复出现零次或一次。</li>
<li><code>{m,n}</code> -- 表示有 m 个重复(默认为0），至多到 n 个重复（默认整数类型极大值）。换言之，<code>{0,}</code> 等同于 <code>*</code>，<code>{1,}</code> 等同于 <code>+</code>，而<code>{0,1}</code>则与 <code>?</code> 相同。</li>
</ul>

匹配遵循两个原则
1. 找到字符串满足模式串的最左子串
2. 子串中的<code>+</code>、<code>*</code>匹配到最大长度

<h3>1.5 圆括号</h3>

<ul>
<li><code>()</code> -- 在模式串上加上圆括号不改变其匹配内容，但是会按顺序将结果子串分组。

<pre><code class="python">str = 'purple alice-b@google.com monkey dishwasher'
match = re.search('([\w.-]+)@([\w.-]+)', str)
if match:
print match.group()   ## 'alice-b@google.com' (the whole match)
print match.group(1)  ## 'alice-b' (the username, group 1)
print match.group(2)  ## 'google.com' (the host, group 2)
</code></pre></li>
</ul>

<h2>二、 查找全部匹配串</h2>

<h3>2.1 简单示例</h3>

<pre><code class="python">## Suppose we have a text with many email addresses
str = 'purple alice@google.com, blah monkey bob@abc.com blah dishwasher'

## Here re.findall() returns a list of all the found email strings
emails = re.findall(r'[\w\.-]+@[\w\.-]+', str) ## ['alice@google.com', 'bob@abc.com']
for email in emails:
# do something with each found email string
print email
</code></pre>

<h3>2.2 分组</h3>

<pre><code class="python">str = 'purple alice@google.com, blah monkey bob@abc.com blah dishwasher'
tuples = re.findall(r'([\w\.-]+)@([\w\.-]+)', str)
print tuples  ## [('alice', 'google.com'), ('bob', 'abc.com')]
for tuple in tuples:
print tuple[0]  ## username
print tuple[1]  ## host
</code></pre>

注：在括号内开始位置添加<code>?:</code>可以不将该括号当做分组处理。

<h2>三、 高级</h2>

<h3>3.1 额外参数</h3>

re模块函数接收额外参数改变其匹配行为，命令示例：<code>re.search(pat, str, re.IGNORECASE)</code>。参数说明:

<ul>
<li><code>IGNORECASE</code> -- 忽略大小写。（默认匹配区分大小写。）</li>
<li><code>DOTALL</code> -- 使<code>.</code>可以匹配换行符<code>\n</code>。（默认<code>.</code>匹配除换行符<code>\n</code>外的其它任意字符。）</li>
<li><code>MULTILINE</code> -- 在一个包含多行的长字符串中，使得<code>^</code>、<code>$</code>可以匹配每一行的行首、行尾。（默认<code>^</code>、<code>$</code>只能匹配到整个字符串的首尾。）</li>
</ul>

<h3>3.2 最短匹配</h3>

一般情况下，<code>*</code>会匹配到符合的最大长度为止，如模式串<code>&lt;.*&gt;</code>匹配字符串<code>&lt;b&gt;foo&lt;/b&gt; and &lt;i&gt;so on&lt;/i&gt;</code>会得到整个字符串<code>&lt;b&gt;foo&lt;/b&gt; and &lt;i&gt;so on&lt;/i&gt;</code>。若是在<code>*</code>/<code>+</code>后加上一个<code>?</code>则可以使<code>*</code>/<code>+</code>最快终结，得到<code>&lt;b&gt;</code>。

同样目的，一个更普遍的用法为<code>&lt;[^&gt;]*&gt;</code>。

<h3>3.3 替换</h3>

示例：

<pre><code class="python">str = 'purple alice@google.com, blah monkey bob@abc.com blah dishwasher'
## re.sub(pat, replacement, str) -- returns new string with all replacements,
## \1 is group(1), \2 group(2) in the replacement
print re.sub(r'([\w\.-]+)@([\w\.-]+)', r'\1@yo-yo-dyne.com', str)
## purple alice@yo-yo-dyne.com, blah monkey bob@yo-yo-dyne.com blah dishwasher
</code></pre>

</content:encoded>

</item>
