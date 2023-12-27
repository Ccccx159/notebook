---
title: Git —— Issue 模板
date: 2023-12-27 09:40:51
tags:
  - 原创
  - Git
categories:
  - 程序员进阶
---
# Git —— Issue 模板

## 1. 什么是 Issue 模板

Issue 模板是在创建 Issue 时，预先填写好的内容，可以是一些提示性的文字，也可以是一些表单，用户在创建 Issue 时，可以根据模板填写内容，这样可以让 Issue 的内容更加规范，也方便了 Issue 的管理。

## 2. 如何使用 Issue 模板

### 2.1 创建 Issue 模板

在项目的根目录下创建一个名为 `.gitlab` 的文件夹，然后在 `.gitlab` 文件夹下创建一个名为 `issue_templates` 的文件夹，然后在 `issue_templates` 文件夹下创建一个名为 `bug_report.md` 的文件，这个文件就是 Issue 模板了。

### 2.2 编写 Issue 模板

在 `bug_report.md` 文件中，可以写一些提示性的文字，也可以写一些表单，例如：

```markdown
---
name: Bug report
about: Create a report to help us improve
title: ''
labels: ''
assignees: ''

---
# 🐞 Bug Reporter

## 📋 Pre-Check

- [ ] I have searched the [issues](https://gitlab.b1gfac3c4t.top:1594/xu4nch3n/sample-gitlab/-/issuess) of this repository and believe that this is not a duplicate.

## 🐛 Bug Summary

> A clear and concise description of what the bug is.


## 🐛 Bug Reproduce

> Steps to reproduce the behavior:

1. Go to '...'
2. Click on '....'
3. Scroll down to '....'
4. See error

## 🐛 Expected Behavior

> A clear and concise description of what you expected to happen.

## 🐛 Possible Solution

> Describe the solution you thought of.

## 🐛 Context

> How has this issue affected you? What are you trying to accomplish? Providing context helps us come up with a solution that is most useful in the real world.

## 🐛 Your Environment

> Include as many relevant details about the environment you experienced the bug in.

|                  |             |
| ---------------- | ----------- |
| OS               | x86?        |
| Compiler         | gcc-4.8.5 ? |
| Lib Ver(s)       |             |
| Lib Url          |             |
| Other Dependency |             |

## 🐛 Additional context

> Add any other context about the problem here.


```


### 2.3 使用 Issue 模板

在项目的 Issues 页面，点击 New issue 按钮，就可以看到 Issue 模板了，用户在创建 Issue 时，可以根据模板填写内容，这样可以让 Issue 的内容更加规范，也方便了 Issue 的管理。

## 3. 参考资料

- [Creating issue templates for your repository](https://docs.github.com/en/free-pro-team@latest/github/building-a-strong-community/creating-issue-templates-for-your-repository)


