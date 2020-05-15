---
layout: post
status: publish
published: true
title: git同步多个repository仓库
description: "git 同步多个 repository 仓库"
excerpt_separator: ===
tags:
- 其他技术
---


由于国内服务器访问GitHub奇慢，所以把仓库复制一份放在了国内。但是苦于要同步两边，所以想有没有办法，同步多个git仓库，是否有办法，只维护一份就可以了。

===

网上搜了一下，大都说的是

		git remote add 

		和 git remote set-url --add ，

		以及 git push origin --all 的用法。

但是自己用了一下，完全用不了。总是拒绝push，说需要先pull，但pull了又说需要merge，我感觉这个坑巨大，于是想找其他法子。自己想了想最土的办法或许可以。试验了下，果然可行。步骤如下：

		1， 两个文件夹，分别装两个仓库，或者多个文件夹装多个仓库，每个不同的仓库一个文件夹。

		2， 把其中一个作为主要维护的仓库，维护完毕后，执行以下步骤。

		3， 写一个shell程序。首先把所有内容全部都复制粘贴到其他仓库去，直接替换掉旧的文件。然后每个仓库一个个的执行

			git add .

			git commit -m 'sync'

			git push origin master

			每个仓库都执行完毕后，同步完成。

也就是，用shell程序把人工手动需要做的事情，让程序去完成，省时省力，一键搞定。

{% include advertisement_content.html %}

###### 举例代码为同步三个仓库，具体的 shell 如下：

		cp -r ../* /Users/luzexi/Desktop/work/gitee/blog/
		cd /Users/luzexi/Desktop/work/gitee/blog/
		git add .
		git commit -m 'sync'
		git push origin master

		cp -r ../* /Users/luzexi/Desktop/work/github/blog/
		cd /Users/luzexi/Desktop/work/github/blog/
		git add .
		git commit -m 'sync'
		git push origin master

		cp -r ../* /Users/luzexi/Desktop/work/gitlab/blog/
		cd /Users/luzexi/Desktop/work/gitlab/blog/
		git add .
		git commit -m 'sync'
		git push origin master

