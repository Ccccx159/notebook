# Unraid USB Drive Crack 教程

**<font color="red">！！！破解教程仅供学习参考，务必支持正版软件，构建良好的软件版权生态！！！</font>**

## 一、简介

由于Unraid OS使用硬盘阵列的形式构建存储池，能尽可能利用硬盘空间，并且可以通过创建校验盘保护数据，因此Unraid OS也不失为一个NAS系统的好选择。

但是因为试用版需要在联网环境下完成授权，导致处于内网环境的NAS机器无法使用。网上虽然有大量的开心版，不过大多是通过第三方提供的一个keyMaker.exe对USB的GUID生成一个key文件进行破解，总感觉不是很安全。

在爬了无数帖子后，终于在老毛子的一个论坛中找到了每个人都能轻松"**转正**"，且安全的方法！论坛连接放在文末，近期论坛中已经释出6.11版本。

## 二、准备工具

1. 一个U盘
2. 官方usb引导创建工具：**Unraid.USB.Creator.Win32-2.1.exe**（官网下载即可）
3. 一个可以使用gcc的编译环境（Debian、Ubuntu、Centos等，虚拟机，云主机都可以）（windos下其实也可以，但是环境配置相对麻烦，不如直接使用虚拟机创建前面提到的三个linux系统）
4. 一份源码构建密钥的源码：`https://github.com/mysll/unraid_test.git`

## 三、Crack 系统盘破解

### 3.1、制作系统盘

通过在官网下载的USB引导创建工具，制作官方系统盘即可，如下图所示：

