---
layout: post
title: Git工作流
category : language-instrument
author: Max
tags : [git, workflow, versioncontrol]
---


<item>
<title>Git工作流</title>
<content:encoded>
<h2>1. 前言</h2>

持续集成和部署是互联网团队必然要面对的问题。网上看到很多带着版本管理的工作流，只能说没有最好的、只有最合适的，需要在实际使用中总结归纳，不断优化。

有一些模型可供参考。

<h2>2. Gitflow工作流</h2>

这是一个比较完整的分支模型，较为复杂，适合管理大型项目。小项目中可以根据实际情况做出简化。

<img src="http://nvie.com/img/git-model@2x.png" alt="GitflowModule" />

<h3>2.1 长期分支</h3>

是持续整个项目周期而存在的分支，项目工程新建时即新建这些分支。

<h5><code>master分支</code></h5>

长期分支之一。在 master 分支上存储了正式发布的历史，可以随时从origin/master取到最新的可发布的代码和版本。

<h5><code>develop分支</code></h5>

长期分支之一，是master的平行分支，是功能的集成分支。

develop分支不必保持绝对稳定，但是一旦达到一个稳定点（需要符合版本计划），就可以被合并入 master 分支，打上标记发布版本。每日构建的代码可以从此分支取得。

<h3>2.2 短期分支</h3>

长期分支不直接做开发，一般从长期分支上支出短期分支编写或修改代码。根据实际目的和行为方式，短期分支有不同的分类。

<h5><code>feature分支</code></h5>

最常用的短期分支的一种，用于开发某一特定功能，从develop分支checkout，向develop分支checkin，命名规则feature-*。各个feature任务需要架构师/技术组长统筹版本计划后分配给项目团队。

一个feature功能应尽量小，使其生命周期不超过三个工作日。（若是预研性质的分支必须较长期内存在，则分支管理者应该以3-5日为周期频繁将develop分支合并到该feature分支，防止该功能分支终结时因差异过大难以合并到develop分支。）

<h5><code>release分支</code></h5>

短期分支的一种，作为预发布分支，从develop分支checkout并修改版本号后，发布给测试工程师测试，根据反馈做bug修复工作，通过回归测试后正式发布版本，并checkin到master和develop分支。命名规则release-* 。

<h5><code>hotfix分支</code></h5>

短期分支的一种，针对已正式发布的版本做紧急问题修复的分支，从master分支checkout，修改小版本号，修复问题后checkin到master和develop分支。命名规则hotfix-* 。

<h2>3. Forking工作流</h2>

与其它工作流最大的不同之处在于不使用单个服务器仓库作为代码基线，每个开发者都有一个独立的服务端仓库。换言之，每个开发者拥有两个远端，代码改动先提交至个人的服务端仓库，然后向公共服务端仓库发起Pull Request。这样项目维护者可以接受任何开发者的提交，而无需给他正式代码库的写权限。很多开源项目使用这种工作方式。

<img src="https://segmentfault.com/image?src=http://static.ixirong.com/pic/gitflow/git-workflows-forking.png&amp;objectId=1190000002918123&amp;token=556ec96e04a6af9aa60d8ec1c999ba30" alt="ForkingflowModule" />

和其它的Git工作流一样，Forking工作流要先有一个公开的<strong>正式仓库</strong>存储在服务器上。但一个新的开发者想要在项目上工作时，不是直接从正式仓库克隆，而是<strong>fork正式项目</strong>，在服务器上创建一个拷贝。

这个仓库拷贝作为他个人公开仓库，<strong>其它开发者不允许push到这个仓库，但可以pull到修改</strong>。在创建了自己服务端拷贝之后，开发者执行git clone命令克隆仓库到本地机器上，作为私有的开发环境。

要提交本地修改时，先push提交到自己公开仓库中，然后给正式仓库发起一个pull request，让项目维护者知道有更新已经准备好可以集成了。

对于贡献的代码，pull request也可以很方便地作为一个讨论的地方。

为了集成功能到正式代码库，维护者pull贡献者的变更到自己的本地仓库中，检查变更以确保不会让项目出错，合并变更到自己本地的master分支，然后push master分支到服务器的正式仓库中。

到此，贡献的提交成为了项目的一部分，其它的开发者应该执行pull操作与正式仓库同步自己本地仓库。

<h2>4. 参考</h2>

<ol>
<li><a href="http://nvie.com/posts/a-successful-git-branching-model/">《A successful Git branching model》nvie.com</a></li>
<li><a href="https://segmentfault.com/a/1190000002918123#articleHeader19">《深入理解学习Git工作流（git-workflow-tutorial）》xirong 整理自 oldratlee 的github</a></li>
</ol>
</content:encoded>
