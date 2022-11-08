```yaml
title: 各种环境下的 android 动态库(so 文件)编译
description: 各种环境下的 android 动态库(so 文件)编译
categories:
- Languages
tags:
- Golang
- Android
date: 2022-10-29T23:45:29+08:00
year: 2022
week: 43
updated: 2022-11-07T02:03:21+08:00
```

## Linux 下的环境编译

各位看官来我们先看编译命令

```shell
export CGO_ENABLED=1
export GOOS=linux
export GOARCH=arm64
export GOARM=7
export CC=aarch64-linux-gnu-gcc
go build -buildmode=c-shared -o hello.so .
```

环境依赖 `apt-get install -y gcc-aarch64-linux-gnu`

我这里使用的 amd64 架构下的 docker 容器, 所以响应主题“**各种环境**”

## MacOS 下的编译环境

* 环境依赖 NDK
  * 这里建议用 android studio 进行管理
  * 默认位置 `~/Library/Android/sdk/ndk`
    * 例如 `$ANDROID_HOME/ndk/25.1.8937393`
    * $ANDROID_HOME/ndk/25.1.8937393/toolchains/llvm/toolchains/prebuilt/darwin-x86_64/bin/armv7a-linux-androideabi28-clang

CGO_CFLAGS="-arch arm64" CGO_LDFLAGS="-arch arm64"

```shell

export CC=$ANDROID_HOME/ndk/25.1.8937393/toolchains/llvm/prebuilt/darwin-x86_64/bin/armv7a-linux-androideabi28-clang
CGO_ENABLED=1 GOARCH=arm GOOS=android go build -buildmode=c-shared -o hello.so .
```

### issue arm64 编译报错 _check_for_64_bit_pointer_matching_GoInt declared as an array with a negative size

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/ABAA25E9537DDDA48596558CA36F16AEA1EE946C.webp)

编译目标架构改为 arm 正常
这个可能是目前 apple silicon 下的问题, 可以忽略

## 编译参数

armv7a-linux-androideabi28 可以指定 androidabi 版本
目标系统的话 linux 其实应该也可以, 详情 `go tool dist list | grep android`

-build-mode 释义 `go help buildmode`

```text
-buildmode=c-shared
		Build the listed main package, plus all packages it imports,
		into a C shared library. The only callable symbols will
		be those functions exported using a cgo //export comment.
		Requires exactly one main package to be listed.
```

## 源码 example

```go
package main // 包名限制为 main

import (
	"C" // 
	"fmt"
)

//export Hello
func Hello() {
    // 被调度函数需要标记 export, 暴露的方法名需与 golang 的方法名一致
    // 不能和 c 内置方法名冲突
	fmt.Println("hello from golang")
}

func main() {} // 即使为空也强制包含 main 入口函数
```