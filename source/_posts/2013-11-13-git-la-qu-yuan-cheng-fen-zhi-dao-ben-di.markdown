---
layout: post
title: "git 拉取远程分支到本地"
date: 2013-11-13 11:15
comments: true
categories: git, git checkout, git remote, git branch 
---
        git checkout -b newbranch_name --track origin/feature/newbranch_name
<p>如果遇到类似如下问题：</p>
        fatal: git checkout: updating paths is incompatible with switching branches.
        Did you intend to checkout 'origin/remote-name' which can not be resolved as commit?
<p>使用下面命令</p>
        git remote show origin
        git remote update
        git fetch
<p>再试试</p>
        git checkout -b local-name origin/remote-name
<p>参考资料:</p> <a href = "http://stackoverflow.com/questions/945654/git-checkout-on-a-remote-branch-does-not-work">Git checkout on a remote branch does not work</a>
