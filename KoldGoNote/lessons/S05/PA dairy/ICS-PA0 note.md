这学期选了大名鼎鼎的 ICS，在刘杰老师班做 RISC-V 的 PA 实验。写一篇博客，来记录踩过的坑和心路历程。

PA 实验手册可参考：[PA 实验手册](https://nju-projectn.github.io/ics-pa-gitbook/ics2024/)

PA 0 主要是环境的安装和配置，我采取的方案是 MacOS 用 Windows App 连接阿里云服务器 (Ubuntu 22.04) 做实验。本来以为在*计算机网络*课上完之后，对配置远程连接已经不在话下。但是还是遇到了**一些问题**。


# ICS PA0 配置日记

## 1. 环境准备

- 云服务器：阿里云 Ubuntu 20.04
    
- 用户：`kasumi`（非 root，避免直接用 root 破坏环境）
	- 之前的计网实验我都是用 root 账户做的，还沾沾自喜不用 sudo 真方便，现在看来是**隐患极大**的...
- 桌面环境：xfce4 + xrdp
	- 配置远程桌面 GUI，是为后面的 PA 需要一些图形化内容做准备。这个方案我的 Windows 和 Mac 都可以很方便访问阿里云的 Ubuntu，很方便我做实验~~和玩耍~~
- 目标：能跑 `make menuconfig`，配置并启动 PA 框架。
    

---

## 2. 安装和基本配置

### （1）依赖安装

```bash
sudo apt update
sudo apt install build-essential bison flex libreadline-dev gdb git \
                 libx11-dev libxext-dev libxft-dev
```

### （2）桌面环境 + 远程

```bash
sudo apt install xfce4 xfce4-terminal xrdp
```

- 修改 `/etc/xrdp/startwm.sh`，最后几行改成：
```bash
. $HOME/.xsessionrc
startxfce4
```

我最终使用的大概是

```bash
	1 #!/bin/sh                           2 # xrdp X session start script (c) 2015, 2017, 2021 mirabilos
    3 # published under The MirOS Licence
    4 
    5 # Rely on /etc/pam.d/xrdp-sesman using pam_env to load both
    6 # /etc/environment and /etc/default/locale to initialise the
    7 # locale and the user environment properly.
    8 
    9 if test -r /etc/profile; then
   10     . /etc/profile
   11 fi
   12 
   13 # test -x /etc/X11/Xsession && exec /etc/X11/Xsession
   14 # exec /bin/sh /etc/X11/Xsession
   15 
   16 exec /home/kasumi/.xsession
```

---

## 3. 第一次坑：`make menuconfig` 报 `.config does not exist!`

- 解决：这是正常提示，只要执行一次 `make menuconfig` 生成 `.config` 就行。
    

---

## 4. 第二次坑：`Gtk-WARNING **: cannot open display`

### 原因

- `DISPLAY` 没有配置好
- DBus 没有启动，`DBUS_SESSION_BUS_ADDRESS` 环境变量为空
	- 我在尝试通过远程桌面启动桌面端应用时候，通过指令 `sudo journalctl -u xrdp -e` 查看错误日志发现的这个错误。

### 解决方案

在 `~/.xsessionrc` 写入：
```bash
#!/bin/bash

# 设置运行目录
export XDG_RUNTIME_DIR="/run/user/$(id -u)"

# 自动启动 dbus
if [ -z "$DBUS_SESSION_BUS_ADDRESS" ]; then
    eval "$(dbus-launch --sh-syntax --exit-with-session)"
    export DBUS_SESSION_BUS_ADDRESS
fi

# 设置 DISPLAY（一般 xrdp 会自动分配，不写也行）
export DISPLAY=:10 # 这里是看日志查看的你的display
```

然后重新登录 xrdp 会话。  
这一步解决了 **xfce4-terminal 打不开** 的问题。

---

## 5. 第三次坑：`fail to load $DBUS_SESSION_BUS_ADDRESS`

### 原因

- 没有在 `.xsessionrc` 里启动 `dbus-launch`
    
- 或者 `xrdp` session 和 `dbus` session 不在同一个作用域
    

### 解决方案

同样通过 `.xsessionrc` 写自动启动脚本（见上）。  
重新登录之后再用：

```bash
echo $DBUS_SESSION_BUS_ADDRESS
```

能看到 `unix:abstract=/tmp/dbus-xxxx` 这样的路径，就说明成功了。

---

## 6. 第四次坑：`.xsession` vs `.xsessionrc`

- `.xsession`：定义启动桌面会话执行的主程序，比如 `exec startxfce4`
    
- `.xsessionrc`：放环境变量配置（DBus、XDG、DISPLAY 等），会在 `.xsession` 之前加载
    
- 正确做法：**不要把 dbus 配置写到 `.xsession`，应该写在 `.xsessionrc`**
    

---

## 7. 最终状态检查

```bash
echo $DBUS_SESSION_BUS_ADDRESS
# 输出: unix:abstract=/tmp/dbus-xxxx...

echo $XDG_RUNTIME_DIR
# 输出: /run/user/1000

echo $DISPLAY
# 输出: :10 (或 :11)
```

能正常打开 `xfce4-terminal`，`make menuconfig` 也能弹出界面，说明环境搞定。

---

## 总结

这次 PA0 配置踩的几个关键坑：

1. **menuconfig 缺 .config** → 实际是正常情况
2. **cannot open display** → 缺少 DISPLAY 和 DBus
3. **DBUS_SESSION_BUS_ADDRESS 丢失** → 需要在 `.xsessionrc` 启动 dbus
4. **混淆 .xsession 和 .xsessionrc** → 前者执行桌面，后者放环境变量
    


