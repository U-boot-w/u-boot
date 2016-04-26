# 系统移植

origen
BSP：
安卓是文件系统

NTFS
fat32-->单文件不超过4g

irom

ELF
LSB ：小段

1.交叉开发
    在主机上编译程序，生成的可执行的程序必须是arm格式。  <br/>
    > 通过（file 可执行程序的名字），生成arm格式,在目标（fs4412）运行程序

1.1如何值主机上安装交叉编译器
    1. 把编译器导出为全局变量
        + /etc/priofile -->当前终端下有效
        + /etc/bash.bashrc -->在这个配置文件下添加
            - sudo vi /etc/bash.bashrc
            - 在最后一行添加：export PATH=$PATH:/home/linux/gcc-4.6.4/bin
            - 保存退出
            - source /etc/bash.bashrc

            >测试：在终端下输入arm-n tab减 看自动补齐
                 + /etc/enviroment -->存放系统的配置文件（导出的环境变量）

1.2挂载
    1. 挂载ip --> sudo mount -t nfs 192.168.X.X:/rootfs 挂载点
    2. 查看 挂载信息 --> df
    3. 取消卸载 --> sudo umount 挂载点

1.4.uboot命令的介绍
    help  查看帮助信息
    printenv 打印环境变量
    baudrate=115200 设置串口的波特率
    
> boorargs=root=/dev/nfs nfsroot=192.168.2.146:/source/fs4412_rootfs init=/linuxrc rw ip=192.168.2.158 console=ttySAC2,115200  <br/>
> bootargs=root=/dev/nfs nfsroot=192.168.1.200:/rootfs/rootfs init=/linuxrc rw ip=192.168.1.123 console=ttySAC2,115200  <br/>
> bootdelay=3 延时时间  <br/>
> ethact=dm9000  <br/>
> ethaddr=11:22:33:44:55:66  <br/>
> fileaddr=41000000  <br/>
> filesize=2E3448  <br/>
> fserverip=192.168.1.200  <br/>
> gatewayip=192.168.2.134  <br/>
> ipaddr=192.168.1.123  <br/>
> netmask=255.255.255.0  <br/>
> serverip=192.168.1.200  <br/>
> stderr=serial  <br/>
> stdin=serial  <br/>
> stdout=serial  <br/>

>1. md 查看内存的地址  <br/>
>2. mm 修改内存的地址，地址会递增  <br/>
>3. nm 修改内存的地址，地址不递增  <br/>


**pc机的启动过程**
    1. BIOS程序默认已经随主板出厂时已经内置。
    操作操作系统在硬盘上，内存在掉电时无作用.

    2. pc上电之后，执行的BIOS程序，BIOS程序负责初始化内存，初始化
    硬盘，然后从硬盘上os镜像读取到内存中运行，然后跳转到内存
    中去，执行os直到启动。

- 嵌入式linux系统启动过程：
    uboot程序部署在flash（emmc），os系统部署在flash上。
    启动过程：
     >  uboot负责初始化内存，初始化flash，然后将os从flash中读取到内存中运行,然后启动os。

**uboot到底是干嘛的？**
    1. uboot主要作用是用来启动操作系统。
    2. uboot初始化内存和flash。
    3. uboot提供了命令行的界面。
**bootloader**
    - uboot是属于bootloader的一一种，uboot通用的引导程序，uboot可以移植到不同的开发板上。
**如何烧写uboot：**

         1. 将sd卡插入到电脑，让ubuntu识别sd卡。当我插上sd卡之后，在装有ubuntu的系统界面弹出一个对话框，显示可移动设备，点击确定。
         2. 在vmware player 软件的右上角有sd卡存储设备的图标，点击右键，选择连接。图标变成高亮。sd卡就被ubuntu识别了.
         3. 对sd进行分区
         sudo ./mkuboot.sh  /dev/sdb
         4. 在sd卡下创建sdupdate
         将uboot-fs4412.bin  拷贝到这个目录。
         5. 将薄码开关拨到sd卡启动



**bootcmd：**  <br/>

当uboot启动时，如果在倒计时结束之前按下任意键，uboot就不会执行这个命令，如果在倒计时结束之前没有按下任意键，uboot就会执行这个命令。

         	     bootcmd=tftp 41000000 uImage;tftp 42000000 exynos4412-fs4412.dtb;bootm 41000000 - 42000000

- 将uImage加载到内存的4100000这个地址将exynos4412-fs4412.dtb 加载到42000000地址（设备树）

         	     setenv bootcmd  tftp  41000000  uImage  \;tftp   42000000  exynos4412-fs4412.dtb  \;bootm  41000000 - 42000000
         	     saveenv


**总结：**
- 如果emmc中的uboot是2013.01的版本只需要从这一步开始往下进行：

1. 将串口和开发板相连接，网线必须必须必须必须必须必须插上。
2. 启动开发板。
3. 设置开发板的ip地址和ubuntu的ip地址。

*uboot:*

```c
setenv  ipaddr  192.168.1.100 --> 开发板的ip地址
setenv  serverip  192.168.1.200 --> ubuntu的ip地址
saveen --> 保存环境变量的信息

ping  192.168.1.200 --> 如果能ping通

host  192.168.1.200  alive;
```                     
**如果出现TTTTTTTTTTT**

### 解决方法：

1. 网线没插，插网线
2. ping   ubuntu的ip地址如果能ping通，但是还是出现TTTTT
- 将tftp服务重新启动

    sudo  /etc/init.d/tftpd-hpa restart  <br/>
    sudo  service   tftpd-hpa restart

3. 如果不能ping通？
    - 查看ubuntu的ip地址是否存在，如果不存在
    - 添加ip地址:
        sudo  ifconfig  eth0  192.168.1.200

4. 启动内核
    - ubuntu:
        将uImage  exynos4412-fs4412.dtb  拷贝到ubuntu的/tftpboot
    - uboot:
        setenv  bootcmd tftp 41000000  uImage  \;tftp  42000000  exynos4412-fs4412.dtb \;bootm  41000000 - 42000000
    - saveenv                


5. 重新启动开发板
