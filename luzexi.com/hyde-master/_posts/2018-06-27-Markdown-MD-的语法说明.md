---
layout: post
status: publish
published: true
title: Markdown(MD)的语法说明
description: "Markdown(MD)的语法说明 md语法 Markdown语法"
excerpt_separator: ????
categories:
- 其他技术
tags:
- md
- 语法
- markdown
---

#Markdown(MD)的语法说明

针对中文,演示Markdown的各种语法
  
标题大小用多个\#号表示大小，\#越多字体越小

\#
#这是 H1

\#\#
##这是 H2

\#\#\#
### 这是 H3

\#\#\#\#
#### 这是 h4

\#\#\#\#\#
##### 这是 h5

\#\#\#\#\#\#
###### 这是 h6

????

### 注意!!!下面所有语法的提示我都先用小标题提醒了!!! 

### 单行文本框
    这是一个单行的文本框,只要1个Tab再输入文字即可
        
### 多行文本框  
    	这是一个有多行的文本框
    	你可以写入代码等,每行文字只要输入2个Tab再输入文字即可
    	这里你可以输入一段代码

### 比如我们可以在多行文本框里输入一段代码,来一个Java版本的HelloWorld吧
	    public class HelloWorld {
	      /**
	      * @param args
		    */
		    public static void main(String[] args) {
			    System.out.println("HelloWorld!");

		    }

	    }

- - -

{% include advertisement_content.html %}

### 链接

链接内容定义的形式为：

	方括号（前面可以选择性地加上至多三个空格来缩进），里面输入链接文字
	接着一个冒号加空格或直接括号
	接着链接的网址
	选择性地接着 title 内容，可以用单引号、双引号或是括弧包着

例如:

		[点击这里你可以链接到www.baidu.com](http://www.baidu.com)<br />
		[点击这里我你可以链接到我的博客](http://luzexi.com)<br />

效果:

1. [点击这里你可以链接到www.baidu.com](http://www.baidu.com)<br />
2. [点击这里我你可以链接到我的博客](http://luzexi.com)<br />

- - -

### 显示图片
		[]前面加!就代表图片了，其他和普通的连接差不多
		![icon](http://luzexi.com/public/apple-touch-icon-144-precomposed.png "icon")

效果:

![icon](http://luzexi.com/public/apple-touch-icon-144-precomposed.png "icon")

###想点击某个图片进入一个网页,比如我想点击blog的icorn然后再进入www.luzexi.com
		[]中加入图片显示，进行嵌套操作
		[![image](http://luzexi.com/public/favicon.ico "blog")](http://www.luzexi.com/)
		这个可以分解拆分为[![image](http://luzexi.com/public/favicon.ico "blog")] 和 (http://www.luzexi.com/) 两部分

效果:

[![image](http://luzexi.com/public/favicon.ico "blog")](http://www.luzexi.com/)

- - -

### 文字被些字符包围

\> 文字被些字符包围

\> 只要再文字前面加上\>空格即可

\> 如果你要换行的话,新起一行,输入\>空格即可,后面不接文字

\> 但> 只能放在行首才有效

效果:

> 文字被些字符包围
>
> 只要再文字前面加上\>空格即可
>
> 如果你要换行的话,新起一行,输入\>空格即可,后面不接文字
> 但\> 只能放在行首才有效

- - -

{% include advertisement_content.html %}

### 文字被些字符包围,多重包围

\> 文字被些字符包围开始

\> \> 只要再文字前面加上\>空格即可

\>  \> \> 如果你要换行的话,新起一行,输入\>空格即可,后面不接文字

\> \> \> \> 但\> 只能放在行首才有效

效果:

> 文字被些字符包围开始
>
> > 只要再文字前面加上>空格即可
>
>  > > 如果你要换行的话,新起一行,输入>空格即可,后面不接文字
>
> > > > 但> 只能放在行首才有效

- - -

### 特殊字符处理
有一些特殊字符如<,#等,只要在特殊字符前面加上转义字符\即可<br />
你想换行的话其实可以直接用html标签\<br /\>

- - -

### 列表

##### 无序列表使用星号、加号或是减号作为列表标记：

	*   Jesse
	*   Sharon
	*   Anne

效果:

*   Jesse
*   Sharon
*   Anne

等同于：

	+   Jesse
	+   Sharon
	+   Anne

效果:

+   Jesse
+   Sharon
+   Anne

等同于：

	-   Jesse
	-   Sharon
	-   Anne

效果:

-   Jesse
-   Sharon
-   Anne

- - -

#####有序列表则使用数字接着一个英文句点

	1.  Jesse
	2.  Sharon
	3.  Anne

效果:

1.  Jesse
2.  Sharon
3.  Anne

等同于:

	1. Jesse
	1. Sharon
	1. Anne

效果:

1. Jesse
1. Sharon
1. Anne

等同于:

	4. Jesse
	2. Sharon
	8. Anne

效果:

4. Jesse
2. Sharon
8. Anne

- - -

{% include advertisement_content.html %}

### 分割线
		* * *

		***

		*****

		- - -

		---------------------------------------

效果:

test1
* * *
test2
***
test3
- - -
testend

- - -

### 强调

		*single asterisks*

		_single underscores_

		**double asterisks**

		__double underscores__

效果:

*single asterisks*

_single underscores_

**double asterisks**

__double underscores__
