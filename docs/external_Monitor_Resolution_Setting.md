执行以下命令为外接显示器更改合适的分辨率：

```shell
xrandr ## 显示所有连接屏幕设备
cvt/gtf 1980 1080 60 ## 计算可行的mode/resolution
xrandr --newmode "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync   ## 定义新的mode
xrandr --addmode DP-2 1920x1080_60.00  ## 为外接显示器添加可选mode
xrandr --output DP-2 --mode 1920x1080_60.00  ## 让显示器输出该mode
```

xrandr 命令只能在当前会话下有效，重启后就没用来。所以可以配置一个脚本每次开机后都运行（注意添加excute权限）。

或者还可以在如下的目录添加配置文件，使得开机自动做出更改：

- /usr/share/X11/xorg.conf.d/10-monitor.conf

```text
Section "Monitor"
    Identifier "External lab monitor ACER V246HL"
    Modeline "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
    Option "PreferredMode" "1920x1080_60.00"
EndSection
Section "Screen"
    Identifier "Screen2"
    Device "DP-2"
    Monitor "External lab monitor ACER V246HL"
    DefaultDepth 24
    SubSection "Display"
        Depth 24
        Modes "1920x1080_60.00"
    EndSubSection
EndSection
```

- /etc/X11/xorg.conf.d 

```shell
kristol@sp:~> ls /etc/X11/xorg.conf.d 
00-keyboard.conf  10-quirks.conf    50-device.conf      50-monitor.conf  70-wacom.conf
10-evdev.conf     11-evdev.conf     50-elotouch.conf    50-screen.conf
10-libvnc.conf    40-libinput.conf  50-extensions.conf  70-vmmouse.conf
```

（由于并不是一直连接这个外接显示器工作，所以暂时使用第一种方法，每次执行脚本来实现适应）