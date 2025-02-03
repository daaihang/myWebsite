+++
title = '使用小米路由器 4C 登录西邮联通校园网'
date = 2024-03-11T23:16:39+08:00
+++

使用广东工业大学的OpenWrt刷机包。亲测可以直接在西安邮电大学长安校区使用（联通校园网），并解除联通校园网的设备使用数量限制。适合有多台设备的同学使用。

参考文章：

- ⭐[在Dr.COM下使用路由器上校园网WIFI](https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt)（若配置别的路由器则可以看这个Github项目）

- [开启telnet刷小米路由器](https://www.right.com.cn/forum/thread-4040540-1-1.html)

本教程仅供学习，请在成功刷入OpenWrt后正确使用系统，并在学习后24小时内删除相关固件和（或）软件包。


## 步骤提要

1. 初始化，简单进入系统。

2. 开启telnet和ftp，刷Breed，防止变砖。

3. 刷OpenWrt，配置网络和Dr.com。

## 文件下载

- **MobaXterm** [官网（MobaXterm Xserver）](https://mobaxterm.mobatek.net/download-home-edition.html)，建议下载便携版本（Portable edition）。
- **R3GV2 patches** [Onedrive链接](https://cntk-my.sharepoint.com/:u:/g/personal/599100706_mail_scun_net/EbfqzlpS8U9BhnNBz50mVLQBI98-6XU3NPFG0XlP7vJ63g?e=OZSheQ)。
- **Breed**（小米4C）[下载链接](https://breed.hackpascal.net/breed-mt7688-reset38.bin)，其他路由器查看[这里](https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt#%E4%B8%8B%E8%BD%BDbreed)的表格。
- **OpenWrt**（小米4C）Github[下载链接](https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt/files/8138729/Mi4C.zip)（内置了Dr.com插件及防检测插件）。
- （可选）自行安装OpenWrt其他软件包。

## 前置步骤：进入系统配置密码

在校内无网情况下，路由器需要先配置WIFI密码才能进行后续步骤。否则无法用命令行登录小米路由器。

路由器插入电源，手机或电脑搜索WIFI，找到名为`Xiaomi_****`的无密码WIFI，直接连接。

打开路由器默认配置界面（IP地址：[192.168.31.1](http://192.168.31.1)），跳过拨号配置界面，直接设置WIFI密码（管理员密码默认于此相同，刷入Breed后该密码会被覆盖）。随后进入路由器主页，即可关闭网页。

## 在小米路由器4C上开启telnet和FTP

解压R3GV2 patches包，双击运行`0.start_main.bat`批处理文件，期间需要在cmd窗口输入你的路由器4C的管理员密码。结束后，路由器4C就可以用telnet远程登录和使用FTP上传下载文件了。

> 原理大概是运用程序漏洞提权，获取root权限并开启FTP服务。

## 用telnet登录到小米路由器4C

在MobaXterm里新建一个session，类型telnet，主机地址为`192.168.31.1`，用户为`root`，密码空，就能登录路由器4C了。

![](https://cdn.jsdelivr.net/gh/daaihang/PicGo/202302211455225.png)

![](https://cdn.jsdelivr.net/gh/daaihang/PicGo/202302211458021.png)

![命令行出现root@XiaoQiang即可，图中未显示。](https://cdn.jsdelivr.net/gh/daaihang/PicGo/202302211459646.png)

看到命令提示符是`root@XiaoQiang`即可完成登录。这其实就是Linux下的Shell。

### 生成eeprom备份文件

在MobaXterm的telnet终端窗口键入以下命令并回车。

```shell
dd if=/dev/mtd3 of=/tmp/eeprom.bin
```

理论上讲，用dd命令可以备份路由器4C的所有分区。**最好把所有分区都备份。**

## 使用FTP上传Breed，并下载eeprom备份文件

在Windows上打开资源管理器，在地址栏输入[ftp://192.168.31.1](ftp://192.168.31.1)然后回车，路由器4C的文件系统就出现了。

> FTP服务使用的是游客匿名登录，无需账户和密码，可以直接回车登录。不要使用`root`账户登录。

把之前下载的`breed-mt7688-reset38.bin`改名为`breed.bin`，然后复制到`/tmp`目录内备用。

另外把`/tmp`目录下的`eeprom.bin`文件下载到本地，做好备份。

## Breed的刷入与配置

### 刷入Breed

在MobaXterm的telnet终端窗口键入命令，不出现错误提示信息就是成功了。

```shell
mtd write /tmp/breed.bin Bootloader
```

### 进入Breed

**以下步骤务必正确进行。**

拔掉路由器4C电源，用牙签按住路由器4C的`reset按钮`不松开，插上电源，路由器4C的灯会闪几下，这需要几秒钟，然后松开reset，路由器已进入Breed。

用网线连接电脑和路由器4C的Lan口（局域网网口，而不是接入外部网络的口），在电脑上用浏览器打开[192.168.1.1](http://192.168.1.1)就能看到Breed的网页界面了。

第一次进入Breed，要在Breed里面把前面备份的`eeprom.bin`文件刷回去。进Breed-固件更新，里面可以刷eeprom。

![](https://cdn.jsdelivr.net/gh/daaihang/PicGo/202302211537572.png)

## 刷入OpenWrt固件

现在正式开始刷入OpenWrt固件，依次点击固件更新-勾选固件-点击选择文件，选择我们刚刚下载的`Mi4C.bin`，然后耐心等待固件刷入完成。

![](https://cdn.jsdelivr.net/gh/daaihang/PicGo/202302211539496.png)

![](https://cdn.jsdelivr.net/gh/daaihang/PicGo/202302211540055.png)

安装完成后会自动重启，这时可以不断刷新浏览器，直到管理界面显示出来，如果没有显示，建议稍后使用192.168.1.1访问管理页面。

![](https://cdn.jsdelivr.net/gh/daaihang/PicGo/202302211544294.png)

> 账号:`root`
>
> 密码:默认没有密码或者默认密码为`password`

## OpenWrt配置

### 端口校园网配置

选择左侧**网络-接口**，选择WAN6-编辑。

![](https://cdn.jsdelivr.net/gh/daaihang/PicGo/202302211620750.png)

协议选择`PPPoE`，然后点击出现的 Switch Protocol（切换协议）按钮；PAP/CHAP用户名为学号+`@unicom`，例如`23338080@unicom`；密码为校园网登陆密码。随后点击`保存`，然后点击`保存并应用`。

![image-20230221162500261](https://cdn.jsdelivr.net/gh/daaihang/PicGo/202302211625399.png)

另一个端口`WAN`**不要**进行相同设置，只设置`WAN6`即可。因为不能多个“设备”登录同一个校园网账号。

### 无线网络设置

选择左侧**网络-无线**。如果提示Disabled（已禁用）就点击Enable（启用）。可能只有一个2.4G的，也可能有一个2.4G的、一个5G的。点2.4G或5G的编辑按钮。ESSID填你想要的WiFi名称。

点击Wireless Security（无线安全）。Encryption（加密）改选为`WPA2-PSK`；Key（密码）填你想要的WiFi密码；最后点击`保存并应用`。

### 配置Dr.com插件

按下图进行配置。

点击左侧**网络-接口**，查看`WAN6`的`MAC`地址并复制，修改此处Dr.com的`MAC拨号`的地址。

注意，在接口名称中，不一定选择的是`eth0.2`，而是选择与`WAN6`对应的接口名称，有可能是`eth1`，下图提示有误。

![image-20230221170502631](https://cdn.jsdelivr.net/gh/daaihang/PicGo/202302211706729.png)

在配置保存后，路由器会自动连接。耐心等待后，若可以在页面中看到接发数据，获取到了IP地址，即配置成功。

![](https://cdn.jsdelivr.net/gh/daaihang/PicGo/202302211642791.png)

> **如果发现路由器一直不能上网，则说明:**
>
> - wan中，学号密码输入错误(可能性30%)；
> - drcom插件中，学号密码输入错误(可能性30%)；
> - 路由器的wan没有与校园网端口连接(可能性20%)；
> - 网线断了，或者路由器坏了(可能性15%)；
> - 压根没开通校园网(可能性4.9%)；
> - 端口被学校网络中心拉黑了(极少出现0.1%)。

### （可选操作）配置防检测

具体看这里：[shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt (github.com)](https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt#步骤六配置防检测)

## 大功告成

🎉至此已经可以上网了！开始享受无线设备数量限制的快乐吧！

本教程大体还是参照Github项目[Drcom-GDUT-HC5661A-OpenWrt](https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt)进行配置。其他路由器可参照该项目。

