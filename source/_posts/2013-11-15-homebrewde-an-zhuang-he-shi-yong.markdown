---
layout: post
title: "Homebrew的安装和使用"
date: 2013-11-15 14:33
comments: true
categories: homebrew install  
---
<h3>Homebrew 是什么</h3>
<br>
官网是这样说的：<br>
<p>在 OS X 中找不到您想要的软件？Homebrew 给你所需。<br>Homebrew 将软件包分装到单独的目录，然后 symlink 到 /usr/local 中。</p>
<!-- more -->
<p>轻松创建您自己的 Homebrew 软件包。
``` ruby
	$ brew create http://foo.com/bar-1.0.tgz
	Created /usr/local/Library/Formula/bar.rb
```
完全基于 git 和 ruby，您可以很方便地撤销、更改或合并上流的更新。</p>
<br>
<h3>安装 Homebrew</h3>
<p>打开 Terminal, 复制并粘贴:</p>
``` ruby
	ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
```
<br>
<h3>Homebrew 使用</h3>
<p>查看寻找自己的包</p>
``` ruby
	brew search //+keyword
```
<p>安装包</p>
``` ruby
	brew install // + 包名
```

