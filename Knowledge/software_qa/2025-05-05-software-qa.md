---
title: 卸载三星自带浏览器
date: 2025-05-05
tags:
  - samsung
mtrace:
  - 2025-05-05
---

# 卸载三星自带浏览器

```shell
adb shell pm uninstall --user 0 com.sec.android.app.sbrowser
```

其中`com.sec.android.app.sbrowser`为三星浏览器的包名。可以用下面的命令确认：

```shell
adb shell pm list packages | grep browser
```

