```yaml
title: RMBP 使用问题各项总结
categories:
- DevOps
tags:
- macOS
date: 2017-09-16T14:22:03+08:00
year: 2017
week: 37
updated: 2021-12-07T08:35:46+08:00
```

第一次遇到苹果电脑突然不能启动, 就近找了一家苹果"专业"维修。
店里维修员拿出一根很破旧的 magsafe 线插电，简单的按了开机键不能开机后，就说要放在他那维修，（这么简单！）。
这操作一看就不靠谱， 先回去查资料学习再说。
  
 <!-- more --> 
 
回家尝试网上搜索一番，重设PRAM和重设SMC了一下问题就好了。😅  
以后就养成了一个习惯出了问题先重置。

## 一次惊慌失措的无法启动

### 重设PRAM
1. 关闭 Mac。
2. 在键盘上找到以下按键的位置：Command (⌘)、Option、P 和 R。您需要在步骤 4 中同时按住这些键。
3. 启动电脑。
4. 出现灰屏前按住 Command-Option-P-R 键。
5. 按下这些键，直到电脑重新启动，您会再次听到启动声。电脑自行重启的第三次，您就可以松开了。

### 重设SMC
1. 关闭电脑。
2. 将 MagSafe 电源适配器连接到电源和 Mac（如果尚未连接的话）。
3. 在内建键盘上，同时按下（左侧）Shift-Control-Option 键和电源按钮。这个步骤不会启动电脑。
4. 接下来，您可以同时松开所有键和电源按钮。
5. 按下电源按钮打开电脑。

## 第一次进水
> 一次不小心把肥仔快乐水打倒一点到键盘旁的音响孔。由于没有多少，擦干就把这件事情给放在一边了。后面开机莫名其妙断电，后来直接开不了机。急忙跑到重庆苹果直营店售后。

1. 官方并不会当场检验，一般就是询问基本情况
2. 我当时并没有意识到是可乐的问题，邮件发来天价报价单，并告知人为不保修
3. 工作人员在检查后没有完全清干净可乐


最后私下找熟人拆机清理，成功点亮。。

私人建议：

1. 一定一定，很少很少的饮料都要注意立即断电。
2. 找专业的人拆机。饮料要用清洗剂（洗板水）清洗，纯净水的话吹干就行了。一般来说单纯断电后去拆机清洗，价格都不太贵


## iPhone 通过 USB 连接 Macbook 就响个不停的问题

我的线能正常充电，但是连接电脑后响个不停。
重设 PRAM 和重设 SMC 没啥效果。
猜测可能是usb接口松了，尝试轻摁线头一会儿就连接正常了。

* 技巧总结
  1. 买专门的热缩管保护数据线线头接口处（这里最容易断）
  2. 手工做数据线保护套（短绕线）

## 手动更换电池
> 笔记本在使用2年左右，就可能因为习惯或者环境问题导致电源鼓包

最好的方式就是自己更换电池，这样待机时长还原，又减少电池爆炸主板全毁的风险