---
title:
  - Arch Linux
date: 2024-04-26
tags:
  - softwareqa/linux
mtrace:
  - 2024-04-26
  - 2024-08-25
---

# Arch Linux

安装archlinux + kde/gnome/dwm/mate 遇到的傻逼事情。

[Installation guide - ArchWiki (archlinux.org)](https://wiki.archlinux.org/title/installation_guide)

## 开启网卡

```shell
ip link set _<设备名>_ up
```

## 中文

中文显示：`/etc/locale.gen`里面把中文加上重启就行，其它的不要搞。然后，需要`locale-gen`，并且是已经安装了中文字体的前提。

[简体中文本地化 - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E6%9C%AC%E5%9C%B0%E5%8C%96)

中文输入法：[Fcitx5 - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/Fcitx5)

- [fcitx5-im](https://archlinux.org/groups/x86_64/fcitx5-im/)
- [fcitx5-chinese-addons](https://archlinux.org/packages/?name=fcitx5-chinese-addons)

> fcitx5-im这个group里的所有包都是要的。有一次我没装fcitx5-gtk，发现比如edge，obsidian之类的软件都没法输入。gtk和qt就是为了针对对应的sdk开发的软件接入输入的功能的。

之后Input Method里面加Pinyin就行了：

![[Knowledge/software_qa/resources/Pasted image 20240427183710.png]]

之后使用了dwm，首先按[集成](https://wiki.archlinuxcn.org/wiki/Fcitx5#%E9%9B%86%E6%88%90)写环境变量，由于我用的xinit，所以根据[随桌面环境自动启动](https://wiki.archlinuxcn.org/wiki/Fcitx5#%E9%9A%8F%E6%A1%8C%E9%9D%A2%E7%8E%AF%E5%A2%83%E8%87%AA%E5%8A%A8%E5%90%AF%E5%8A%A8)在.xinit里加上

```shell
fcitx5 &
```

## 安装流程

大致安装流程：

- 下载ISO；
- 进入；
- 联网，iwctl
- archinstall
	- 安装 dhcpcd iwd
	- 装NetworkManager
- 进入，再次联网， **ip link set ... up**
- 安装plasma sddm

具体的安装流程：

进入镜像之后，先联网：

> 进入镜像失败，我遇到了这个：[############### INIT NOT FOUND ############### : r/linuxmint (reddit.com)](https://www.reddit.com/r/linuxmint/comments/18eohux/init_not_found/)使用GRUB2模式启动就好了。

```shell
iwctl
```

使用iwctl进行链接。使用说明：[iwd - ArchWiki (archlinux.org)](https://wiki.archlinux.org/title/Iwd#iwctl)

直接连：

```shell
station <name> connnect <SSID>
```

比如，name通常是wlan0，ssid就是wifi的名字。也可以扫描一下。具体的使用wiki里都有。

然后archinstall，选自己喜欢的就好。我这里不知道为什么linux内核不能换，只能用默认的内核。

我没加任何额外的包，因为我装了NetworkManager，之后用这个联网就啥都能装了。

## Proxy代理

终端需要设置代理，在随便一个脚本比如`.zshrc`里

```shell
export http_proxy=127.0.0.1:7897
export https_proxy=$http_proxy
export ftp_proxy=$http_proxy
export rsync_proxy=$http_proxy
export no_proxy="localhost,127.0.0.1"
```

后面的7897是clash端口。

[代理服务器 - Arch Linux 中文维基 (archlinuxcn.org)](https://wiki.archlinuxcn.org/wiki/%E4%BB%A3%E7%90%86%E6%9C%8D%E5%8A%A1%E5%99%A8)

后面我换了v2ray + v2raya来代理。注意到了这个问题：

- 不加export情况下，如果v2raya里Transparent Proxy/SystemProxy为Off，那么就是不开梯子上网的情况，即使是Running的状态。这种状态类似于打开了Clash，但是没开启System Proxy选项；
- 不加export情况下，如果v2raya里Transparent Proxy/SystemProxy为On的某一个，那么就是开梯子了，不加export也能翻墙；
- 所以我怀疑加了export的意思就是允许终端等其它读这个配置的程序走代理。
- 有些应用开了梯子没法下载了，比如linuxqq。所以按照第一条进行配置，就能下载了。

## copyq, flameshot 不支持 wayland

copyq的快捷键和tray，flameshot的pin，都不支持wayland。所以最后用的x11。

## 显示器白屏还闪

连接显示器会白屏还会闪。用的 AMD 的 GPU，参考 [AMDGPU - ArchWiki](https://wiki.archlinux.org/title/AMDGPU#Screen_flickering_white_when_using_KDE) ，我用的是 grub。所以在`/etc/default/grub`的`GRUB_CMDLINE_LINUX_DEFAULT`加入`amdgpu.sg_display=0`。加完就像下面这样：

```config
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet amdgpu.sg_display=0"
```

然后 `grub-mkconfig -o /boot/grub/grub.cfg`，重启。验证的话，重启之后输入：

```shell
cat /proc/cmdline
```

结果样例：

```shell
BOOT_IMAGE=/vmlinuz-linux-lts root=UUID=c117dfc0-be87-46b6-b5fd-95995e77fda3 
rw zswap.enabled=0 rootfstype=ext4 loglevel=3 quiet amdgpu.sg_display=0
```

## pacman \& yay tips

- [pacman/Tips and tricks - ArchWiki](https://wiki.archlinux.org/title/Pacman/Tips_and_tricks#Removing_unused_packages_(orphans))
- [pacman - ArchWiki](https://wiki.archlinux.org/title/Pacman)
- [Jguer/yay: Yet another Yogurt - An AUR Helper written in Go](https://github.com/Jguer/yay)

### 列出一个包里的文件（当然包括可执行程序）

```shell
yay -Ql <package_name>
```

> 这里可以用`| grep <xxx>`来过滤`bin`来过滤可执行文件

### 查找一个命令属于哪个包

```shell
yay -Qo <command>
```

### 列表备份和恢复

`pacman -Qqe > pkglist.txt`

### 更新不到新包，并且aur会下载失败

傻逼的我现在才想到，是mirror list的问题。在[这里](https://archlinux.org/mirrors/status/#successful)找到完全同步，并且Mirror Score最低的镜像。然后把他放到`/etc/pacman.d/mirrorlist`的最前面。这样就能同步到了。

## 稳定内核

[Arch Linux 更换到稳定版LTS内核 – 寻](https://poemdear.com/2019/03/27/arch-linux-%E6%9B%B4%E6%8D%A2%E5%88%B0%E7%A8%B3%E5%AE%9A%E7%89%88lts%E5%86%85%E6%A0%B8/)

## Grub Font Size

[HiDPI - ArchWiki](https://wiki.archlinux.org/title/HiDPI#Change_GRUB_font_size)

## SysRq

- [Keyboard shortcuts - ArchWiki](https://wiki.archlinux.org/title/Keyboard_shortcuts#Enabling)
- [Linux Magic System Request Key Hacks — The Linux Kernel documentation](https://docs.kernel.org/admin-guide/sysrq.html)
- [Magic SysRq key - Wikipedia](https://en.wikipedia.org/wiki/Magic_SysRq_key)

## Arch Font

Arch Linux 用的字体就是 Linux 内核的字体：[term](https://wiki.archlinux.org/title/Linux_console#Fonts)，名字叫`CONFIG_FONT_TER16x32`。字体的官网：[Terminus Font Home Page](https://terminus-font.sourceforge.net/)，字体的ttf版本：[Terminus Font](http://terminus-font.sf.net/)。

安装字体：[Fonts - ArchWiki (archlinux.org)](https://wiki.archlinux.org/title/fonts#Manual_installation)

查找字体的准确名称：`fc-scan -b`，来自包`fontconfig`。

## Mate Shortcut

我发现，按照之前 Ubuntu 的设置，把那两个导航快捷键干掉之后还是不行。所以我当时就怀疑还有其它的快捷键给拦截了。看一下 gsettings，可以发现它有查询的功能，按照下图查找：

![[Knowledge/software_qa/resources/Pasted image 20240513143038.png]]

发现果然有。把这两个也干掉之后，log out一下再回来就没问题了。

## Alacritty

- [Alacritty (github.com)](https://github.com/alacritty)
- [Alacritty - ArchWiki (archlinux.org)](https://wiki.archlinux.org/title/Alacritty)

我的习惯是，配置文件放在`~/.config/alacritty/alacritty.toml`，配色使用的是[gruvbox_dark](https://github.com/alacritty/alacritty-theme/blob/master/themes/gruvbox_dark.toml)

## Dunst

[Dunst - ArchWiki (archlinux.org)](https://wiki.archlinux.org/title/Dunst)

我找到的比较好的配置：[dunstrc (github.com)](https://gist.github.com/h3ndry/f490f0f5bff6256d8f97d8047babe611)

## Ranger

[ranger.github.io/ranger.1.html](https://ranger.github.io/ranger.1.html) 这里提到了，如果不想预览大文件，可以这样设置：

```shell
set preview_max_size 1000000
```

这样所有大于1M的文件就都不会预览了。

> see also: [set preview_max_size 1000000](https://github.com/ranger/ranger/issues/966)

## Audio

音频配置的问题。

使用pipewire，不要直接用alsa进行音量控制（比如amixer）。安装：

- pipewire
- pipewire-alsa
- pipewire-pulse

pipewire兼容alsa和pulseaudio。后面的两个就是为了兼容用的。当用了这些之后，所有的声音就工作正常了。下面说怎么控制音量。

pipewire我看它官网的描述，它的特色不是为了控制音频输出的，而是midi其它一些乱七八糟的。不过，它确实能控制音频，**因为它对pulseaudio兼容**。所以，控制音量其实还是在用pulseaudio的能力。不过，我们用的不是[pulseaudio包](https://wiki.archlinux.org/title/PulseAudio#Installation)里面的东西，而是libpulse。

libpulse是最底层的库，只是个库。所以pipewire必然会依赖它。我们可以直接使用libpulse提供的pactl命令：[PipeWire - ArchWiki](https://wiki.archlinux.org/title/PipeWire#PulseAudio_clients)。

比如`pactl info`会输出当前的audio server信息：

```shell
❯ pactl info
... ...
Server Name: PulseAudio (on PipeWire 1.0.7)
... ...
Default Sink: alsa_output.usb-Apple__Inc._EarPods_N6CPX970D3-00.analog-stereo
Default Source: alsa_input.pci-0000_64_00.6.HiFi__hw_acp63__source
... ...
```

其它的自己看wiki吧，都非常简单。我这里要强调一下，千万不要把pipewire和pulseaudio一起装，它们两个是会打架的：

- pipewire-alsa和pulseaudio-alsa冲突；
- pipewire-pulse和pulseaudio  pulseaudio-bluetooth冲突。

只装`pipewire-`开头的，不要装其它的。我之前就都装了。出现了很多奇怪的问题。比如所有的在线视频和音乐都无法播放，但是网页浏览快得飞起（我没试本地视频，估计也一样）。一开始还以为是网络的问题，后来我无意中更新了一下pulseaudio，居然就好了。这才让我意识到是这里的问题，但是当时也没意识到是冲突的问题，后来研究了半天才发现。

另外，pactl的用法，看man。下面这段就是复制下来的：

```shell
set-sink-volume SINK VOLUME [VOLUME ...]
	Set the volume of the specified sink (identified by its symbolic name or numerical index). VOL‐
	UME  can  be  specified  as an integer (e.g. 2000, 16384), a linear factor (e.g. 0.4, 1.100), a
	percentage (e.g. 10%, 100%) or a decibel value (e.g. 0dB, 20dB). If  the  volume  specification
	start with a + or - the volume adjustment will be relative to the current sink volume. A single
	volume  value  affects  all  channels;  if multiple volume values are given their number has to
	match the sink's number of channels.
```

或者，我们可以用pamixer，这个需要装一下。注意，我们查一下可以知道，这个东西不依赖pulseaudio，只依赖libpulse，所以是没问题的。

> PS：原来，pipewire-pulse是支持动态切换sink的，是支持耳机线控的，是支持实时查询Default Sink的音量的。就是因为之前一直冲突，所以这些功能都用不了。。。**还有，尼玛蓝牙也是因为这个用不了。😅**

## Lock Screen

首先，锁屏用的是slock，很轻量。重要的不是锁屏，是触发锁屏的东西。一开始是xautolock，但是后来发现它太简单了，只能定时锁屏，所以换成了xss-lock。

[Session lock - ArchWiki](https://wiki.archlinux.org/title/Session_lock#systemd_events)看这个，xss-lock默认会订阅suspend事件和lock-session事件。所以，我们什么配置都不加，就可以做出响应。

[Session lock - ArchWiki](https://wiki.archlinux.org/title/Session_lock#D-Bus_notification)再看这里，loginctl这种工具会发送锁屏事件，也就是相当于执行：

```shell
loginctl lock-session
```

然后，看修改电源按钮的方法：[Power management - ArchWiki](https://wiki.archlinux.org/title/Power_management#ACPI_events)。这里介绍了HandlePowerKey就是电源键按下之后的事情。所以我们找到`/etc/systemd/logind.conf`，这样修改：

```conf
[Login]
HandlePowerKey=lock
```

这样你按下电源键，就等于执行了`loginctl lock-session`。接下来，需要处理这个事件，处理就是靠xss-lock了。我们在`~/.xsessionrc`里这么写：

```shell
xss-lock -- slock &
```

这样就是相当于订阅了之前说的那些事件。这样，就可以在盒盖（suspend）和按下电源键（lock）之后都执行slock去锁定屏幕了。

## DWM

彻底配置一遍 Arch Linux + DWM。

### DWM 准备工作

首先，dwm最好用xinit启动，同时一些资源需要读取x的数据库。所以安装：

```
xorg-xinit xorg-xrdb
```

#### 缩放

整体的缩放也要调，就是 100\%, 200\% 之类的。dwm 读取的也是 X Resources 的内容，所以参考：

- [X resources - ArchWiki](https://wiki.archlinux.org/title/X_resources#xinitrc)
- [HiDPI - ArchWiki](https://wiki.archlinux.org/title/HiDPI#X_Resources)

在`~/.Xresources`里加入：

```Xresources
Xft.dpi: 192
```

然后在`~/.xinitrc`里：

```shell
[[ -f ~/.Xresources ]] && xrdb -merge -I$HOME ~/.Xresources
```

#### 自启动

[xinit - ArchWiki](https://wiki.archlinux.org/title/xinit#Autostart_X_at_login)

对于zsh，有一个profile：

- `/etc/zsh/zprofile` Used for executing commands at start for all users, will be read when starting as a _**login shell**_. Please note that on Arch Linux, by default it contains [one line](https://gitlab.archlinux.org/archlinux/packaging/packages/zsh/-/blob/main/zprofile) which sources `/etc/profile`. See warning below before wanting to remove that!
    - `/etc/profile` This file should be sourced by all POSIX sh-compatible shells upon login: it sets up `$PATH` and other environment variables and application-specific (`/etc/profile.d/*.sh`) settings upon login.

在`~/.zprofile`中加入：

```shell
if [ -z "$DISPLAY" ] && [ "$XDG_VTNR" = 1 ]; then
  exec startx
fi
```

> 空格不能省，不然报语法错误；`-z` 表示字符串判空，这里的意思是，环境变量`$DISPLAY`没有定义的时候为true。

设置到这里，还是差一步，我们在login的时候zsh会执行`.zprofile`的内容，从而调用`startx`，这个时候就会执行`.xinitrc`里的内容，导入rdb的资源。最后还差的就是真正启动dwm：

```shell
exec dwm
```

#### 多显示器

多显示器：[xrandr - ArchWiki](https://wiki.archlinux.org/title/xrandr)。我用的前端是arandr。

#### 状态栏

参考[suckless](https://dwm.suckless.org/tutorial/)，状态栏可以用xsetroot修改，所以需要安装`xorg-xsetroot`。显示电量并且修改的话参考[Advanced Linux Sound Architecture - ArchWiki](https://wiki.archlinux.org/title/Advanced_Linux_Sound_Architecture)，安装`alsa-utils`工具。在`~/.xinitrc`里加入这个脚本，这样在startx之后会执行：

```shell
#!/bin/bash
# Taken from:
#	https://raw.github.com/kaihendry/Kai-s--HOME/master/.xinitrc
#

xrdb -merge $HOME/.Xresources

while true
do
	VOL=$(amixer get Master | tail -1 | sed 's/.*\[\([0-9]*%\)\].*/\1/')
	LOCALTIME=$(date "%H:%M +%Y-%m-%d")
	OTHERTIME=$(TZ=Europe/London date +%Z\=%H:%M)
	IP=$(for i in `ip r`; do echo $i; done | grep -A 1 src | tail -n1) # can get confused if you use vmware
	TEMP="$(($(cat /sys/class/thermal/thermal_zone0/temp) / 1000))C"

	if acpi -a | grep off-line > /dev/null
	then
		BAT="Bat. $(acpi -b | awk '{ print $4 " " $5 }' | tr -d ',')"
		xsetroot -name "$BAT $VOL $TEMP $LOCALTIME"
	else
		xsetroot -name "$VOL $TEMP $LOCALTIME"
	fi
	sleep 20s
done &

exec dwm
```

> 该脚本改编自[suckless tutorial最后的Status](https://dwm.suckless.org/tutorial/)里面的[xinitrc](https://dwm.suckless.org/tutorial/xinitrc.example)。

当然，上面只是一个实例，我自己优化之后的版本：

```shell
#!/bin/bash
# Taken from:
#	https://raw.github.com/kaihendry/Kai-s--HOME/master/.xinitrc
#

xrdb -merge $HOME/.Xresources

while true
do
	VOL="🔈 $(amixer get Master | tail -1 | sed 's/.*\[\([0-9]*%\)\].*/\1/')"
	LOCALTIME=$(date "+%H:%M %Y-%m-%d") 
	OTHERTIME=$(TZ=Europe/London date +%Z\=%H:%M)
	IP=$(for i in `ip r`; do echo $i; done | grep -A 1 src | tail -n1) # can get confused if you use vmware
	TEMP="$(($(cat /sys/class/thermal/thermal_zone0/temp) / 1000))C"

	if [[ $(acpi -a | awk '{ print $3 }') = "on-line" ]]; then
		BATPRE="🔌"
	else
		BATPRE="🔋"
	fi
	BAT="$BATPRE $(acpi -b | awk '{ print $4 }' | tr -d ',')"
	xsetroot -name "$BAT | $VOL | $LOCALTIME"
	sleep 20s
done &

exec dwm
```

### DWM 源码修改

字体，在`config.h`里修改：

```c
static const char *fonts[] = { "Terminus (TTF):size=18" };
static const char dmenufont[] = "Terminus (TTF):size=18";
```

st的字体也需要单独设置，默认给的pixelsize，改成size才是跟随缩放的：

```c
/*
 * appearance
 *
 * font: see http://freedesktop.org/software/fontconfig/fontconfig-user.html
 */
static char *font = "Terminus (TTF):size=12:antialias=true:autohint=true";
```

### My Steps

1. 安装archlinux
2. 执行安装yay
3. 执行`yay-script-dwm-base.sh`
4. 安装dwm（现在是spreadwm）, st（现在是[[#Alacritty|alacritty]]）, dmenu，slstatus，slock（这三个已经都整合到spreaddwm里了）
5. 设置.xinitrc, .Xresources保证启动和dpi缩放
6. 执行`yay-script-dwm-font.sh`，安装字体
7. 安装`microsoft-edge-stable-bin`
8. 安装`v2raya`, `v2ray`，[[#Proxy代理|开梯子]]
9. 安装zsh，并设置默认shell为zsh
10. 执行`oh-my-zsh.sh`，配置终端
11. 执行`yay-script-dwm-software.sh`，安装常用软件
12. `udisk2`提供的命令（udiskctl）用来挂载硬盘比较好
13. 安装`davfs2`，按照[wiki](https://wiki.archlinux.org/title/Davfs2#Using_fstab)里去配置fstab，这个配置了每次登录都会自动mount
14. ~~设置状态栏~~（slstatus已经取代之）
15. 执行`yay-script-fcitx.sh`安装输入法并配置（参考[[#中文]]）
16. [设置时区](https://wiki.archlinux.org/title/System_time#Time_zone)
17. 安装[Dunst - ArchWiki (archlinux.org)](https://wiki.archlinux.org/title/Dunst)，接收通知，并加入`~/.xinitrc`中。

## Xrandr

arandr的设置如何保存：[3.2Configuration using arandr](https://wiki.archlinux.org/title/Xrandr#Configuration_using_arandr)

# Awesome Software

[[Knowledge/software_qa/2024-08-09-software-qa|2024-08-09-software-qa]]

## Trouble Shooting

### Flameshot pin

flameshot 的 pin 不工作：[Flameshot PIN feature doesn't work · Issue #2598 · flameshot-org/flameshot (github.com)](https://github.com/flameshot-org/flameshot/issues/2598)。这是因为flameshot需要先启动，然后才能gui。看这个：[Flameshot - ArchWiki (archlinux.org)](https://wiki.archlinux.org/title/Flameshot#Sub-commands_exit_immediately_with_no_output)。另外，issues里开发者说，不会为了这些用户去做这个case的适配（emm，很现实。。。）。

### Accidentally delete /etc/ files

不小心把`/etc/alsa/conf.d`给删了，最后用[how to reinstall all packages in the system? / Pacman & Package Upgrade Issues / Arch Linux Forums](https://bbs.archlinux.org/viewtopic.php?id=34832)里的方法给找回来了。这里记录一下，这个文件是`pipewire-alsa`和`pipewire-audio`拥有的。

### Restore xmodmap

记录一下键盘。之前本来想设置按键调节音量，根据acpid的wiki和一大堆东西好不容易搞好了，这个过程中不小心动了`~/.Xmodmap`。之后左右键被搞没了。然后我本来想用`sudo showkey`来检测，后来发现，`showkey`展示的keycode根本就是错的！`xev`才是对的。这才排查出来之前的左右键已经被当成音量控制按键设置为空了。最后，根据[keyboard - How do I clear xmodmap settings? - Ask Ubuntu](https://askubuntu.com/questions/29603/how-do-i-clear-xmodmap-settings)的说法，执行：

```shell
setxkbmap -layout us
```

就设置会默认的US布局了。

### Cannot mount WebDAV

很傻逼的一个东西，我使用：

```shell
sudo mount -t davfs http://path/to/my/synology/nas:<port>
```

去挂载我的群晖，用的http协议，然后挂载失败了。但是如果用https协议去挂载就可以。错误显示：

```shell
❯ sudo mount -t davfs http://spreadzhao.synology.me:10114
mount.davfs: Mounting failed.
Could not read status line: connection was closed by server
```

后来偶然间发现，只要我把代理关了，就可以挂载了。所以弄了一下，发现v2raya设置成这样就能开代理挂载了：

![[Knowledge/software_qa/resources/Pasted image 20240520233700.png]]

### idea on dwm

[idea-dwm_sudo pacman -s wmname-CSDN博客](https://blog.csdn.net/u010563350/article/details/104948256)

- 安装wmname

```shell
export _JAVA_AWT_WM_NONREPARENTING=1
export AWT_TOOLKIT=MToolkit
wmname LG3D
```