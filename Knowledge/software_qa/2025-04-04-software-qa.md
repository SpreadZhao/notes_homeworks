---
title: Flameshot on Wayland
date: 2025-04-04
tags: 
mtrace: 
  - 2025-04-04
---

# Flameshot on Wayland

之前一直没切wayland，最大的原因就是Flameshot截图功能在Wayland上就是一坨。这次研究了下，虽然还是不太行，但是差不多也可以了。

先介绍下我的配置：

![[Knowledge/software_qa/resources/Pasted image 20250404122612.png|400]] ![[Knowledge/software_qa/resources/Pasted image 20250404122633.png|400]]

左边的带鱼屏就是一个2k的水平，然后100%缩放，右边的这个稍微高一点，所以用了150%缩放。

用wayland触发`flameshot gui`的时候，会发现截图的区域永远都只是在这个带鱼屏上，并且只显示一部分。

后面我找了很多方法，发现用这种方式，能稍微可用：

```shell
env QT_SCREEN_SCALE_FACTORS="1.2;1" flameshot gui
```

这个`1.2;1`是一点一点试出来的。我猜和显示器的分辨率以及设置的实际缩放大小都有关系。在我的case中，设置成这个值，可以让两个显示器屏幕都显示在左边的带鱼屏上框选。

然后，就是pin to screen。在wayland里，pin之后上面会多出来一个tool bar，很烦人。可以加一个Window Rule给它去掉：

![[Knowledge/software_qa/resources/Pasted image 20250404124909.png]]

上面的Window Match是告诉kde如何匹配到这个Flameshot截图pin之后出现的窗口，其实也可以直接在窗口的tool bar上右键添加：

![[Knowledge/software_qa/resources/Pasted image 20250404125039.png]]

然后就是增加了两个属性，显示在其它窗口上面，还有去掉上面的tool bar。这样之后，pin上去的窗口就和X上没啥区别了。

唯一还没搞定的是，移动这个窗口现在只能先按住Meta键。