![](https://user-images.githubusercontent.com/35327600/209258446-6d68256c-f8bf-4275-a102-13b0c288f0d3.png)

制作过程不在这里进行赘述了，B站大佬[司波图](https://space.bilibili.com/28457/channel/seriesdetail?sid=896368)的Unraid系列教程中介绍的很详细了，不了解的朋友可以移步观看。

这里有几个需要注意的点：
1. 由于服务器在国外，直接在线下载可能会很慢，或者没有速度，也可以单独下载官网中的zip包，然后在图中第一步中选择"Local Zip"；

2. 图中第二步"Select your USB Flash device"，注意是否是待制作的U盘，确认后将USB名称后面中括号内的一串数字记录下，如图所示：

   ![](https://user-images.githubusercontent.com/35327600/209281193-0a968f78-8add-4d06-97cb-c80b44517b2a.png)

   这是U盘的**GUID**，在后续破解中还需使用到。（如果忘记保存了，也没关系，重新打开这个制作工具就能再次看到了)

3. 点击“**Write**”后会提示当前选中U盘会被格式化，如果没选错的话，OK就行；

等待进度条满后，系统盘就制作完成了。<font color="blue">如果在制作系统盘时，选择了"Local Zip"，那么接下来还需要注意以下一点</font>:

> 打开U盘目录，找到根目录下“make_bootable.bat”文件，右键点击后，选择管理员模式运行

完成以上步骤后，暂时先不要拔下U盘，后续还需要对内部的文件进行操作。

### 3.2、制作密钥

制作密钥过程需要在linux环境中进行（如果你的windows系统也部署了gcc，那也可以在windows下操作）。利用windows自带的“Hyper-V”虚拟机搭建一个Ubuntu非常的简单方便，不会的朋友可自行百度教程。下面我以Ubuntu为例介绍密钥的制作过程。

1. 首先将准备工具中第4点提到的源码下载下来，依次执行以下两条命令：
    ~~~shell
    git clone https://github.com/mysll/unraid_test.git ./Unraid_test
    cd Unraid_test
    ~~~
    ![](https://user-images.githubusercontent.com/35327600/209284969-1ccd4d06-b0ad-4129-a746-3e7d487923ae.png)
    当然也可以不用以上命令，直接点击源码链接，去github网站上进行下载。（此处也有可能有网络问题，存在网络问题是，请百度关键词“github 下载失败”）

2. 进入源码目录后，执行命令 `gcc -fPIC -shared unraid.c -o BTRS.key`，会一点C语言或者C++在linux环境编译的话，就能明白这一步在做什么，不明白也没关系，无脑执行即可。
   <font color="red">此时会在当前目录下生成一个名为“**BTRS.key**”的文件，你猜的没错，这就是我们所需要的密钥文件</font>

3. 将第二步中生成的“**BTRS.key**”拷贝到U盘的“**config**”目录下

    ![](https://user-images.githubusercontent.com/35327600/209305975-9f6ec553-e639-4fd3-8170-6584aa525047.png)

4. 在config目录下找到一个名为“**go**”的文件，先将该文件备份一下，然后用以下内容替换原文件的内容

    ~~~shell
    #!/bin/bash
    # ---------修改以下三项内容，只需要修改等号右边内容，左边不要变更--------- #
    # GUID 将单引号内的内容替换成你自己U盘对应GUID
    export UNRAID_GUID='xxxx-xxxx-xxxx-xxxxxxxxxxxx'
    # NAME 随便填
    export UNRAID_NAME=unraid_test
    # 这是unix的一个时间戳，百度关键词“unix时间戳”，找一个转换网站，将当前时间转换为时间戳后填入，下面有示例
    export UNRAID_DATE=1658129986
    # -----------不要修改！！！不要修改！！！不要修改！！！------------------ #
    export UNRAID_VERSION=Pro
    LD_PRELOAD=/boot/config/BTRS.key /usr/local/sbin/emhttp &
    ~~~

  时间戳转换示例，[在线转换工具](https://tool.lu/timestamp/)：

  ![](https://user-images.githubusercontent.com/35327600/209291869-4c00d313-2e98-414c-b755-e2f48ab44e50.png)

此时可以拔下U盘，然后插到你的NAS上，愉快的使用啦~~~

当然了，还是希望有能力的朋友支持正版，毕竟这种开心版是否在后续使用中还是有所缺陷，例如强大的my_server就无法使用。更何况软件版权生态也是需要大家共同维护的。

## 四、Backup 系统盘备份

由于Unraid的系统文件完全被存储在U盘中，并且十分的轻量，因此备份变得非常方便。

在APP市场中下载“**User Scripts**”插件，通过定时执行备份脚本，实现定期备份系统盘的功能。

![](https://user-images.githubusercontent.com/35327600/209297717-10fcb3ff-b65c-48fe-a796-a7a419912c20.png)

![](https://user-images.githubusercontent.com/35327600/209298058-adb48d23-c188-4000-b24a-0619df42ebb2.png)

创建完成后点击脚本前的小齿轮，选择"**EDIT SCRIPT**"，将下方内容贴入，然后点击"**SAVE CHANGES**即可。

~~~shell
#!/bin/bash
CUR_TIME=$(date "+%Y%m%d%H%M%S")
BACKUP_FILE="unraid_flash_backup_${CUR_TIME}.tar.gz"
# 将等号后面的路径修改为你自己的备份路径
BACKUP_PATH="/your/back/up/path/"

tar -czvf ${BACKUP_PATH}${BACKUP_FILE} /boot;
~~~

保存后可以点击 "RUN SCRIPT" 测试一下备份文件是否成功创建。注意不可关闭运行后的弹窗，否则脚本会中止执行。

> 这里还有更进阶的玩法，可以结合阿里云的webdav和rclone挂载，将备份包上传至网盘中，并删除过期备份包，避免占用过多网盘空间，这个有需要的话，可以后面再出一篇详细教程。

## 五、Rescure 系统盘恢复

由于Unraid OS的完整系统都存在U盘中，因此当系统配置出现问题，导致无法进入引导，无法进入GUI界面时，并不会影响阵列！

因此系统盘的恢复，比备份更简单。只需要使用官方的USB制作工具创建一个官方的系统盘，然后将备份包中的config目录完全拷贝至当前的U盘内即可！是不是超级，非常，极其简单和方便！！！

<mark>当然还有一个注意点，如果U盘更换了，config目录中的"**go**"文件中的GUID也需要同步修改一下。千万别忘记啦！</mark>