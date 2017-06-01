---
layout: post
title: 使用Python在Markdown插入图片并自动获取链接
---

{{ page.title }}
================

# 前言
以前写博客都是使用网站的在线编辑器，这种开着网页写东西的感觉，真是。。反正我更喜欢在自己的PC上写好Markdown，然后直接在网站上导入。现在基本大一些的技术网站都可以支持从本地导入Markdown。但是Markdown的插入图片有点麻烦，有些网站比如简书的在线Markdown编辑器支持良好，而如CSDN，如果我使用本地图片位置链接，虽然本地可以插入图片，但是在线导入后显然不能直接获取我的本地资源，因此贴本地链接无效。所以我们一般采取的做法是：

 * 1.找到需要插入的图片，修改文件名以区分
 * 2.上传图片到某个图床，我用的是七牛云
 * 3.复制图床上的图片的链接；然后用markdown格式插入图片
 
操作不复杂，但是如果次次这么干，每次都得网页登陆七牛云。然后进空间，再上传，找到该图片所在位置。一旦插入的图片多了，就感觉很烦。

# 解决
好，现在借鉴了一个童鞋的方法，稍微对其脚本做了修改，先贴上人家的github源地址：
[https://github.com/wuchangfeng/markdown-helper](https://github.com/wuchangfeng/markdown-helper)

因为Python语言作为脚本使用非常强大，因此该段源代码以Python写成，首先你的本地需要先安装Python。Python是不向下兼容的，而且Python2和Python3有语法、模块等的一些区别。该脚本亲测同时兼容Python2.x和3.x，没有问题

	#! /usr/bin/python
	#-*- coding: utf-8 -*-
	
	from qiniu import Auth, put_file, etag, urlsafe_base64_encode
	import qiniu.config
	from qiniu import BucketManager
	import sys,time
	import os
	import msvcrt
	import datetime
	import subprocess
	import time
	
	
	# you will get md_url in this file
	result_file = "ss.txt"  
	
	if os.path.exists(result_file):
	    os.remove(result_file)
	os.chdir(sys.path[0])
	
	
	# you need get yours msg here
	access_key = ""
	secret_key =  ""
	bucket_name =  ""    # 填入你的七牛空间名称
	bucket_url =  ""   # 填入你的域名地址
	md_url_result = "md_url.txt"
	
	img_suffix = ["jpg", "jpeg", "png", "bmp", "gif"]
	
	def upload_img(bucket_name,file_name,file_path):
	    # generate token
	    token = q.upload_token(bucket_name, file_name, 3600)
	    info = put_file(token, file_name, file_path)
	    # delete local imgFile
	    # os.remove(file_path)
	    return
	
	def get_img_url(bucket_url,file_name):
	    # ?imageMogr2/thumbnail/!65p
	    # file_names = file_name + '?imageMogr2/thumbnail/!75p'
	    file_names = file_name + '?'+time.time()
	    img_url = 'http://%s/%s' % (bucket_url,file_names)
	    # generate md_url
	    md_url = "![%s](%s)\n" % (file_name, img_url)
	    return md_url
	
	
	def save_to_txt(bucket_url,file_name):
	    url_before_save = get_img_url(bucket_url,file_name)
	    # save to clipBoard
	    addToClipBoard(url_before_save)
	    # save md_url to txt
	    with open(md_url_result, "a") as f:
	        f.write(url_before_save)
	    return
	
	# save to clipboard
	def addToClipBoard(text):
		command = 'echo ' + text.strip() + '| clip'
		os.system(command)
	
	if __name__ == '__main__':
	    q = Auth(access_key, secret_key)
	    bucket = BucketManager(q)
	    imgs = sys.argv[1:]
	    for img in imgs:
	    	# name for img with local time 
	        up_filename = os.path.split(img)[1]
	        upload_img(bucket_name,up_filename,img)
	save_to_txt(bucket_url,up_filename)


![01.PNG](http://om2doplmh.bkt.clouddn.com/01.PNG?20170312_195042)

首次安装Python时，在安装选项时将上图图所示选项勾选上，这样安装完毕后会自动为我们添加环境变量，在任意目录都可以使用dos命令运行python指令了:

![02.PNG](http://om2doplmh.bkt.clouddn.com/02.PNG?20170312_195341)


因为是引入了七牛云的SDK，第一次使用时要将sdk集成进来，cmd命令行：

*>>> pip install qiniu*

在原先的脚本基础上做了修改（可见注释的代码，以及新增的）。图片上传并复制链接完毕后不在当前位置删除，而是保留。每次上传的图片名以原有文件名加上操作时的时间字符串加以区分。图片上传到七牛云后生成的七牛云存储的链接，既存在了剪贴板里，可以直接ctrl+v到Markdown文档上，同时在当前目录下创建一个"md_url.txt"并将每次生成的链接存储在其中。

# 使用方法
将要获取链接的本地图片拖拽到脚本文件.py上，命令行窗口一闪而过，具体消失的时间依图片上传的速度而定。这时候图片已经成功上传到七牛云空间并获取链接，如下动图所示：

![1.gif](http://om2doplmh.bkt.clouddn.com/1.gif?20170302_141013)

谢谢前人的代码和思路。该脚本同时支持["jpg", "jpeg", "png", "bmp", "gif"]格式文件的上传。上面的gif链接就是使用该脚本生成的。这样可以大大加快我们在诸如CSDN等技术网站上写博客的速度，是不是很方便呢？

# 补充！！
在win7以及升级激活后的win10系统中，默认情况下有可能我们无法拖放一个文件给python脚本让其去处理这个文件的，这是因为windows认为python脚本不是一个合法的可拖放的目的对象（drop target）。网上已经有人给出了解决办法，我们只需在注册表中配置一下即可，具体做法如下

新建.reg文件，文件名随意，内容为：

    Windows Registry Editor Version 5.00

	[HKEY_CLASSES_ROOT\Python.File\shellex\DropHandler]
	@="{60254CA5-953B-11CF-8C96-00AA00B8708C}"

保持后执行该注册表文件即可
