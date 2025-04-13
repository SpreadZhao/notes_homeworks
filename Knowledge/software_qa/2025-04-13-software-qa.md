---
title:
  - Wayland & Wechat
  - Wayland & LibreOffice
date: 2025-04-13
tags:
  - wayland
mtrace:
  - 2025-04-13
---

# Wayland & Wechat

其实我之前遇到过类似的坑：[[Knowledge/software_qa/2024-08-25-software-qa|2024-08-25-software-qa]]。当时是直接在`/etc/environment`里设置环境变量。但是，Fcitx里的说明已经有了这点：[Using Fcitx 5 on Wayland - Fcitx](https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland#KDE_Plasma)

> - Certain Gtk/Qt application that only works under X11 may still need to set `GTK_IM_MODULE` or `QT_IM_MODULE` for them individually.
> - If you set `GTK_IM_MODULE/QT_IM_MODULE` globally, you will hit this issue [Candidate window is blinking under wayland with Fcitx 5](https://fcitx-im.org/wiki/Special:MyLanguage/FAQ#Candidate_window_is_blinking_under_wayland_with_Fcitx_5 "Special:MyLanguage/FAQ")

意思就是说，适配了wayland的app，即使原来是x的，那么也可以读取`XMODIFIERS=@im=fcitx`配置。但是有些App就没适配。比如微信。这个时候就只能给它单独设置了。在微信的desktop entry里加上：

```
[Desktop Entry]
Type=Application
Name=Wechat
Exec=env GTK_IM_MODULE=fcitx QT_IM_MODULE=fcitx /home/spreadzhao/App/WeChatLinux_x86_64.AppImage
Terminal=false
```

单独给执行命令加上环境变量。

# Wayland & LibreOffice

使用Wayland + KDE + Fractional Scaling之后，LibreOffice在两个屏幕上出现了一个大一个小的情况。还是没有适配好缩放的问题。解决方法：[Xwayland](https://wiki.archlinux.org/title/Xwayland)

启动的时候把Wayland显示的环境变量置空，就能让LibreOffice以XWayland模式启动。这样就没问题了。置空的办法和上面微信是一样的操作：

```
Exec=env WAYLAND_DISPLAY= libreoffice --calc %U
```

这里等于号右边什么都没有就是置空了。用这种方式以后，无论是从开始菜单启动，还是直接在dolphin里打开文件，缩放都是正常的了。