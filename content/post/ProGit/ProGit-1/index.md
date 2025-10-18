---
title: ProGit Learn Record 1 ：起步
description: learn progit book
slug: progit-book-1
image:
draft: false
categories:
  - Programming
date: 2025-10-14 15:21:06+08:00
tags:
  - Git
weight:
---
>本系列文章基于《Pro Git》一书，作为本人学习记录
# Git的来源
版本管理是开发中常用的需求，最开始的版本管理软件，基于本地数据库管理，最流行的便是RCS。后来，面对不同系统的开发者需要协同工作的需求，产生了集中版本控制系统（Centralized Version Control Systems，CVCS），但是CVCS会导致一旦服务器不工作就会导致整个仓库的历史遗失，于是针对这一问题，分布式版本控制系统（（Distributed Version Control System，DVCS）诞生了，这种版本控制系统会在下载仓库中的文件时，一同下载历史更改，所以即使服务器挂掉，也可以通过本地的仓库恢复完整记录。  
Linux作为一个系统内核，拥有着极其繁重的版本控制需求。一开始，Linux社区使用Bitkeeper作为版本控制软件，但是随着合作关系的结束，Bitkeeper公司结束了Linux免费使用该软件的权利，于是Linus自己编写Git这个版本控制软件，并且设定了如下目标：
- 速度
- 简单的设计
- 对于非线性开发的支持（拥有大量的并行开发的分支）
- 完全基于分布式
- 高效管理超大规模的项目（如Linux内核）  
---

- Git不记载每个文件与初始版本文件的差异，它对每个文件进行快照。Git对待数据更像是一个快照流。  
- Git会把在数据存储前使用哈希进行校验他，这使得Git保证了数据的完整性。  
- Git一般只会往Git的数据库中添加数据，你很难从Git数据库中删除数据

---
Git有三种状态：**已提交（Committed）** 、**已修改（modified）** 和**已暂存（staged）** 。
- modified 表明已修改了文件，但尚未保存到数据库中。
- staged 表明对一个已修改文件的当前版本做了标记，使其包含在下次提交的快照中。
- committed 表示数据已安全的保存在本地数据库中。   

这使得Git有用三个阶段：工作区、暂存区和Git目录。

- 工作区是对项目的某个版本独立提取出来的内容。 这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。
- 暂存区是一个文件，保存了下次将要提交的文件列表信息，一般在 Git 仓库目录中。
- Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。 这是 Git 中最重要的部分，从其它计算机克隆仓库时，复制的就是这里的数据。

因此，最基本的Git工作流就是：  
1. 修改文件（工作区）
2. 将你的更改暂存（暂存区）
3. 提交更新（将暂存区的快照永久地提交到Git目录中）  

# 命令行  
本系列只使用Git 命令行，因为  
1. 不是所有人都一定有GUI工具，但一定有CLI。
2. GUI通常对Git的所有功能取一个子集，CLI拥有Git的所有功能。
## 下载并安装Git
 请自行在 [Git 官网](https://git-scm.com/downloads) 或通过搜索引擎自行查找下载安装方式。 
## 配置（config）
1. 在`/etc/gitconfig` 文件中，包含系统中每一个用户及他们仓库的通用配置。
2. 在 `~/.gitconfig` 或 `~/.config/git/config` 文件包含当前用户的配置。使用 `--global` 选项可以使系统中所有仓库生效
3. 项目仓库中 `.git/config` 是该仓库的设置，使用 `--local` 可以使git强制使用它（虽然默认就是它）  
Windows 系统在 `$HOME` 目录下（ `C:\Users\$User`) 的`.gitconfig`目录，系统级目录在 `C:\ProgramData\Git\config`  
使用 `git config --list --show-origin` 查看所有配置
## 用户信息  
使用  
`git config --global user.name <name>` 和
`git config --global user.email <email>`  
如果你想在不同的项目使用不同的名字和邮箱，那么在那个项目里使用不带`--global` 的命令。  
## 文本编辑器
配置默认文本编辑器，Git 会在输入信息时使用它，如果未配置，则会调用默认的编辑器   
  
`git config --global core.editor EDITOR`

## 检查配置  
使用 `git config --list` 检查所有能找到的配置。  
使用 `git config <key>` 如 `git config user.name` 检查某一项配置  
使用 `git config --show-origin <key>` 来检查哪一个文件最后设置了该值  

## 获取帮助 
使用 `git help <verb>`  或 `git <verb> --help` 或 `man git-<verb>` 获取manpage。如`git help config`   
如果使用 `-h` ，则可以获得简短的快速参考