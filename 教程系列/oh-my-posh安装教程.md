# oh-my-posh安装教程

## 简介

+ github: [https://github.com/JanDeDobbeleer/oh-my-posh](https://github.com/JanDeDobbeleer/oh-my-posh)
+ docs:   [https://ohmyposh.dev/](https://ohmyposh.dev/)

docs中有详细的安装和使用说明，下文中有不明白的可以移步官方文档自行学习使用~

## 安装

下面以新安装的x86_64的debian环境进行举例

`Linux debian 5.10.0-19-amd64 #1 SMP Debian 5.10.149-2 (2022-10-21) x86_64 GNU/Linux`

针对内网和非root用户进行安装，避免污染其他用户的使用环境

~~~bash
#!/bin/bash

# 获取posh可执行文件，内网环境可以直接从github中下载后，拷贝到用户下指定目录
# 当前命令中下载的可执行文件是基于amd64架构的，根据自己的系统架构选择下载
mkdir -p ~/oh-my-posh
wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/posh-linux-amd64 -O ~/oh-my-posh/posh;
chmod +x ~/oh-my-posh/posh;

# 下载主题
mkdir -p ~/oh-my-posh/.poshthemes;
wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/themes.zip -O ~/oh-my-posh/.poshthemes/themes.zip;
unzip -q ~/oh-my-posh/.poshthemes/themes.zip -d ~/oh-my-posh/.poshthemes;
rm ~/oh-my-posh/.poshthemes/themes.zip -vf;
chmod u+rw ~/oh-my-posh/.poshthemes/*.omp.*;

# 向~/.profile，或者~/.bashrc，或者~/.bash_profile中写入posh初始化命令，下面命令中以~/.profile为例，并指定主题 montys.omp.json
sed -i '$a eval "$(~/oh-my-posh/posh init bash --config ~/oh-my-posh/.poshthemes/montys.omp.json)"' ~/.profile;

# 重新启用bash
source ~/.profile;

# 若安装了zsh，则最后两行命令用下面两行替代
# 向~/.zshrc中写入posh初始化命令，并指定主题 montys.omp.json
#sed -i '$a eval "$(~/oh-my-posh/posh init zsh --config ~/oh-my-posh/.poshthemes/montys.omp.json)"' ~/.zshrc;
# 应用新的.zshrc文件
#exec zsh;

~~~

## 示例动画

![](https://user-images.githubusercontent.com/35327600/211761346-9d91a9ad-bc18-4128-8cf4-49a91ebfcb0e.gif)

## montys.omp.json主题微调

由于montys主题默认会在powerline中输出完整的路径，当路径层级较深时，powerline显示就太长了，因此参考文档进行了微调。

~~~diff
diff --git a/themes/montys.omp.json b/themes/montys.omp.json
index c7797e88..c8eeb6c4 100644
--- a/themes/montys.omp.json
+++ b/themes/montys.omp.json
@@ -17,10 +17,11 @@
           "foreground": "#ffffff",
           "powerline_symbol": "\ue0b0",
           "properties": {
-            "folder_icon": "\uf115",
+            "folder_icon": "\uf07c",
             "folder_separator_icon": "\\",
             "home_icon": "\uf7db",
-            "style": "full"
+            "style": "agnoster_short",
+            "max_depth": 3
           },
           "style": "powerline",
           "template": " <#000>\uf07b \uf553</> {{ .Path }} ",
~~~


效果对比：

before：

![](https://user-images.githubusercontent.com/35327600/211757664-172072e7-7c19-49e0-b900-7c6975471931.png)

after:

![](https://user-images.githubusercontent.com/35327600/211762075-d3a46f69-5bf8-4dfe-a15a-e39ff4ca9198.png)