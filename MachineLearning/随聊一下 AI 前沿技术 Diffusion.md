```yaml
title: 随聊一下 AI 前沿技术 Diffusion
categories:
- MachineLearning
tags:
- DreamBooth
- Stable Diffusion
date: 2022-11-08T10:55:49+08:00
year: 2022
week: 45
updated: 2022-11-08T10:55:49+08:00
```

最近 ai 画图火了, 顺道了解一下.  
**Stable Diffusion** is SOTA for now

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/CD000442380B3ED657F25E0441CB5A1E48C8B4F6.webp)


<!-- more -->

大概十天前, 网传 NovelAI 数据库泄漏.   
应家里"画师"要求. 第一次尝试了一下

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/31316822AB090BC51C71BF091249DFBE0817C3A0.webp)

这个很像小黄图生成器, 完全是基于文字进行 AI 画画
据说素材来自于 https://gelbooru.com/

这里分享 prompt 提取工具 (bookmarklet)

```javascript
javascript:var ret=[];document.querySelectorAll("#tag-list > li.tag-type-general a[href^='index.php?page=post']").forEach((a) => ret.push(a.innerText));ret.join(" ");dev_null=prompt("",ret);
```

## 由 Dreambooth 引发的第二次了解

诚然, NovelAI 的尝试带点娱乐性, 而且我并没有感觉到多大的乐趣. 
更多的是浅浅了解我完全不熟悉的领域, 毕竟短短天把天就搞出来的东西也就只能是应用工程学级别的了

最近学习 Google 的学术分享作业的原因归结于社区的反应炸出来的

社区里他们 AI 工程师反响剧烈, 出于兴趣我便对这个领域 microlearning 了一下

大概效果于题图
找了点 Gakki 的图片做了简单训练个几次, 结果还挺香

## 收获

从上面的技术了解 routine 来说, 感觉收获挺大的. 对行业横向的了解更多的是满足自己.

作为沉淀, 我深入了解了一下社区和应用.

不得不感叹人的经验是有限的, 即使眼红 https://AvatarAI.me 上线一天一万刀. 想要做到这个结果也并不是那么容易

扩大 AI 前沿技术的下沉应用在技术迸发出来的早期应该是最适合小程序推出的. 

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/A97774D6D474CA03F795FAEE1D94AAF8554A5C59.webp)