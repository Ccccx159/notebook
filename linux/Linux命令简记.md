# Linux命令简记

## tar

常用选项：
+ -c: 建立压缩档案
+ -x：解压
+ -t：查看内容
+ -r：向压缩归档文件末尾追加文件
+ -u：更新原压缩包中的文件
+ -f: 使用档案名字，这个参数是最后一个参数，后面只能接档案名

可选参数：
+ -z：有gzip属性的
+ -j：有bz2属性的
+ -Z：有compress属性的
+ -v：显示所有过程
+ -O：将文件解开到标准输出

### 压缩、解压

~~~bash
# 压缩 file.tar.gz：压缩档案名称；file：待压缩目录
tar -czvf file.tar.gz ./file

# 解压
tar -xzvf file.tar.gz
~~~

### 加密压缩、解压

~~~bash
# 加密压缩
# 使用 --exclude 选项排除指定目录
tar -czvf - --exclue="file/exclude_dir" ./file | openssl enc -e -des3 -salt -k P@ssw0rd | dd of=file.tar.gz.des3

# 解压
dd if=file.tar.gz.des3 | openssl enc -d -des3 -salt -k P@ssw0rd | tar -xzvf -
~~~

### 分卷加密压缩、解压

~~~bash
# 分卷加密压缩
# split 参数
# -b, --bytes=SIZE, 指定每个分割文件的大小，单位有K, M, G, P等
# -d, --numeric-suffixes, 指定分割文件的后缀为数字
# -a, --suffix-length=N, 指定分割文件数字后缀的长度，如果-a 1，则后缀为*.1，*.2；如果-a 2，则后缀为*.01，*.02
# -c, --line-bytes=SIZE, 指定每行最大的字节数
# -l, --lines=NUMBER, 指定每个文件最大的行数
tar -czvf - --exclue="file/exclude_dir" ./file | openssl enc -e -des3 -salt -k P@ssw0rd | split -b 200m -d -a 2 - file.tar.gz.des3.

# 解压
cat file.tar.gz.des3.* | openssl enc -d -des3 -salt -k P@ssw0rd | tar -zxvf -
~~~

### FAQ

1. --exclude参数报错
	
	~~~bash
	tar: The following options were used after any non-optional arguments in archive create or update mode.  These options are positional and affect only arguments that follow them.  Please, rearrange them properly.
	~~~
	这是因为不同版本的tar，`--exclude`选项添加的位置存在差异。当tar的版本小于1.30时，该选项可以放在待压缩目录之后；若tar版本为1.30时，`--exclude`需要添加在待压缩目录之前。如下所示
	~~~bash
	# tar --version tar (GNU tar) 1.30
	tar -czvf file.tar.gz --exclude="file/exclude_dir" ./file
	
	# tar --version tar (GNU tar) 1.26
	tar -czvf file.tar.gz ./file --exclude="file/exclude_dir"
	~~~

## grep

常用选项：
+ -c 或 --count : 计算符合样式的列数。
+ -h --no-filename : 在显示符合范本样式的那一列之前，不标示该列所属的文件名称。 
+ -H --with-filename : 在显示符合范本样式的那一列之前，标示该列的文件名称。
+ -i 或 --ignore-case : 忽略字符大小写的差别。
+ -E 或 --extended-regexp : 将样式为延伸的正则表达式来使用。
+ -o 或 --only-matching : 只显示匹配PATTERN 部分。
+ -v 或 --invert-match : 显示不包含匹配文本的所有行。
+ -V 或 --version : 显示版本信息。
+ --include : 搜索指定的文件
+ --exclude : 搜索结果中排除指定文件
+ --exclude-from \<fileList\>: 在搜索结果中排除fileList中的文件，一行一个“pattern”

### 多条件与（and），或（or）查询

~~~bash
# 与（and）
grep 'pattern1' file | grep 'pattern2'

# 或（or），用法1
grep -E 'pattern1|pattern2' file
# 或（or），用法2
grep 'pattern1\|pattern2' file
# 或（or），用法3
egrep 'pattern1|pattern2' file

~~~

### 输出不包含指定内容的行，（非（NOT）查询）

~~~bash
# 输出不包含‘pattern’的行
grep -v 'pattern' file
~~~

