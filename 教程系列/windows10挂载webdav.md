# windows10挂载webdav

## 一、简介

当前市面上大部分的网盘，可以挂载到 "**Alist**" 中。Alist 又支持 webdav协议。这就意味着通过 Alist 的 webdav 服务，我们可以直接将网盘挂载到本地，类似于本地磁盘一样读写网盘中文件。

但是在本地挂载的过程中，无论是添加网络位置，还是映射网络驱动器，都会出现文件路径不对、网络错误无法访问等错误，如下所示：

![](https://user-images.githubusercontent.com/35327600/215680157-9de51400-15cc-4d22-9d01-5e6bf8411e8f.png)

![](https://user-images.githubusercontent.com/35327600/215675986-ba71818f-88cb-4151-86b9-aae6ee69aa22.png)

本文将简单介绍如何在Windows环境下挂载本地webdav。

## 二、问题原因

导致简介中的问题其实非常简单。<mark>windows 默认的 WebClient 服务仅支持 https 协议</mark>，而本地搭建的 webdav 服务和链接都是基于 http 协议的，因此才造成了挂载失败的情况。

对于部分高手来说，将 webdav 服务转换为 https 协议必然是更安全，更好的选择。但是对于部分仅内网挂载访问，安全性需求较低的朋友来说，升级 https 的代价可能有些高昂，因此使 windows 自带的 WebClient 支持 http，可能是更快捷方便的选择。

### 三、设置 WebClient，允许 http 链接挂载

步骤1：
按下 "**windows徽标键**" + "**R**"，打开运行窗口，输入`regedit`，点击确定后，打开注册表编辑器窗口。

步骤2：
将路径定位到以下路径：`计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WebClient\Parameters`。双击右侧界面中的 `BasicAuthLevel` 条目，将数值数据修改为“2”，点击确定后关闭注册表编辑器。

![](https://user-images.githubusercontent.com/35327600/215690501-29b6636a-a17b-4fcd-a871-f3ddf031dfef.png)

步骤3：
按下 "**windows徽标键**" + "**R**"，打开运行窗口，输入`services.msc`，点击确定后，打开“服务”界面。找到 "**WebClient**"
服务，右键点击打开选项菜单，选择重新启动，稍等几秒，待完成后，关闭“服务”界面。

![](https://user-images.githubusercontent.com/35327600/215691793-fc3e4385-f6e4-47ed-bd8e-7a00d3a7cf4f.png)

完成上述三个步骤后，WebClient 服务已经允许使用 http 协议进行挂载。

## 四、挂载测试

1. 映射网络驱动器
	![](https://user-images.githubusercontent.com/35327600/215694498-777dba03-505a-4922-9485-ba99c0eb5809.png)
	![](https://user-images.githubusercontent.com/35327600/215694514-24f0ddf9-f2b2-4059-8e7a-107b216565d9.png)

2. 添加网络位置
	![](https://user-images.githubusercontent.com/35327600/215695212-f220eeb9-23f0-4500-987d-dea5146898a4.png)
	![](https://user-images.githubusercontent.com/35327600/215695227-595b63d6-c31e-4d06-83da-98179b01fdb1.png)

可以看到，在修改注册表后，映射网络驱动器和添加网络位置，都能正确访问 webdav 服务了。