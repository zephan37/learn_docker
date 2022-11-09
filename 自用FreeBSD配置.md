# 自用FreeBSD配置，图像界面使用bspwm

## 一、安装

U盘安装没啥好说的，FreeBSD的安装很简单，主要参照如下：

https://docs.freebsd.org/zh-cn/books/handbook/install/

https://book.bsdcn.org/di-er-zhang-an-zhuang-freebsd/di-ling-jie-tu-jie-an-zhuang.html

## 二、网络配置

安装好后开机进系统，为了后续的配置，第一件事情应该是考虑网络的配置。

### 1、网卡

#### （1）列出可用网卡

默认下执行如下命令：

`ifconfig`

此时正常来说应该会显示两张网卡，1是电脑本身的网卡，例如我的是em0，2是lo0，这个不用管他。

#### （2）使用网线

若有网线，可以直接将网线接入，此时应该可以接入网络。若网线接入后还是无法连接网络，可以使用：

`dhclient em0` 使用DHCP分配IP地址，分配后应该将可以接入网络。

为了使系统重启后可以自动使用dhcp分配， 可以在/etc/rc.conf 中增加如下配置：

`ifconfig_em0="DHCP"`

#### （3）手机USB共享网络

若无网线，可以用USB将手机与PC连接后，在手机内打开使用USB共享网络，此时使用ifconfig会发现多处了一个网卡：ue0

与网卡一样，使用 `dhclient ue0` 分配IP后，即可接入网络

### 2、wifi

#### （1）列出无线网卡驱动

运行 `sysctl net.wlan.devices`，他会告诉你的无线网卡驱动，例如我的电脑输出为iwm0。如果冒号输出后边没有东西，那就是识别不了，下面不用看了。

#### （2）加载无线网卡驱动

后编辑/boot/loader.conf使开机加载wifi设备驱动：

```
if_iwm0_load="YES" 
legal.realtek.license_ack=1
```

而后于/etc/rc.conf中加入以下使开机创建wlan0网络并使用DHCP分配IP

```
wlans_iwm0="wlan0"
ifconfig_wlan0="WPA DHCP"
```

重启电脑以使rc.conf生效

重启后运行ifconfig，此时你可以看到wlan0网卡已经可以使用了。

#### （3）使用wifi

搜索wifi

`ifconfig wlan0 up scan`

连接wifi

编辑/etc/wpa_supplicant.conf，加入

```
network={
  ssid="wifi名称"
  psk="wifi密码"
}
```

重启后，你会发现wifi已经自动连接上，并且可以愉快地接入网络了。



任何时候当出现无法接入网络时，使用service netif restart重启网络，一般问题即可解决

## 三、换源

### 1、设置中科大源

虽然现在可以连接网络了，但是使用pkg等安装软件时，速度是在是慢的可怜，此时应该先换源，个人最喜欢使用中科大源：

pkg源设置依据官方指南：

https://mirrors.ustc.edu.cn/help/freebsd-pkg.html

ports源设置依据官方指南：

https://mirrors.ustc.edu.cn/help/freebsd-ports.html

### 2、开启ports

在能使用 ports 之前， 你必须先获得 Ports Collection - 本质上是 **/usr/ports** 目录下的一堆 **Makefile**、 补丁和描述文件。

在安装 FreeBSD 系统的时候， sysinstall 会询问是否需要安装 Ports Collection。 如果选择 no， 那可以用下面的指令来安装 Ports Collection：

```
1、portsnap fetch
2、portsnap extract
3、portsnap update
```



## 三、sudo设置

FreeBSD 基本系统默认不自带 `sudo` 命令，需要使用 `root` 权限自行安装：

`pkg install sudo`



sudo 可以解析 /etc/sudoers.d/ 目录中的文件，且目录中的文件是按字母顺序加载的，. 或 ~ 开头的文件会被跳过。文件名应该以双字母开头，例如 01_foo，

因此在/etc/sudoer.d中建立“00_用户名”的文件，代表这个文件最先被载入，然后在其中输入：

`用户名 ALL=(ALL) ALL` 

意思是让某用户可以执行所以命令



## 四、配置显卡驱动

