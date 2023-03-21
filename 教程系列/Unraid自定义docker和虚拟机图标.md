# Unraid自定义docker和虚拟机图标

## 一、自定义docker图标

在 Unraid 中可以通过自带的应用市场安装 docker 容器，但是也有部分docker并未上架市场，需要通过 “ADD CONTAINER” 按钮手动安装，这样的 docker 容器并没有自带图标。还有一部分即使是在应用市场安装的 docker 容器本身有图标，但是由于国内的网络环境原因，导致图标链接不可访问，最终在 docker 界面中显示一个灰底的问号。

这里用一个手动安装的 python:alpine3.16 的容器作为示例进行说明：
![](https://user-images.githubusercontent.com/35327600/220238709-7d4f5fff-c5f6-4955-a2be-b84d23658354.png)

下面介绍两种方式来自定义设置 Docker 容器的图标。

### 1.1、方法一：直接修改 Docker 容器配置

无论是从应用市场安装的 docker 容器还是手动创建的，都会在安装前展示容器的配置界面，两种安装方式的差别仅仅就是应用市场的 docker 容器会将大部分的配置参数预先填充，避免了手动配置填充的过程，为部分经验不足的新手提供了相当的便利。

一般来说，配置界面只展示了基础配置，我们点击右上角的 "BASIC VIEW" 按钮，打开高级配置 "ADVANCED VIEW":
![](https://user-images.githubusercontent.com/35327600/220241255-d0f8e445-8d3d-4de6-929a-01719f8c7d87.png)

打开后可见新增了许多配置项：
![](https://user-images.githubusercontent.com/35327600/220242308-ad3b6a1f-621c-4fc7-b13f-5e5d56e45ec3.png)

可以看到新增的配置项中，有一项名为 "Icon URL" 的配置，这里填充的就是当前 docker 容器的图标链接。我们在网上找一个 Python 的图标，并复制其图片地址进行配置填充：
python图标链接：`https://github.com/walkxcode/dashboard-icons/blob/main/png/python.png?raw=true`
![](https://user-images.githubusercontent.com/35327600/220243495-3daabdd1-79bf-42d9-a819-375c763719de.png)

配置完成后，点击 "APPLY" 按钮更新应用配置，待完成后在 Docker 界面就能看到图标由原本的问号，变为刚在填充的链接所展示的 python 图标了。
![](https://user-images.githubusercontent.com/35327600/220243941-f6dc6ec8-7df4-4cb9-a83b-39ec20738bdf.png)

方法一操作简单，直接在 WebUI 中进行配置即可。但是也有朋友想用本地自己制作的个性化图标，一种方法就是将图标上传到图床，然后使用图床链接进行配置。如果是具有个人版权的图标，不想上传到公共图床或者网络上，同时也不想自建图床，因为自建图床的学习成本相对较高，那么有没有其他办法进行修改配置呢？请看方法二——本地存储图标配置。

### 1.2、方法二：本地存储图标图片并配置（需使用命令行操作，不会的朋友请尽量使用方法一）

<mark>再次重申一下，方法二需要使用命令行操作，不会的朋友请尽量使用方法一，避免对系统本身造成无法修复的损害</mark>。

+ 首先我们先将图标下载到本地（因为我本地没有 Python 的图标，所以此处需要下载，如果本地已经有对应图标，就不需要下载），并存放到 unRAID 的任意共享目录中。
+ 将图片名称修改为以下样式 `<docker 容器名>-icon.png` 。例如此处使用命令 `mv python.png python-icon.png` 即可完成重命名。
	这里我直接采用的 unRAID 后台命令行进行操作，也可以直接在windows下进行重命名
	![](https://user-images.githubusercontent.com/35327600/220259348-6b70c342-98e6-4d57-bade-6e6ee4111d04.png)
+ 然后我们通过命令行将重命名后的图标文件拷贝到这个目录：`/var/lib/docker/unraid/images` 。
	~~~ shell
	cp python-icon.png /var/lib/docker/unraid/images;
	~~~
	其实在这个目录下，使用 ls 命令就能发现，所有 docker 容器的图标都被存放在这个目录底下。
	此时 docker 的图标还未生效，刷新界面后仍然显示的是灰底的问号。
+ 下一步，我们需要去修改 docker 容器的配置。和**方法一**一样，打开高级配置，找到 "Icon URL" 配置项，但是此时<font color=red>不需要填充真实的链接，随意填入内容即可</font>。这是为了告诉 unRAID 系统，这个 docker 容器是有图标的，你得给我显示出来。
	但是为什么可以填充随意内容呢？结合上一步中在目录 `/var/lib/docker/unraid/images` ，我们不难猜到，unRAID 只是根据 "Icon URL" 中的链接去下载图标到特定目录，然后根据 docker 容器的命名和图标文件的命名进行匹配和展示的。由于我们自行将图标重命名并且存放到了指定位置，所以 unRAID 系统自然能进行匹配和展示了。
	
	![](https://user-images.githubusercontent.com/35327600/220262561-88362131-b355-498c-8193-899e7bc8564d.png)
	
	填充完成后，还是老样子，点击下方 “APPLY” 按钮，此时你会发现图标已经变成你自定义的样子了。
	
	![](https://user-images.githubusercontent.com/35327600/220262946-6d53e1b0-5197-4e09-b79e-a4a96aa2ccb5.png)
	
	

## 二、自定义虚拟机图标

unRAID 提供的虚拟机图标类型较少，可能无法满足部分朋友的需要，此时也可通过自定义的方式进行配置。配置方法十分简单，下面一起来看下。

注意：虚拟机的配置修改需要在虚拟机处于关机状态下进行。

1. 第一步，打开虚拟机配置编辑，点击右上角的 “FORM VIEW”，打开 "XML VIEW" 模式
	![](https://user-images.githubusercontent.com/35327600/220265145-e9d995dd-acd7-46fa-b717-b205d76de807.png)
2. 按下键盘的 `Ctrl` + `F` 键，在弹出的搜索框中输入 "icon"，并回车进行搜索，找到 XML 文件中对应的字样
	![](https://user-images.githubusercontent.com/35327600/220266149-ece8dd67-c4a1-4b5f-9111-a50b7999b615.png)
3. 将下划线部分 `icon="/mnt/user/domains/DSM7.1/synology_icon.png"` 双引号中的内容，修改为你自定义虚拟机图标的绝对路径。图标存放位置不限。
	> 图标文件的绝对路径，以存在位置为 unRAID 上创建的共享路径为例，可以参考这样修改：/mnt/user/<共享路径>。将尖括号内的路径修改为图标在共享目录下的路径即可。以上文为例，我的共享目录是 domains，并在 domains 下创建了 DSM7.1 的子目录，图标存放在子目录中，因此绝对路径就是：/mnt/user/domains/DSM7.1/synology_icon.png
4. 点击下方 "UPDATE" 按钮后即可生效
	![](https://user-images.githubusercontent.com/35327600/220266951-c1cf9a8d-e093-4de1-a11d-f7cf9434d1ac.png)

