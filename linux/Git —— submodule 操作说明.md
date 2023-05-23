---
title: Git —— submodule 操作说明
date: 2023-05-19 10:22:34
tags: 
- 原创
- Git
- submodule
categories:
- 程序员进阶
---

# Git —— submodule 操作说明

## 1. add 添加子模块
~~~ shell
# git add submodule -b master https://github.com/coolsnowwolf/lede.git ./lede
git add submodule -b <branch-name]> <git-repository-url> [local-path]
# 本地提交
git commit -m "add submodule xxxx"
# 推送到远程仓库
git push
~~~
## 2. checkout 子模块检出
~~~shell
# 有两种方式：
# 1. 使用 --recursive 参数，跟随主仓库递归 clone
git clone <your main repository url> --recursive # 此时 clone 下来的主项目会直接 clone 远程仓库中记录的 commit id 版本的子模块

# 2. 单独 checkout 子模块
git clone <your main repository url> # 不带 --recursive 递归参数时，submodule 无法被一起 clone 下来
git submodule update --init --recursive # 将 submodule 更新到远程仓库中记录的 commit id 版本
~~~

## 3. update 更新/切换子模块 commit id 和当前分支

这里存在一个较大的坑，默认检出的子模块并不属于任何分支，而是一个 "detached head" ，虽然可以提交更改，但是并没有本地分支跟踪提交的更改，这意味着<font color=red>下次更新子模块会丢失这些更改</font>。

因此在对子模块进行开发修改前，请先切换其所属分支和对应的 commit id。

~~~shell
# 默认添加的 submodule 的 commit id 是 add 时默认分支当前的一个 commit id，当子模块原始仓库更新后，期望切换到指定的 commit id 版本，或者像要切换分支
git pull
git submodule update # 更新本地仓库，避免出现冲突
cd <submodule dir>
git checkout <branch name> # 切换分支
git pull # 拉取新分支源码
git checkout <commit id> # 更新子模块版本

# 回到主仓库目录，提交子模块的引用版本修改
cd ..
git add . # 暂存 submodule 的引用版本修改
git commit -m "update submodule xxx from xxx to xxx" # 提交
git push # 推送到远程仓库

~~~

## 4. commit 提交子模块
~~~shell
git pull 
git submodule update # 确保提交前已将本地仓库更新到远程仓库最新版本，避免提交出现冲突
cd <submodulde dir>
git add .
git commit -am "submodule modify"
git push # 将子模块提交的更改推送至远程仓库
~~~

由于子模块和主模块是独立的两个仓库，主模块仅仅应用了子模块的 url 和 commit id。因此当子模块推送更改后，生成新的 commit id，但是主模块对子模块的引用配置并未发生更改，因此需要在主模块中同步进行提交更改。

~~~shell
cd ../ # 回到主模块目录
git add .
git commit -am "submodule reference modify"
git push # 推送主模块对子模块的引用记录更改到远程仓库
~~~

可以看到对于子模块的修改，我们需要分别提交和推送子、主木块的更改，当然我们也可以将 "推送至远程仓库" 这一步合并：
~~~shell
cd <main module dir> # 进入主模块目录
# 使用 --recurse-submodules=on-demand 选项，可以在推送主模块更改时，自动推送未推送的子模块
git push --recurse-submodules=on-demand
~~~

如果出现子模块提交了更改记录，但是未推送到远程仓库，主模块提交了子模块引用记录的变更，并完成了推送到远程仓库的操作。此时拉取主模块没问题，但是在拉取子模块时，会出现 "not our ref" 的报错。这是因为主模块引用了一个远程仓库未记录的 commit id 版本的子模块。需要在提交了变更记录的子模块中完成 push 即可。为了避免<mark>忘记推送子模块修改，仅推送了主模块的引用记录变更</mark>，可以将主模块的推送命令修改为：
~~~shell
# 使用 --recurse-submodule=check 选项可以自动检查子模块未 push 的错误
git push --recurse-submodule=check
~~~
当使用 "--recurse-submodule=check" 选项时，如果子模块存在未 push 情况，则当前 push 操作会报警；并且如果子模块存在 push 失败的情况时，也同样会报错。可以直接将其写入 git 配置，减少重复劳动：
~~~shell
git config push.recurseSubmodules check
~~~

## 5. modify 修改 submodule 远程仓库 url

~~~shell
cd <main module dir> # 进入主模块目录
# 修改主模块中 .gitmodules 中的 url
# 使用 sync 命令同步修改至 .git/config 中
git submodule sync
git commit -am "modify submodule url" # 提交修改
git push # 推送到主模块远程仓库
~~~

如果时别人修改了子模块 url，则拉取主模块的更新后，使用 sync 命令同步到本地 .git/config 中即可：
~~~shell
cd <main module dir> # 进入主模块目录
git pull # 拉取主模块更新，即获取 .gitmodule 中 url 的修改
git submodule sync # 将 .gitmodule 中的修改同步到本地仓库的配置中 .git/config
~~~

## 6. deinit 移除已有的 submodule

~~~shell
git submodule deinit <submodule name>
git rm <submodule dir>
git commit -am "remove submodule xxx"
git push
~~~



## 参考资料

1. [《Git - 使用git submodule的规范操作》](https://ldjhust.github.io/2018/08/22/Standard-Operation-of-Git-Submodule.html)
2. [《Git submodule 知识总结》](https://knightyun.github.io/2021/03/21/git-submodule)
3. [《来说说坑爹的 git submodule》](https://juejin.cn/post/6844903920645455879)