我的电脑是intel的核显，其他显卡可以参考https://book.bsdcn.org/di-er-zhang-an-zhuang-freebsd/di-jiu-jie-wu-li-ji-xia-xian-ka-de-pei-zhi.html



安装驱动：

`pkg install drm-kmod`



开机加载驱动：

编辑/etc/rc.conf，加入：

`kld_list="i915kms"`



开启硬解：

`pkg install xf86-video-intel libva-intel-driver`



开启亮度调节：

`pkg install intel-backlight`



而后重启



## 五、配置声卡驱动

先加载声卡驱动：

`sysrc snd_hda="YES"`

然后重启。



## 六、安装xorg

`pkg install xorg`

重启



## 七、bspwm与sxhkd

### 1、安装

安装bspwm与sxhkd

`pkg install bspwm`

它会同时安装bspwm与sxhkd



安装alacritty与rofi

`pkg install alacritty`

`pkg install rofi`

### 2、配置

建立配置文件目录：

`mkdir -p .config/bspwm && mkdri -p .config/sxhkd`



复制配置文件到目录内

```
cp /usr/local/share/examples/bspwm/bspwmrc .config/bspwm/
cp /usr/local/share/examples/bspwm/sxhkdrc .config/sxhkd/
```



编辑.config/sxhkd/sxhkdrc设置默认终端与程序启动器

```
# terminal emulator
super + Return
		alacritty
	
# program launcher
super + p
		rofi -show run
```

之后就可以用win+回车键打开终端，使用win+p打开程序启动器



编辑.xinitrc，若不存在则创建

`exec bspwm`



### 3、打开

现在就可以`startx`打开bspwm了



## 八、代理服务器

经过startx进入图像化界面后，对ports的使用需求增加，但是ports使用时速度很慢，因此先配置代理

### v2ray

亲测，在写此文时pkg install v2ray安装的v2ray无法使用，唯一的办法是直接使用二进制可执行文件

#### （1）安装v2ray

v2ray-freebsd-64.zip，该文件可与github v2ray的官方仓库内找到

`https://github.com/v2fly/v2ray-core/releases/download/v5.1.0/v2ray-freebsd-64.zip`

解压与安装：

