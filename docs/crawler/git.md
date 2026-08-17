# 本地 Git 安装与配置指南

!!! 致谢
    本章内容复制自韩文弢老师开设的程序设计训练（Rust）[课程文档中的对应章节](https://lab.cs.tsinghua.edu.cn/rust/git/)。转载前已获得授课教师授权。在此对原作者表示感谢。

本文仅介绍如何在本地使用 Git 进行代码管理，本次作业无需使用远程仓库。

## Git 安装与配置

如果使用 Windows，请在 WSL 中进行下列操作。

如果使用 Debian/Ubuntu 系列的发行版：

```shell
sudo apt update
sudo apt install -y git
```

如果使用 macOS：

```shell
brew install git
```

也可以用 macOS 自带的 git 版本，但是它版本会老一些。

安装完，可以用 `git --version` 命令确认安装成功：

```shell
$ git --version
git version 2.x.x
```

此后，请使用下面的命令配置 git 身份：

```shell
git config --global user.name "Your Name" # 通常可以是自己名字，或者 ID
git config --global user.email "name@example.com" # 例如清华邮箱
```
## 初始化本地仓库
在你想要管理的项目文件夹下执行：
```shell
git init
```

## Git 简要使用

Git 基本操作：

1. `git commit -am "在这里写提交说明"`：新建一个提交（commit），并且给这个提交设置一个提交说明（commit message）
2. `git add .`：将更改添加到暂存区
3. `git log"`: 查看历史提交

更详细的 Git 使用，见：

- [Git 简明指南](https://rogerdudler.github.io/git-guide/index.zh.html)
- [Git 教程](https://www.liaoxuefeng.com/wiki/896043488029600)
- [缺失的一课](https://missing-semester-cn.github.io/2020/version-control/)
- [Git Cheatsheet](https://education.github.com/git-cheat-sheet-education.pdf) 