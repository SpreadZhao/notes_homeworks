---
title: Wayland with Fractional Scaling & Jetbrains
date: 2025-04-24
tags: 
mtrace: 
  - 2025-04-24
---

# Wayland with Fractional Scaling & Jetbrains

我使用了Fractional Scaling，然后在设置里：

![[Knowledge/software_qa/resources/Pasted image 20250424003818.png]]

让系统帮忙缩放。然后就发现，Jetbrains系列的软件，在我的主屏幕上就非常大，然后在另一个显示器上就是正常的。解决方法就是，在vm options里加上：

```
Dawt.toolkit.name=WLToolkit
```

但是，根据很多帖子，这个选项都是有问题的。[Dawt.toolkit.name=WLToolkit](https://youtrack.jetbrains.com/issue/JBR-8223/Pure-Wayland-on-Arch-Hyprland-no-cursor#focus=Comments-27-11649983.0-0)。所以，最后我还是不用了。