````
$ mkdir v2ray
$ mv v2ray-freebsd-64.zip v2ray
$ cd v2ray 
$ unzip v2ray-freebsd-64.zip v2ray
$ cd ..
$ sudo mkdir /usr/local/bin/v2ray
$ cp v2ray/* /usr/local/bin/v2ray/
````

#### （2）配置v2ray

将你的v2ray配置文件复制到v2ray文件夹内

`cp config.json /usr/local/bin/v2ray/`

#### （3）启动v2ray

与root下

`/usr/local/bin/v2ray/v2ray run  & `

#### （4）系统设置代理

`export http_proxy="http://127.0.0.1:port"`

`export http_proxy="http://127.0.0.1:port"`

#### （5）设置v2ray开机自启

也可以在`.xinitrc`内加入如下使得开机启用

```
export http_proxy="http://127.0.0.1:port"
export http_proxy="http://127.0.0.1:port"
/usr/local/bin/v2ray/v2ray run  &
```



## 九、本地化

### 1、安装wqy字体

startx进入图像界面后会发现中文是乱码，安装wqy字体解决

```
cd /usr/ports/x11-fonts/wqy
sudo make install
```

因为第八步已经设置了代理，所以这里速度很快

安装好后中文乱码即可解决

### 2、系统本地化设置

设置LANG与MM_CHARSET

我使用的是.xinitrc来加载

```
export MM_CHARSET=ISO-8859-8-I
export LANG=zh_CN.UTF-8
```



## 十、linux兼容层

通过兼容linux，就可以使用一些Linux程序，例如chrome等

### 1、先配置系统自带的centos兼容层

#### （1）开启服务

```
sysrc linux_enable="YES"
sysrc kld_list+="linux linux64"
kldload linux64
pkg install emulators/linux-c7 dbus
service linux start
sysrc dbus_enable="YES"
service dbus start
dbus-uuidgen > /compat/linux/etc/machine-id
reboot
```

#### （2）配置开机挂载

配置fstab开机挂载

以下写入 `/etc/fstab`:

```
linprocfs   /compat/linux/proc	linprocfs	rw	0	0
linsysfs    /compat/linux/sys	linsysfs	rw	0	0
tmpfs    /compat/linux/dev/shm	tmpfs	rw,mode=1777	0	0
```

#### （3）挂载检查

检查挂载会不会出错：

`mount -al`

重启

### 2、安装ubuntu兼容层

注意，必须先配置好系统默认的centos兼容层

一开始，先将`nullfs_load="YES"`写入`/boot/loader.conf

#### （1）构建ubuntu20.04

```
pkg install debootstrap
debootstrap focal /compat/ubuntu http://mirrors.163.com/ubuntu/
reboot
```

#### （2）挂载文件系统

将如下写入到/etc/fstab:

```
devfs           /compat/ubuntu/dev      devfs           rw,late                      0       0
tmpfs           /compat/ubuntu/dev/shm  tmpfs           rw,late,size=1g,mode=1777    0       0
fdescfs         /compat/ubuntu/dev/fd   fdescfs         rw,late,linrdlnk             0       0
linprocfs       /compat/ubuntu/proc     linprocfs       rw,late                      0       0
linsysfs        /compat/ubuntu/sys      linsysfs        rw,late                      0       0
/tmp            /compat/ubuntu/tmp      nullfs          rw,late                      0       0
/home           /compat/ubuntu/home     nullfs          rw,late                      0       0
```

注意以上的

`/tmp            /compat/ubuntu/tmp      nullfs          rw,late                      0       0`

`/home           /compat/ubuntu/home     nullfs          rw,late                      0       0`

很重要，其将FreeBSD的tmp与home映射到Linux内，其中tmp内涉及到硬件的映射，是很关键的，它使得Linux能和主机FreeBSD共享音频和X11套接字

#### （3）检查挂载

然后检查挂载会不会出错：

`mount -al`

重启

### 3、兼容层ubuntu配置

先chroot进入ubuntu系统：

`sudo chroot /compat/ubuntu /bin/bash`

#### （1）换源

按如下换源

https://mirrors.ustc.edu.cn/help/ubuntu.html

#### （2）本地化

配置语言环境和时区：

```
dpkg-reconfigure locales
dpkg-reconfigure tzdata
```

安装中文字体

`apt install fonts-wqy-microhei`

`apt install fonts-wqy-zenhei`

#### （3）声音配置

ubuntu是FreeBSD内的一个子系统，经测试一开始声音无法使用，按如下配置开启声音：



ubuntu先安装pulseaudio

`apt install pulseaudio`



于主机FreeBSD上编辑/usr/local/etc/pulse/system.pa

`load-module module-native-protocol-unix auth-anonymous=1 socket=/tmp/pulse-native`

它告诉如何命名 Pulseaudio 套接字 ( /tmp/pulse-native)。稍后 Linux 兼容层中配置 Pulseaudio 时将需要这些信息。



主机FreeBSD上pulseaudio软件包不附带rc脚本，因此需要我们创建一个，创建文件/usr/local/etc/rc.d/pulseaudio，该脚本用来启动pulseaudio服务

```
#!/bin/sh

# PROVIDE: pulseaudio
# REQUIRE: DAEMON FILESYSTEMS
# KEYWORD: nojail shutdown

. /etc/rc.subr

name="pulseaudio"
desc="Start the Pulseaudio server"
rcvar="pulseaudio_enable"
pulseaudio_bin="/usr/local/bin/${name}"
pulseaudio_pidfile="/var/run/pulse/pid"
start_cmd="${name}_start"
stop_cmd="${name}_stop"
load_rc_config "${name}"

pulseaudio_start()
{
    ${pulseaudio_bin} --system --disallow-module-loading &
}

pulseaudio_stop()
{
    if [ -f "${pulseaudio_pidfile}" ]
    then
        kill $(cat "${pulseaudio_pidfile}")
    fi
}

run_rc_command "$1"
```

注意赋予它可执行权限 `chmod +x /usr/local/etc/rc.d/pulseaudio`

而后让他在开机启动：

编辑/etc/rc.conf

`pulseaudio_enable="YES"`

重启



经如上，pulseaudio启用的条件已经具备，ubuntu子系统只需配置全局变量：

`export PULSE_SERVER=unix:/tmp/pulse-native`即可开启声音

但是当我们使用ubuntu子系统的程序时，总不能每次得先切换到子系统配置全局变量吧，这样很麻烦，于是我们可以在ubuntu子系统上的/etc/bash.bashrc中配置全局变量：

`export PULSE_SERVER=unix:/tmp/pulse-native`

/etc/bash.bashrc文件会在我们使用bash自动配置该全局变量

以后需要给ubuntu子系统配置全局变量时，也可以在此文件配置，例如要配置ubuntu子系统代理时，也可在/etc/bash.bashrc文件中配置：

```
export http_proxy="http://127.0.0.1:port"
export http_proxy="http://127.0.0.1:port"
```

#### （5）添加个人用户

ubuntu子系统默认是root环境下，但以root启动程序会有风险，添加个人用户用以启动程序，例如我这里创建一个linux-zephan的用户

```
adduser linux-zephan
```



最后，于主机FreeBSD上编辑.xinitrc，加入

`xhost +`



以上完成



## 十一、使用Linux兼容层运行Linux应用，以chrome为例

### 1、进入ubuntu环境

`sudo chroot /compat/ubuntu /bin/bash`

### 2、安装google-chrome

`apt install google-chrome.deb`

### 3、在ubuntu子系统中创建chrome启动脚本/bin/FreeBSD-chrome

```
 #!/bin/bash
 export DISPLAY=:0			#ubuntu子系统默认是root环境下，但以root启动程序会有风险。添加此变量是为了后续可以使用个人用户启动该程序
 google-chrome-stable --no-sandbox --no-zygote --in-process-gpu 	#启动chrome
```

### 4、退出ubuntu环境，回到FreeBSD环境，于/usr/local/bin/下创建文件夹linux-app

`mkdir -p /usr/local/bin/linux-app`

### 5、于linux-app文件夹内创建chrome启动脚本linux-chrome-app，如下：

```
chroot /compat/ubuntu /bin/bash -c "su -c /bin/FreeBSD-chrome - linux-zephan"
```

chroot /compat/ubuntu 为切换到ubuntu root的意思，意思是后面的程序是运行于ubuntu下

/bin/bash -c 意思是使用ubuntu的bash

su -c /bin/FreeBSD-chrome - linux-zephan的意思是使用用户linux-zephan来运行脚本/bin/FreeBSD

赋予可执行权限：

`chmod +x /usr/local/bin/linux-app/linux-chrome-app`

此时使用 `sudo /usr/local/bin/linux-app/linux-chrome-app `即可以看到linux兼容层下的chrome被打开了。

### 6、FreeBSD中让个人用户使用linux兼容层的程序

现在使用 `sudo /usr/local/bin/linux-app/linux-chrome-app `即可打开linux兼容层下的chrome了，但是我不想使用root才能运行linux下的程序，因为我使用rofi作为程序启动器，rofi无法启动root程序，因此可以作如下配置：

#### （1）、创建启动脚本

于/usr/local/bin中创建linux chome文件，为了方便区分，该文件使用“linux-程序名”的命名格式，文件中添加如下内容：

`sudo /usr/local/bin/linux-app/linux-chrome-app`

并赋予可执行权限

#### （2）、使启动脚本具有免密启动的权限

在/usr/local/etc/sudoers.d/文件夹下创建01_linux-app文件，并加入以下内容：

`username ALL=(root) NOPASSWD:/usr/local/bin/linux-app/linux-chrome-app`

现在，你应该可以使用linux-chrome命令打开chrome了，享受它吧！

若是以后还想添加其他免密启动程序，直接在01_linux-app内添加就好，例如下：

`username ALL=(root) NOPASSWD:/usr/local/bin/linux-app/linux-chrome-app，/usr/local/bin/linux-qq`



如上是添加一个linux app的过程，若是以后想要添加linux下的兼容层程序，按照如上配置chrome的过程配置即可

## 十二、polybar配置

