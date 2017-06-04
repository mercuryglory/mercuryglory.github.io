---
layout: post
title: 使用Jekyll搭建自己的博客
---

{{ page.title }}
================

# 介绍
Github Pages是可以托管在Github上的静态网页。用户可以使用它提供的模板，经过Jekyll的再加工后上传到服务器。Jekyll是一款Blog生成器，是Github官方使用的静态站点生成器，不需要数据库的支持。简单的说，利用这两者搭建自己的小型的博客网站（不包含较复杂的功能，数据量不大）是非常方便的，你可以将其看作是一个普通的仓库，你在本地所作的修改提交上去以后，github将会根据Jekyll规范生成并更新你的博客网页。

# 搭建环境
## 创建自己的Github Pages
官方教程，可以很清楚的指引你如何创建Github Pages:[https://pages.github.com](https://pages.github.com)

阮一峰写的：[链接](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html) 对于Jekyll的使用有简要的介绍，但是对于如何搭建环境没有介绍。而Jekyll是基于Ruby的开源插件，在windows下搭建Ruby简直无力吐槽。

## windows10上安装Jekyll
### 下载安装Chocolatery
Chocolatery是一款专为Windows系统开发的、基于NuGet的包管理器工具，类似于Node.js的npm,MacOS的brew，Ubuntu的apt-get，简称为choco。

* 以管理员身份运行powershell:
![使用Jekyll搭建自己的博客_1.png](http://om2doplmh.bkt.clouddn.com/使用Jekyll搭建自己的博客_1.png)

* 命令行窗口执行 `iwr https://chocolatey.org/install.ps1 | iex` ，会报出以下错误，需要更改执行策略为RemoteSigned，下载成功。
![使用Jekyll搭建自己的博客_2.PNG](http://om2doplmh.bkt.clouddn.com/使用Jekyll搭建自己的博客_2.PNG)

### 用choco下载安装ruby，通过RubyGems下载安装Jekyll。
![使用Jekyll搭建自己的博客_3.png](http://om2doplmh.bkt.clouddn.com/使用Jekyll搭建自己的博客_3.png)
命令行窗口下输入 `jekyll -v` 可查看当前jekyll的版本，如能正确查看则下载安装成功。我当前使用的是 3.4.2 

# 使用Jekyll搭建页面
如果照前面的步骤已经正确的创建了Github Pages的话，你的仓库名，同时也是你的博客站点名，应该就是*用户名.github.io*。你可以将仓库的地址clone到本地，按Jekyll的组织架构填充内容，并再次提交并push。或者你可以按照下面的方式直接在本地创建项目，最后再添加到远程仓库。
## Jekyll语法特性
我没有掌握全jekyll的语法特性，关于它的介绍这篇文章说的比较清楚：
[http://www.jianshu.com/p/c04475ba80e4#](http://www.jianshu.com/p/c04475ba80e4#)

## 基本目录结构
命令行或者Git Bash下运行 `jekyll new article` 就可以创建出一个最基本的Jekyll模板，其目录结构如图：

![使用Jekyll搭建自己的博客_4.png](http://om2doplmh.bkt.clouddn.com/使用Jekyll搭建自己的博客_4.png)

Jekyll经过了多次版本迭代，对于默认创建的目录也有较大的变化：

* _posts:用于存放博客的文件夹，一般采用markdown语法写成
* _coinfig.ym：jekyll的配置文件，里面可以定义很多配置参数，具体可查看 [官网](http://jekyllrb.com/docs/configuration/)
* about.md:可以看作 content 填充模板，build后生成html文件，相当于一个页面
* index.md:同上

在命令行中直接运行 `jekyll serve`，这样就在本地开启了一个监听端口4000的服务器。浏览器中直接运行`localhost:4000`就可以查看效果：

![使用Jekyll搭建自己的博客_5.png](http://om2doplmh.bkt.clouddn.com/使用Jekyll搭建自己的博客_5.png)

本地部署成功，或者Build成功后，会在项目目录下生成 _sites 文件夹

## 目录结构解析
jekyll构造的目录结构比较小巧，但也需要花点时间了解。
对照本地运行的效果，再去看默认生成的文件。比如 **index.md**:
![使用Jekyll搭建自己的博客_6.png](http://om2doplmh.bkt.clouddn.com/使用Jekyll搭建自己的博客_6.png)


根据前面介绍的语法特性，可以看出这个文件只是充当了入口，相当于网站的首页。引用的模板layout是**home**。

同理查看about.md，发现引用的模板是**page**，再看_posts文件下的md文档，引用的模板是**post**。

全局搜索发现这几个模板html都存放在jekyll默认的模板库下面，像我的本机存放地址就是
**C:\tools\ruby23\lib\ruby\gems\2.3.0\gems\minima-2.1.0\_layouts**

打开三个文件，发现引用的都是同级目录下的default，再去看这个文件，内部代码如下:

{% raw %}

	<!DOCTYPE html>
	<html lang="{{ page.lang | default: site.lang | default: "en" }}">
	
		{% include head.html %}
	
		<body>
	
			{% include header.html %}
	
			<main class="page-content" aria-label="Content">
	      	<div class="wrapper">
	        	{{ content }}
	      	</div>
			</main>
	
			{% include footer.html %}
	
		</body>
	
	</html>
{% endraw %}

可以看出，jekyll的结构除了采用html常用的模板方式，还有解耦的思想，将可以公用的部分完全提取出来。default定义页面的基本样式，其中的head.html、header.html、footer.html都存放在上级目录的_includes文件夹下，分别负责网页信息、网页头部、网页尾部。中间的 content 则由引用该模板的html文件填充。header.html和footer.html中的site.title和site.email存放在项目目录下的_config.yml配置文件中，site.pages则表示所有引用page作为模板的页面，也就是.md，比如about.md。md文档的存放目录，使用post.html作为模板。

## 样式
default模板引用的是上级目录下的assets文件夹下的main.css。你也可以自定义样式，这需要一些html和css基础。

## 更新博客
按照已经定义好的方式，用Markdown写自己的东西。标准的命名格式是**YYYY-MM-dd-title.md**

除了文章主体用markdown语法，在文章顶部还应添加引用的布局以及网页的标题，标准格式如下:

    ---
	layout: "要应用的模板布局"
	title:  "标题（可以不同于文档名，会作为网页标题使用）"
	date:   2017-05-31 15:49:52 +0800
	categories: "所属目录，可不填"
	---
	正文

# 注意！
我在使用Jekyll搭建博客中遇到的一个坑。就是在markdown文档中插入上面提到的default.html代码的时候，一直报这个错：
![使用Jekyll搭建自己的博客_8.png](http://om2doplmh.bkt.clouddn.com/使用Jekyll搭建自己的博客_8.png)
检查后发现是Jekyll在解析markdown生成页面的时候，将作为代码引用的Liquid语句也解析了，自然在该项目的_includes文件夹下不会找到head.html这个文件了。我是通过这篇[博客](http://markzhang.cn/blog/2013/11/07/embed-liquid-codes-in-blog/)解决问题的，这里使用的方法是使用raw标签将代码包起来，不让jekyll将其当作liquid解析。markdown要真正使用jekyll生成网页，自然需要按照jekyll的语法糖来。

# 存在的问题：
文章的左右间距，过长的代码超出了背景的边界。默认的目录结构不够直观。