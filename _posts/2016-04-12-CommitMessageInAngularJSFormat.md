---
layout: post
title: Commit message in AngularJS format
category : language-instrument
author: Max
tags : [versioncontrol, commit message]
---


<item>
  <title>Commit message in AngularJS format</title>

  <content:encoded><h2>1. 前言</h2>

代码管理时，我每次提交的信息都比较散乱，后期查看历史时往往不能很快找到自己想要的信息。从<a href="http://www.commitlogsfromlastnight.com/">commitlogsfromlastnight</a>网站处可以看到各路网友各式各样的commit message，可以感受下。读了阮一峰老师的文章后有一些领悟，学习&amp;记录。

<h2>2. AngularJS规范</h2>

<h3>2.1 目标</h3>

这也是格式化的Commit message的优势。

<ol>
<li>查询历史时，提供更简明的信息

只看首行即可清楚每次提交的大概：

<pre><code>$ git log HEAD --pretty=format:%s
</code></pre></li>
<li>过滤某一类型的提交信息，快速查找

比如只查看新增的功能：

<pre><code>$ git log HEAD --grep feature
</code></pre>

或者过滤掉某些不重要的提交（空行、注释等）：

<pre><code>$ git bisect skip $(git rev-list --grep irrelevant HEAD)
</code></pre></li>
<li>自动生成CHANGELOG

CHANGELOG是发布新版本时，用来说明与上一个版本差异的文档，一般包含<strong>new features, bug fixes, breaking changes</strong>等信息。</p></li>
</ol>

<h3>2.2 格式</h3>

<p>每个Commit message包括三个部分，<code>Header</code>、<code>Body</code>、<code>Footer</code>，以空行分隔。

<em>注：任何一行都不要超过72个字符，避免自动换行影响美观。</em>

<h4>2.2.1 Revert</h4>

有一种特殊情况，即如果当前commit用于撤销之前的某个 commit，则必须以'revert:'开头，后面跟着被撤销commit的<code>Header</code>。示例：

<pre><code>revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
</code></pre>

<em>注：Body部分的格式是固定的，必须写成<code>This reverts commit &lt;hash&gt;.</code>。其中的 hash 是被撤销 commit 的 SHA 标识符。</em>

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 CHANGELOG 里面。如果两者在不同的发布，那么当前 commit，会出现在 CHANGELOG 的Reverts小标题下面。

<h4>2.2.2 Header</h4>

每个<code>Header</code>也包含三个部分，<code>type</code>（必）、<code>scope</code>（选）、<code>subject</code>（必）。

<h5><strong>type</strong></h5>

<code>type</code>标明commit的类型，分为如下7个:

<ul>
<li><strong>feat</strong> (feature)</li>
<li><strong>fix</strong> (bug fix)</li>
<li><strong>docs</strong> (documentation)</li>
<li><strong>style</strong> (formatting, missing semi colons, …)</li>
<li><strong>refactor</strong></li>
<li><strong>test</strong> (when adding missing tests)</li>
<li><strong>chore</strong> (maintain)</li>
</ul>

<h5><strong>scope</strong></h5>

<code>scope</code>标明commit的影响范围，可以是\$location, \$browser, \$compile, \$rootScope, ngHref, ngClick, ngView等，视情况而定。

<h5><strong>subject</strong></h5>

<code>subject</code>是commit的简要描述，要求

<ul>
<li>动宾短语（祈使语气，一般现在时）</li>
<li>首字母小写</li>
<li>结尾不加句号</li>
</ul>

<h4>2.2.3 Body</h4>

<code>Body</code>是对本次commit的详细描述，可以分成多行。有两个注意点:

<ul>
<li>和<code>subject</code>一样，使用祈使语气，一般现在时态。</li>
<li>说明代码变动的动机，以及与以前行为的对比。</li>
</ul>

<h4>2.2.4 Footer</h4>

<code>Footer</code>用于两种情形，不兼容变动<code>Breaking changes</code>和Issue关联<code>Referencing issues</code>。

<h5><strong>Breaking changes</strong></h5>

如果当前代码与上一个版本不兼容，则 Footer 部分以BREAKING CHANGE开头，后面是对变动的描述、以及变动理由和迁移方法。示例：

<pre><code>BREAKING CHANGE: isolate scope bindings definition has changed and
the inject option for the directive controller injection was removed.

To migrate the code follow the example below:

Before:

scope: {
myAttr: 'attribute',
myBind: 'bind',
myExpression: 'expression',
myEval: 'evaluate',
myAccessor: 'accessor'
}

After:

scope: {
myAttr: '@',
myBind: '@',
myExpression: '&amp;amp;',
// myEval - usually not useful, but in cases where the expression is assignable, you can use '='
myAccessor: '=' // in directive's template change myAccessor() to myAccessor
}

The removed `inject` wasn't generaly useful for directives so there should be no code using it.
</code></pre>

<h5><strong>Referencing issues</strong></h5>

如果本次commit与某个Issue相关，或者是关闭某个BUG Issue，可以在<code>Footer</code>中将其关联起来。示例：

<pre><code>Closes #123, #245, #992
</code></pre>

<h3>2.3 示例</h3>

<pre><code>feat($browser): onUrlChange event (popstate/hashchange/polling)

Added new event to $browser:
- forward popstate event if available
- forward hashchange event if popstate not available
- do polling when neither popstate nor hashchange available

Breaks $browser.onHashChange, which was removed (use onUrlChange instead)
</code></pre>

<hr />

<pre><code>fix($compile): couple of unit tests for IE9

Older IEs serialize html uppercased, but IE9 does not...
Would be better to expect case insensitive, unfortunately jasmine does
not allow to user regexps for throw expectations.

Closes #392
Breaks foo.bar api, foo.baz should be used instead
</code></pre>

<hr />

<pre><code>feat(directive): ng:disabled, ng:checked, ng:multiple, ng:readonly, ng:selected

New directives for proper binding these attributes in older browsers (IE).
Added coresponding description, live examples and e2e tests.

Closes #351
</code></pre>

<hr />

<pre><code>style($location): add couple of missing semi colons
</code></pre>

<hr />

<pre><code>docs(guide): updated fixed docs from Google Docs

Couple of typos fixed:
- indentation
- batchLogbatchLog -&gt; batchLog
- start periodic checking
- missing brace
</code></pre>

<hr />

<h2>3. 参考</h2>

<ol>
<li><a href="http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html">《Commit message 和 Change log 编写指南》阮一峰的网络日志</a></li>
<li><a href="https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0">《Git Commit Message Conventions》google文档</a></li>
</ol></content:encoded>

</item>
