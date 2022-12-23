# Unraid下虚拟DSM7.1，并开启相册人脸识别

**<font color="red">风险提示！！！请勿直接应用于生产环境或者单一数据存储环境，当前仅为测试版本！！！数据无价，请务必做好数据备份！！！</font>**

**<font color="red">风险提示！！！请勿直接应用于生产环境或者单一数据存储环境，当前仅为测试版本！！！数据无价，请务必做好数据备份！！！</font>**

**<font color="red">风险提示！！！请勿直接应用于生产环境或者单一数据存储环境，当前仅为测试版本！！！数据无价，请务必做好数据备份！！！</font>**

## 一、前言

由于主机换新，之前的旧设备闲置也是闲置，于是参考 [Unraid 教父——司波图](https://space.bilibili.com/28457/)一系列教程搭建了一台 Unraid 系统的 NAS。之前四盘位的 J3455 蜗牛星际也快满了，并且性能远远低于换代下来的 i5 8500，正好将内容迁移到 Unraid 上。但是群晖的相册套件说实话简单上手好用，还自带人脸识别，所以还是打算在 Unraid 上虚拟一个黑群晖，专门用于相册的备份和维护。废话不多说，下面开始干货。

## 二、配置

|        |  |
| :------- | -: |
| sys    |                                          Unraid 6.10.3 |
| cpu    |                   Intel® Core™ i5-8500 CPU @ 3.00GHz |
| 主板   |  Gigabyte Technology Co., Ltd. B360M AORUS Gaming 3-CF |
| 内存   |                                       DDR4 -  24G 普条 |
| 硬盘   | 希捷 ST18000NM013J_ZR56YQB9 - 18 TB<br />西数 SN570 -1TB |

基本配置如上，其中西数的 SN570 作为 cache 使用，无校验盘。“亡命之徒”本徒了。

**<font color="red">注意：如果需要添加校验盘，则校验盘容量必须大于等于阵列中单硬盘最大容量，以上配置就需要一个 18T 的硬盘作为校验盘。没有校验盘时，重要资料务必多地备份！</font>**

## 三、准备事项

**以下内容都是基于 DSM918+ 7.1.0 展开，其余设备型号或者版本因精力有限未做尝试！**

1. 准备一个 tinycore-redpill 的基础镜像（tinycore-redpill-uefi.v0.8.0.0.img）：[https://github.com/pocopico/tinycore-redpill](https://github.com/pocopico/tinycore-redpill)
2. 准备 DSM918+ 7.1.0patch 文件：[https://cndl.synology.cn/download/DSM/release/7.1/42661-1/DSM_DS918%2B_42661.pat](https://cndl.synology.cn/download/DSM/release/7.1/42661-1/DSM_DS918%2B_42661.pat)
3. 主板 bios 打开核显（安装过程中发现，核显被设置为自动，当有独显时，核显默认不开，会导致 Unraid 无法获取核显信息，**Intel GVT-g** 插件无法使用）
4. ssh 工具，如 xshell，putty，MobaXterm 等

## 四、创建虚拟 DSM 流程

<mark><font color="red">再次提醒！！！务必做好数据备份，在无需担心数据损失的前提下进行以下操作！！！</font></mark>

<mark><font color="red">再次提醒！！！务必做好数据备份，在无需担心数据损失的前提下进行以下操作！！！</font></mark>

<mark><font color="red">再次提醒！！！务必做好数据备份，在无需担心数据损失的前提下进行以下操作！！！</font></mark>

### 1、创建虚拟机

创建虚拟机的基本步骤请参考——[unRaid 下黑群晖，Freenas，OMV 的安装方法——司波图 UNRAID 陪玩教程 05](https://www.bilibili.com/video/BV1R7411s7gP?spm_id_from=333.999.0.0)。

最新的创建参数和大佬的有些差别：

1. Machine：Q35-6.2
2. BIOS：OVMF（tinycore 选择 UEFI 镜像）
3. USB Controller：3.0(qemu XHCI)
4. Primary vDisk 选择我们事先拷贝到 isos 目录下的镜像文件（tinycore-redpill-uefi.v0.8.0.0.img），并选择 USB 模式
5. 添加第二块 vDisk，此处设置为 sata 模式，其余按需设置即可
6. 网卡设置为 e1000
7. 取消勾选“Start VM after creation”

此时已基本完成虚拟机相关设置，如下图所示：

![](https://user-images.githubusercontent.com/35327600/180962087-4ce99534-f44e-4cf7-bb9e-26dc5148f248.png)

创建后，重新编辑虚拟机，打开 xml 模式，修改以下红框内划线处 `controller = "0"` 为 `controller = "1"`。

![](https://user-images.githubusercontent.com/35327600/180955218-2dc76d15-879c-4f12-b029-1f90cf0ca5b8.png)

本人在尝试了无数次卡重新安装 pat 的死循环后，最终在 xp 论坛上找到了解决方案。就是这个 sata disk 的 controller 索引错误导致无法找到 sata 磁盘控制信息，从而卡在安装 pat 文件错误的死循环中。原贴链接：[https://xpenology.com/forum/topic/63333-tutorial-install-dsm-71-on-unraid-6103/#comment-287607](https://xpenology.com/forum/topic/63333-tutorial-install-dsm-71-on-unraid-6103/#comment-287607)

整体安装 DSM 的流程也可参考原贴。

### 2、创建完整引导镜像

1. 开启虚拟机，并开启 VNC，看到如下界面：

    ![](https://user-images.githubusercontent.com/35327600/180959772-e0ea5062-238d-4e65-a9f0-db37e77e379e.png)

    按照图片中描述操作，获取当前虚拟机 ip 地址
2. 通过 ssh 工具进行连接虚拟机

    ```shell
    ssh tc@192.168.2.191
    <输入密码：P@ssw0rd>
    
    Connecting to 192.168.2.191:22...
    Connection established.
    To escape to local shell, press 'Ctrl+Alt+]'.
    
    WARNING! The remote SSH server rejected X11 forwarding request.
       ( '>')
      /) TC (\   Core is distributed with ABSOLUTELY NO WARRANTY.
     (/-_--_-\)           www.tinycorelinux.net
    
    tc@box:~$ 
    ```
3. 依次无脑执行以下命令，命令执行过程中会有部分内容需要手动确定，有 yY 输 y，无 yY 直接回车

    * `./rploader.sh update now`
    * `./rploader.sh fullupgrade now`
    * `./rploader.sh serialgen DS918+`
    * `./rploader.sh satamap now`
    * `./rploader.sh identifyusb now`
    * `./rploader.sh ext apollolake-7.1.0-42661 add https://raw.githubusercontent.com/pocopico/rp-ext/master/e1000/rpext-index.json`
    * `./rploader.sh build apollolake-7.1.0-42661`

    执行最后一个命令时，可能会有红色日志提示，内容如下：

    ```shell
    [!] Extension is already added (index exists at /home/tc/redpill-load/custom/extensions/pocopico.e1000/pocopico.e1000.json). For more info use "ext-manager.sh info pocopico.e1000"
    *** Process will exit ***
    ```

    该提示目前使用下来无影响，出现如下打印，表示镜像构建成功：

    ![](https://user-images.githubusercontent.com/35327600/180965299-4c8231d4-a541-4275-b167-0ba521330ab8.png)
4. 关闭虚拟机，打开 xml 编辑，查看之前修改的 `controller = "1"` 是否又变回默认值 `"0"`。如果发生改变，请再一次手动更改为 `"1"`，否则将无法正确安装 pat 文件。

### 3、创建 DSM7.1

启动虚拟机，并在 VNC 中手动选择 USB 引导。（起始界面还有一个 SATA 引导，未进行测试，喜欢折腾可以试试）

后续就和常规安装群晖一样，使用浏览器访问 https://finds.synology.com/，获取新安装的 DSM 信息，上传 pat 文件进行安装。

![](https://user-images.githubusercontent.com/35327600/180968642-5e580051-e9a7-4e82-be03-267eadf2e1b0.png)

恭喜，DSM 7.1 至此已完成完整的安装！接下来的基本操作就不在赘述了，按照指引正常处理就行。

## 五、开启相册人脸识别

按照上述流程创建的群晖是无法开启相册套件中的人脸识别功能，因为核显没有直通给群晖，导致群晖无法调用核显进行人脸识别。我们需要借助“**Intel GVT-g**”插件虚拟化核显，并配置给我们的群晖虚拟机。

1. 配置好群晖后，关闭虚拟机
2. 在 Unraid APP 市场中安装插件 **Intel GVT-g**
3. 在 PLUGINS 界面中打开 **Intel GVT-g** 配置
4. 根据当前虚拟显存的模式，分配给群晖虚拟机，确定后点击 “ASSIGN VM”

    ![](https://user-images.githubusercontent.com/35327600/180971454-d802c4c7-6050-452f-b954-d1bc4e8a3375.png)
5. 回到虚拟机的 xml 配置进行修改，安装时的 `controller = "1"` 还是需要注意的地方。其余部分按下图进行修改：

    ![](https://user-images.githubusercontent.com/35327600/180977569-0c9566b4-8ab8-4232-89d8-a39959387c5a.png)

    找到 xml 中新增的 \<hostdev\>，将其中的 `bus = '0x01' slot='0x00'` 修改为 `bus = '0x00' slot='0x02'`，这是因为虚拟化后核显的地址默认为 `0000:00:02.0`

    由于我们将虚拟化核显的总线（bus）和设备号（slot）修改了，和 xml 中部分原有配置产生冲突，因此需要将其余 `bus = '0x00' slot='0x02'` 所在的行删除，如下图所示：

    ![](https://user-images.githubusercontent.com/35327600/180977611-5185c30b-d8d6-43d6-94c8-b6dfc9714bfd.png)
6. 修改完成后，重启群晖虚拟机，在 `控制面板->终端机和SNMP` 中打开 ssh 功能。使用 ssh 工具进行连接

    ~~~shell
    输入：ls /dev/dri
    显示：card0  renderD128
	~~~

如果能正常显示上述内容，如果不是设备特殊，此时应该已经能进行人脸识别，我们测试一下

## 六、人脸识别功能测试

在套件中心搜索关键词“photo”，下载安装完成后打开，点击右上角用户图标，在设置中“启用个人空间人物相册”

![](https://user-images.githubusercontent.com/35327600/180982129-913020e6-c84d-4f68-9292-c982bc34b2c2.png)

找几张图上传，等一段时间后，查看一下人物相册，是否根据人脸识别自动创建了对应的相册，如果正确创建了，那么恭喜！

![](https://user-images.githubusercontent.com/35327600/181008438-89e5df3b-ec6e-4bad-ab56-56d63161810a.png)


## 七、参考链接

* pocopico 大佬的 github：[https://github.com/pocopico/tinycore-redpill](https://github.com/pocopico/tinycore-redpill)
* xpenology 论坛：[https://xpenology.com/forum/topic/63333-tutorial-install-dsm-71-on-unraid-6103/#comment-287607](https://xpenology.com/forum/topic/63333-tutorial-install-dsm-71-on-unraid-6103/#comment-287607)
* Jinlife 大佬博客：[https://blog.jinlife.com/index.php/archives/49/](https://blog.jinlife.com/index.php/archives/49/)
* 张大妈：[https://post.smzdm.com/p/a5dl2808/](https://post.smzdm.com/p/a5dl2808/)
* 司波图 B 站教程视频：[https://space.bilibili.com/28457/channel/seriesdetail?sid=896368](https://space.bilibili.com/28457/channel/seriesdetail?sid=896368)