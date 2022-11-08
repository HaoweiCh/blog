```yaml
title: 解压 iOS app 中 Assets.car 的资源(图标等)
categories:
- ReverseEngineering
tags:
- iOS
- DataCollect
date: 2022-11-01T12:28:00+08:00
year: 2022
week: 44
updated: 2022-11-01T12:28:00+08:00
```

本意拿到 ios ipk 包后想修改图标方便多开后区分哪个是哪个

聊一下从 BOM 格式的压缩包(Assets.car) 中提取~~并修改~~资源文件

> Export images from OS X / iOS .car archives.  
> Very rough code, probably tons wrong with it, but still useful.

<!-- more -->

现在的缺点是只能导出图片文件, 想要做到 Assets.xcassets 格式貌似还很麻烦.

* 当前环境环境
    * xcode
        * Version 14.0.1 (14A400)
        * ![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/28F181C89D5A852458037F10A8D8EE8B8F724946.webp)
    * macOS
        * 13.0
        * ![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/FEF5420748847CECB8744810EFAC5C2C36FE8652.webp)

## Cartool 使用说明

1. 拉本项目 `git clone --depth=1 https://github.com/HaoweiCh/cartool.git`
  * https://github.com/HaoweiCh/cartool
2. Command+B 编译
3. *Show Build folder in Finder*
    * ![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/2BC3B0B70BDDDF5AA390C953BCE118BD359D0715.webp)
4. cp ./Products/Debug/cartool destination-you-want
    * 拷贝 cartool 到任何位置你喜欢的如 /usr/local/bin 就可以全局访问了
5. cartool Payloader/demo.app/Assets.car ./assets
    * 导出 Assets.car 的所有资源到 assets 目录下（记得提前创建 assets 目录哦）

## 逆向分析

* 苹果官方一直在扩展 Asset 目录的功能
  * Xcode 13 
    * Alternate App Icon sets
    * 运行时使用 assets 里的 icon
    * `UIApplication.shared.setAlternateIconName(nil) `
  * Xcode 12
    * 添加 svg 图片集支持
  * Xcode 11
    * Fixed an Asset Catalog bug that prevented named colors from being found at runtime 
  * Xcode 10
    * 优化图片压缩算法(Apple Deep Pixel Image Compression), 支持模式切换
  * Xcode 9 
    * 添加色彩集( Color Asset )支持, 优化 PDF 支持

出乎我意料的是官方几乎没有披露是怎么编译 assets 目录到 .car 文件的, 而且我在网上能找到的文档相当少.

据此, 这篇技术分析文章我计划披露一些 car 文件的格式信息. 

随文你我都可以建立一个项目来处理 car 文件.

声明: 这篇文章和相关代码完全是出于学习和研究的目的, 不保证导出数据完整和精确性.

文章基于 macOS 13 Ventura 和 Xcode 14 不保证后期版本的一致性

### 什么是 Asset 目录?

从 Xcode 5 开始引入解决图片管理, 比如同一图标文件不同分辨率不同DPI ( AppIcon@2x, AppIcon@3x ). 

* `.xcassets` 目录
  * https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_ref-Asset_Catalog_Format/index.html
  * https://help.apple.com/xcode/mac/current/#/dev10510b1f7
  * ![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/838C55A4DA659F6E4FD916262E609522FF92CC30.webp)

### 什么是 `.car` 文件

asset 目录并不是直接复制到 app 包, 而是打包压缩编译成 `.car` 文件.

换句话说 .car 包含所有 asset 内容, 下面简化表达怎么在 app 运行时调用资源

`UIImage *myImage = [UIImage imageNamed:@"MyImage"];`

未完待续...