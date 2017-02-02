---
layout: post
title: Build GitHub Pages with Jekyll-Bootstrap
category : language-instrument
author: Max
tags : [github-pages, jekyll, jekyll-bootstrap]
---

This is a note on how I start this blog.


## 1. Create a New Repository

Go to https://github.com and create a new repository named `malikey.github.io`.
(`malikey` is my git-hub ID.)

## 2. Get Jekyll-Bootstrap

Make sure you have install ruby and jekyll already and they work well first.

Enter these commands into terminal in a directory you want your blog to be:

```
$ git clone https://github.com/plusjade/jekyll-bootstrap.git myblog
$ cd myblog
$ git remote rm origin
$ git remote add origin git@github.com:malikey/malikey.github.io.git
```

After this step, you will get a directory with structure like so:

```
.
├── _config.yml
├── _drafts
├── _includes
├── _layouts
|   ├── default.html
|   ├── page.html
|   └── post.html
├── _plugins
├── _posts
├── _data
|   └── members.yml
├── _site
├── assets
├── index.html
├── README.md
.      .
.      .
└── sitemap.txt
```

`_posts` saves all of blogs in format of `YEAR-MONTH-DAY-title.md`. You can get a brief introduction at [jekyll official website](http://jekyllrb.com/docs/structure/).

## 3. Configure

### 3.1 Install a Theme

The default theme is some like twitter. If you do not like it, you could get a new one
from http://themes.jekyllbootstrap.com/ .
Install a theme like so:

```
rake theme:install git="https://github.com/dhulihan/hooligan.git"
```

### 3.2 Update Author Attributes

In _config.yml remember to specify your own data:

```
title : Halt in Air
tagline: They lived happily ever after.
author :
  name : Xin Ma
  email : maxinnx@163.com
  github : malikey

markdown: rdiscount  

production_url : http://malikey.github.io
```

_**Note: **_[`rdiscount`](http://rubygems.org/gems/rdiscount) is a fast implementation of gruber's markdown in C.
Install it manually. You may also need to install [`disqus`](https://www.disqus.com/), a comment plugin, if you
want to preview your site locally. Register disqus [here](https://disqus.com/admin/create/).

### 3.3 Rewrite `index.md`

`index.md` is default home page of your blog site. Before rewriting it, strongly recommend you reading it first.

## 4. Push

Save changes and push it to github, and your blog will be available after several minutes at http://malikey.github.io .

```
$ git push -u origin master
```

Of course, if you can't wait any longer (So do I ^_^), you can boot jekyll on your machine as an alternative.

```
$ jekyll serve
```

Your blog is now available at: http://localhost:4000/. Enjoy it!

## 5. Reference

1. [在Github上搭建Jekyll博客和创建主题](http://yansu.org/2014/02/12/how-to-deploy-a-blog-on-github-by-jekyll.html)

2. [使用 GitHub, Jekyll 打造自己的免费独立博客](http://blog.csdn.net/on_1y/article/details/19259435)

3. [空想枫de博客](http://blog.it580.com/tag/jekyll/)