### 使用实例

1. 根据可执行文件名称和运行参数，过滤输出进程id
	~~~bash
	ps ef | grep -i '${process_name}' | grep -i '${excute_param}' | grep -v 'grep' | awk '{print $1}'
	~~~

## dd

dd 命令用于读取、转换并输出数据。可从标准输入或文件中读取数据，根据指定的格式来转换数据，再输出到文件、设备或标准输出。

格式：
`dd [operand]`
`dd option`

常用参数：

+ bs=BYTES：同时设置输入/输出的块大小为BYTES字节
+ count=N：仅拷贝N个block块，块大小等于ibs指定的字节数
+ cbs=BYTES：一次转换BYTES个字节，即指定转换缓冲区大小
+ ibs=BYTES：一次读取BYTES个字节，即指定读取的一个块大小为BYTES字节
+ obs=BYTES：一次输出BYTES个字节，即指定输出的一个块大小为BYTES字节
+ if=FILE：输入文件名，默认为标准输入
+ iflag=FLAGS
+ of=FILE：输出文件名，默认为标准输出
+ oflag=FLAGS
+ skip=N：从输入文件开头跳过N个block块后开始复制
+ seek=N：从输出文件开头跳过N个block块后开始复制
+ status=LEVEL：打印到stderr的信息级别
	+ 'node'：    抑制错误消息之外的所有内容、
	+ 'noxfer'：  抑制最终的传输统计数据
	+ 'progress'：显示定期传输统计信息
+ conv=\<CONVS KEY WORDS\>
	+ lcase：把大写字符转换为小写字符
	+ ucase：把小写字符转换为大写字符
	+ noerror：出错时不停止
	+ notrunc：不截断输出文件

### 与管道符（|）配合读取/输出文件

1. 读取文件
	~~~bash
	# 配合 grep 查找指定内容
	dd if=./testfile status=none | grep 'HelloWord'
	~~~
	![](https://user-images.githubusercontent.com/35327600/212609068-f83073cf-06b9-4c73-ad8b-55af99d80d98.png)

2. 输出文件
	~~~bash
	# 将docker image信息写入指定文件
	docker image ls | dd of=./docker_image_info.txt status=none
	~~~
	![](https://user-images.githubusercontent.com/35327600/212609286-2525b883-4bfc-4ac9-a804-09f720f3828e.png)

### 备份磁盘并恢复

~~~bash
# 备份
# SATA硬盘被挂载在/dev/sda，将该SATA硬盘备份到sda.img中
dd if=/dev/sda of=/root/sda.img

# 恢复
# /dev/sda 硬盘出现故障时，将备份的sda.img恢复到指定的sdb盘中去
dd if=/root/sda.img of=/dev/sdb

# 复制完整磁盘环境
dd if=/dev/sda of=/dev/sdc
~~~

### 压缩备份

~~~bash
# 备份
dd if=/dev/sda | gzip > /root/sda.img.gz

# 恢复
gzip -dc /root/sda.img.gz | dd of=/dev/sdc
~~~

### 备份磁盘MBR表

~~~bash
# 一块磁盘的第一个扇区的 512 个字节所存储的正是这块磁盘的 MBR 信息，我们尝试用 dd 命令备份 MBR：
dd if=/dev/sda of=/root/sda.mbr.img count=1 bs=512

# 恢复
dd if=/root/sda.mbr.img of=/dev/sda
~~~

### 简单的磁盘读写性能测试

通过 `/dev/null` 和 `/dev/zero` 完成读写性能测试
+ `/dev/null`，也叫空设备，小名“无底洞”。任何写入它的数据都会被无情抛弃。
+ `/dev/zero`，可以产生连续不断的 null 的流（二进制的零流），用于向设备或文件写入 null 数据，一般用它来对设备或文件进行初始化。

~~~bash
# 向磁盘上写入一个大小为1Gb的大文件, 通过计算该命令执行时间，判断磁盘写性能
dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file

# 读取一个刚才生成的1Gb的文件，通过计算命令执行时间，判断磁盘读取性能
dd if=/root/1Gb.file bs=64k of=/dev/null
~~~

## sed



## awk

