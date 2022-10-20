---
layout: post
title: "解决conda报错"
subtitle: "问题解决记录"
author: "Joy"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.2
tags:
  - 其他
---

## （一）解决conda安装包出现ProxyError报错问题

在conda中安装numpy时出现如下报错：

![img](/img/in-post/proxyerror.png)

- 问题原因：VPN自动修改代理
- 解决方式：在当前环境中输入：

```
set _PROXY=http://?:?
```

