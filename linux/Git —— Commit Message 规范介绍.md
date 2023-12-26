---
title: Git —— Commit Message 规范介绍
date: 2023-12-26 09:41:22
tags:
  - 原创
  - Git
categories:
  - 程序员进阶
---
# Git —— Commit Message 规范介绍

## 为什么要规范 Commit Message

日常开发中，我们经常会使用到 Git 进行代码管理，而 Git 中最常用的命令就是 `git commit`，我们通过 commit 命令将修改后的代码提交到本地仓库，然后再通过 `git push` 命令将本地仓库的代码推送到远程仓库。

git 规定提交时必须要写提交信息，作为改动说明，保存在 commit 历史中，方便回溯。规范的 log 不仅有助于他人 review, 还可以有效的输出 CHANGELOG，甚至对于项目的研发质量都有很大的提升，尤其是一些长期持续迭代维护，且多版本长期并存的项目。

优秀的规范化 Commit Message 应该具备以下优点：

1. 清晰明了：commit message 应该清晰明了，说明本次提交的目的，具体做了什么操作。这样可以让团队成员更好地理解每次提交的内容。

2. 便于追溯：规范的 commit message 可以帮助程序员对提交历史进行追溯，了解发生了什么情况。

3. 提高研发效率：一旦约束了 commit message，意味着我们将慎重的进行每一次提交，不能再一股脑的把各种各样的改动都放在一个 git commit 里面，这样一来整个代码改动的历史也将更加清晰。

4. 自动化工具的友好性：规范的 commit message 可以被自动化工具用于生成发布日志或自动化版本号管理。

5. 降低代码维护成本：如果 commit message 写得不清楚，例如使用 "fix bug" 这样笼统的描述，可能会导致后续代码维护成本特别大，有时自己都不知道自己的 "fix bug" 修复的是什么问题

因此，规范化的 commit message 对于团队协作和项目管理是非常重要的。

## Commit Message 规范

在 Git 中 Commit Message 的规范又很多种，其中比较受欢迎的有以下这些：

1. Angular 规范：这是目前使用最广的写法，比较合理和系统化，并且有配套工具可以辅助生成（VS CODE 插件：git-commit-plugin）。

2. Conventional Commits 规范：这是一个基于语义化版本 (semver) 和简单的消息格式的轻量级约定，一传达代码更改的意图。

这里我们着重介绍以下 Angular 规范。

### Angular 规范



Angular 规范是一种广泛应用于软件开发中的提交信息（commit message）规范，其目的在于提供一种统一、清晰的提交信息格式，以便于开发者和维护者理解代码的变动情况。

Angular 规范将提交信息分为三个部分：Header、Body 和 Footer。

1. **Header**：Header 是提交信息的头部，包含三个字段：type、scope 和 subject。
    - **type**：type 字段用于指明<mark>本次提交的类型</mark>，例如：feat（新功能）、fix（修补bug）、docs（文档）、style（格式）、refactor（重构）、perf（性能优化）、test（增加测试）、chore（构建过程或辅助工具的变动）等。
    - **scope**：scope 字段用于指明<mark>本次提交影响的范围</mark>，例如：数据层、控制层、视图层等。这个字段是可选的。
    - **subject**：subject 字段是对<mark>本次提交目的的简短描述</mark>，不超过50个字符。

2. **Body**：Body 是提交信息的主体部分，用于<mark>详细描述本次提交的内容和原因</mark>。这个部分是可选的。

3. **Footer**：Footer 是提交信息的尾部，通常用于<mark>记录不兼容变动和关闭 Issue</mark>。这个部分也是可选的。

Angular 规范的提交信息格式如下：

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

以上就是 Angular 规范的基本内容。遵循这种规范可以帮助我们更好地管理代码提交的内容，使得每次提交的目的和主要改动都能清晰地体现在 Commit Message 中，便于后续的代码维护和版本管理。具体的规范可能会因团队和项目的不同而有所差异。所以，虽然 Git 本身没有强制的 Commit Message 规范，但在实际使用中，我们通常会根据实际需求来约定一些规范，以提高代码的可读性和维护性。


## Commit Message 规范工具

在实际开发中，我们通常会使用一些工具来帮助我们规范 Commit Message，这样可以避免我们手动编写 Commit Message，提高开发效率。

这里介绍一个 VS Code 插件：**git-commit-plugin**，它可以帮助我们快速生成符合 Angular 规范的 Commit Message。这是它的仓库地址：[https://github.com/RedJue/git-commit-plugin](https://github.com/RedJue/git-commit-plugin)。我们也可以直接在 VS Code 的扩展商店中直接下载安装。

>注意：商店中有两个名为 git-commit-plugin 的插件，我们选择作者为 redjue 的那一个。另一个为fork的版本，已经不再维护。

![git-commit-plugin](https://gitlab.b1gfac3c4t.top:1594/xu4nch3n/notebooks/uploads/0cd57b334faac82c88858d17cb3ff3e3/1703558434900.png)

安装完成后，我们点击 VS Code 左侧面板的存储库图标，可以看到存储库的右边出现了一个 git 图标，点击它就可以打开 git-commit-plugin 的界面。

1. 打开 git-commit-plugin 界面：

      ![](https://gitlab.b1gfac3c4t.top:1594/xu4nch3n/notebooks/uploads/153afa2bf93c4a55d9e4c5420aa43ee0/1703558938449.png)

2. 选择提交类型，这里以 fix 为例：

      ![](https://gitlab.b1gfac3c4t.top:1594/xu4nch3n/notebooks/uploads/1ec1f6e85b71bd74e244632584d64eb3/1703559027194.png)

3. 按照 Angular 规范依次完成 scope，subject，body 和 footer 字段

      ![](https://gitlab.b1gfac3c4t.top:1594/xu4nch3n/notebooks/uploads/45e369bb0b664b9ee79ae550bc0524cb/1703559610753.png)

4. 完成每项内容的填写后，点击 Complete，插件会将填写的内容，按照规范自动生成在左侧源代码管理面板中的提交信息中：

      ![](https://gitlab.b1gfac3c4t.top:1594/xu4nch3n/notebooks/uploads/cd6eb2280db4b9e0e125bc83329c0bd6/1703559528891.png)

      ![](https://gitlab.b1gfac3c4t.top:1594/xu4nch3n/notebooks/uploads/df8e94be63ac8bdc9528e84a2bf2cd83/1703559648902.png)
      
      注意此处 VS Code 会提示“当前行比72超出39个字符”，这是因为 VS Code 限制了提交消息每行的最大长度。我们在上一步编辑 Body 时，受限于插件的界面，将所有内容都写在了一行中，因此在这里我们需要将 Body 部分的内容进行换行，达到美观和规范的目的。

5. 最后我们点击提交按钮将本次修改提交到本地仓库，再点击同步就可以将本地仓库的提交记录推送到远程仓库了。


## Commit Message 模板的配置和使用




