# Ruijie-SUSE-Openwrt-Login
四川轻化工大学宜宾校区锐捷客户端自动登录插件 

![image](https://github.com/user-attachments/assets/17bdc081-90bf-49cb-a91d-fd00cb5fdc56)


## 这个项目有什么作用

众所周知的一点是由于校园网的原因，每个校园网账号只能登录一个台式设备，正常情况下如果我有一台台式【电脑】放在实验室，同时我还希望放一台【服务器】在实验室，\
我希望可以本地直接从电脑访问服务器，那么就需要一个路由器，但是由于是正常的路由器设备，基于校园网只能登录同时登录一个设备的前提，那么电脑和服务器一起访问互联网的话就会直接被校园网踢下线，\
因此需要所谓的·软路由技术·来让所有经过路由器的流量都进行一次伪装，使得校园网认为路由器后面连接的所有设备实际上都是一个设备，本项目就可以做到这一点，适合有服务器的实验室但不想搭建FRP或者因为FRP带宽太低而想在本地搭建局域网的同学。

## 实现原理

基本原理非常简单，就是通过openwrt+自己定义的脚本来实现，openwrt是一个用于路由器的操作系统，实际上就是一个linux系统，\
传统的小米、华为等路由器厂商也有自己的路由器操作系统，但是这些操作系统基本没法执行自己的脚本，因此不适合用来做我们这个项目，传统厂商的操作系统的优势在于有硬件加速，\
而openwrt会损失一部分速度，因为他没有专用的NAT加速芯片，而是通过CPU来进行数据包转发，但优势是他是开源的，并且还有非常多的插件，本项目中就是用到了LUCI这个插件，使得路由器可以自动登录、自动流量转发。

## 前期准备

[Xshll工具](https://www.xshell.com/zh/free-for-home-school)\
[我为你准备的工具压缩包 ](https://pan.baidu.com/s/19_cwpLHE86uUWZdano5S_w?pwd=szsq)
提取码为 szsq

## 如何使用


### 特别注意：本文采用的路由器设备是贝尔0326GMP也就是Nokia EA0326GMP 如果采用其他路由器 请自行进行刷机

在上面的工具压缩包链接中，其中包含了贝尔0326GMP编译好了的openwrt操作系统包、WinSCP、\
贝尔326GMP的uboot刷机包（uboot的作用是为了防止我们的openwrt刷机中途万一出问题不会变成板砖，也方便以后有一天你把路由器恢复成原始的官方操作系统从而变回一个普通路由器）、\
SSH开启工具，我把它打包成了一个压缩包，请一次性下载。

### 正式开始

#### 打开SSH
首先把路由器开机，先不要把光猫连接到路由器（简单来说就是不要插光猫到路由器Wan口的网线），然后用一根网线插到路由器的Lan口并把另一端插到电脑上，然后在浏览器输入: 192.168.10.1，用户名是user，密码是:【你路由器背面写的默认终端密码】。

在-系统管理-备份和恢复-选择文件-找到本项目(解压后文件夹)中的 EA0326GMP_SSH.tar.gz 文件，点击恢复

导入后设备会重启，大概3分钟左右后设备重启完成

#### 安装 uboot 上传到路由器

打开 WinSCP 工具，在左边栏选择 Scp 协议，在右边栏输入路由器的 IP 地址(192.168.10.1)，用户名 root，密码是空\
把 mt7981-nokia-ea0326gmp-fip-expand.bin 上传到路由器的 /tmp/ 目录 \
然后打开Xshll工具，同样的输入路由器的IP地址，用户名root，密码为空，登录进去\
然后输入`cat /proc/mtd`，你应该会看到以下的输出：
```shell
dev:    size   erasesize  name
mtd0: 00100000 00020000 "bl2"
mtd1: 00080000 00020000 "u-boot-env"
mtd2: 00200000 00020000 "factory"
mtd3: 00200000 00020000 "FIP"
mtd4: 00200000 00020000 "config"
mtd5: 00200000 00020000 "config2"
mtd6: 07680000 00020000 "ubi"
```
注意mtd3哪一行最后的一个FIP，这是我们要写入的分区，有可能你那里是fip或者其他名字，请记住这个名字等下要用，默认应该都是fip或者FIP
然后执行下面这个命令：
```shell
mtd write /tmp/mt7981-nokia-ea0326gmp-fip-expand.bin FIP 
```
注意此处的FIP是上面我们提到的哪个名称，可能是fip也可能是FIP或者其他，此处务必填写你在上面执行cat /proc/mtd时候看到的名称，否则会变成板砖。
##### 进入uboot
然后执行以下步骤，建议看清楚了再去操作：\
- 关机拔掉电源
- 用牙签顶住黑色的 reset 键，然后插上电源，然后开机
- 等待 5 秒后，电源灯会闪烁三下，第三下闪烁之后，松开 reset 键，按住时间太长或者太短都无法进入 uboot，请注意观察电源灯闪烁
- 回到电脑输入 【192.168.1.1】，就可以看见 uboot 的界面了，此处IP地址变成了1.1而不是10.1了请注意

然后你将会看到这么一个界面：\
![image](https://github.com/user-attachments/assets/d0655870-5006-462a-b245-f9a67e919398)
然后点击选择文件，选择工具包中的`nokia_ea0326gmp-all.bin`文件，然后upload，等待上传到100%以后,点击出现的upgrade 然后等待路由器重启，这一步大概需要2分钟，可以先去玩玩手机。

等路由器重启完成后在浏览器输入: 【192.168.1.1】，

用户名和密码都是root,后期可以自己改，进去以后，找到左边的网络，下面的SUSELogin插件，然后填写自己的校园网账号密码以及选择好自己的运营上，修改一下间隔时间为1，
![image](https://github.com/user-attachments/assets/8572e9f8-fa47-49d4-8e2c-c388bcce76fc)
然后点击右下角的保存应用设置，等待路由器1分钟后开始自动认证，如果在【日志】中看到以下界面则说明认证成功，认证成功后请把间隔时间修改为10，否则日志会过于频繁。
![image](https://github.com/user-attachments/assets/c5df744e-8bfd-41f7-b876-8f5a08df4a8d)

此插件就算断网以后也会按照间隔时间去自动的认证，因此不用担心第二天起来要认证。但需要注意的是如果你在路由器以外的地方比如宿舍等地方登陆了自己的校园网，这种顶号行为会导致路由器下线，需要重新认证。



#### 本文的插件来源 

脚本代码是来自SUSE以前的前辈做的Luci插件：[blackyau](https://github.com/blackyau/luci-app-suselogin)，我已经直接帮你写好到了openwrt系统中，前辈的固件不太容易买到，贝尔0326GMP则随便买到而且还很便宜，也是全千兆口的，甚至还搭载的是MT7981B的芯片，性能比较强，有一个WAN口三个LAN口

