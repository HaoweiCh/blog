---
title: APPLE silicon 安装 homebrew 踩坑细节
description: APPLE silicon 安装 homebrew 踩坑细节
categories:
- DevOps
tags:
- Apple silicon
- homebrew
date: 2021-12-09T06:12:43+08:00
year: 2021
week: 49
updated: 2021-12-09T06:12:43+08:00
---

我在写玩具项目是用到一些依赖库， 如 macdriver 出现编译报错。

不得不踩坑学习一下 silicon (m1 pro) 下 一些细节

<!-- more -->

## 关键点
* 一般来说, x86 程序可直接运行（Rosetta 自动介入） 
* 可以右击应用，在应用详情里强制要求使用Rosetta 打开
* 通过 `arch -x86_64`, 如 `arch -x86_64 go` 强制要求命令行 app 使用 Rosetta
  * 当然，如果 terminal 是 x86 模式（Rosetta 已经介入）， 不需要 `arch -x86_64 zsh` 
* nodejs 和 golang 开发者都会遇到部分库在 arm 原生模式下不好用，建议同时安装两种 homebrew


## Homebrew 现状

即使是现在（2021-12-09T05:53:31）不是所有工具链或者依赖库能正常的支持 m1 max arm 芯片架构

虽然自动 x86 模式大部分情况下运行完美, 一些比普通应用更复杂一些的编译工具，工具链的依赖链条里得保持一致的应用构建CPU架构. 

我们大多习惯于用 homebrew 安装应用, 我们不得不再安装一个原生 x86 版本的 homebrew, 要不然我们只能安装 arm64 的工具.

---

*友情提示：请确保在 x86 的 shell 里安装 x86 homebrew*

```bash
arch -x86_64 bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

当前 termial 或者说 shell 会话里，你可以直接运行 homebrew 去安装 x86 的包

```bash
brew install go
```

正常安装的话，homebrew 应该安装在 `/usr/local` 下，而不是 `/opt`. 

从这里看两个版本不会有什么冲突，但是，显而易见，一般不建议同时安装两个版本，容易混乱。
单版本 hombrew 的话直接以 `arch -x86_64` 前置命令来运行也不会出问题

```
arch -x86_64 brew install go
```

双版本 homebrew 的话需要指定 homerew 完整路径

```bash 
arch -x86_64 /usr/local/bin/brew install go
```

## Tips 技巧，细节

### Terminal Trick

直接复制 Terminal.app, 然后设置其中一个通过 Rosetta 打开，简单快捷不少

Thanks for the tip [@tmc](https://github.com/tmc)

### Golang 交叉编译 cross compiling

虽然 arm64 版本的 Go 编译器也可以通过制定 GOARCH 交叉编译 x86_64 原生 app，但是 CGO 就复杂多了。
这也是为什么 GO 也需要安装两个版本。


### 应用 CPU 架构检查

通过 `file` 命令检查可执行程序(binary)的目标编译架构