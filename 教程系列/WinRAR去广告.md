---
title: WinRAR去广告
date: 2023-05-18 10:57:00
tags: 
- 原创
- WinRAR
- 去广告
categories:
- 折腾之路
---

# WinRAR 去广告。。。

搜索引默认提供的 winrar 官网由国内公司代理。。。免费版中的广告就是被这个国内代理商插入的。。。真的就是中国人专坑中国人。。。

直接去源网站 https://www.rarlab.com/ 下载正式发布的英文版或者繁体中文版，纯净无广告。。。

无语至极。。。

不想重新安装的，也可以使用以下方法：

1. 在 WinRAR 的安装目录下新建一个 txt 文件，并重命名为 "**rarreg.key**"
2. 用记事本，notepad++等软件打开 "**rarreg.key**"，并粘贴以下内容后保存即可
	~~~ shell
	RAR registration data  
	Federal Agency for Education  
	1000000 PC usage license  
	UID=b621cca9a84bc5deffbf  
	6412612250ffbf533df6db2dfe8ccc3aae5362c06d54762105357d  
	5e3b1489e751c76bf6e0640001014be50a52303fed29664b074145  
	7e567d04159ad8defc3fb6edf32831fd1966f72c21c0c53c02fbbb  
	2f91cfca671d9c482b11b8ac3281cb21378e85606494da349941fa  
	e9ee328f12dc73e90b6356b921fbfb8522d6562a6a4b97e8ef6c9f  
	fb866be1e3826b5aa126a4d2bfe9336ad63003fc0e71c307fc2c60  
	64416495d4c55a0cc82d402110498da970812063934815d81470829275
	~~~

完成上面两个步骤后，在打开 rar 文件就没有广告弹窗了。


参考：[《WinRAR去除 屏蔽广告弹窗方法》](https://blog.csdn.net/qq_39313596/article/details/85169627)
