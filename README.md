# OneCloud2刷Openwrt配置

## 一、路由刷Opnwrt固件后初步设置

1.设置路由密码

2.更新软件包

```bash
opkg update
opkg install resize2fs  #安装resize2fs
```

3.更改分区

```bash
df -h #查看系统分区及使用情况
lsblk

resize2fs /dev/system
df -h
lsblk
```

## 二、挂载外置USB硬盘

1.安装分区软件fdisk

```bash
opkg install fdisk
```

2.插入外置的USB硬碟, 用 fdisk -l 查看, 可找到外置的USB硬碟在/dev/sda 空间为465.78 GiB (即500G)

```bash
fdisk -l

```

3. 删除旧有分区, 建立新的分区并重新格式化为ext4![img](https://www.right.com.cn/forum/data/attachment/forum/202105/14/090152goddatudwfofwwfw.png)

```bash
fdisk /dev/sda
d
2
d
1
n
p
1
w
mkfs.ext4 /dev/sda
y
y
```

4.建立挂载目录

```bash
mkdir -p /mnt/sda1
mount /dev/sda /mnt/sda1
df -h #用mount挂载USB硬碟, 可查看已经挂载成功
```

5.最后编辑 /etc/rc.load 加入开机自启动

```
mount /dev/sda /mnt/sda1/ #在exit 0上一行加入
```

6如果想对NTFS支援读写，就需要安装ntfs-3g (即可免除上面格式化为ext4的动作)，还可以很方便地转到windows读取。

```bash
opkg install ntfs-3g  #
```

## 三、**SMB分享 (Windows 共享目录)**

1.安装samba软件

```bash
smbd -v #检查samba版本
opkg update
opkg install kmod-usb-storage block-mount samba36-server luci-app-samba
opkg install samba36-server luci-app-samba
```

2.到浏览器主页Network Shares设置

## ![img](https://www.right.com.cn/forum/data/attachment/forum/202105/14/090427xv6pvwcwnzvbpp0e.png)



## 四、filebrowser

1.登陆SSH及安装wget作准备

```bash
opkg update && opkg upgrade wget && opkg install ca-certificates
```

2.下载filebrowser

```bash
wget https://github.com/filebrowser/filebrowser/releases/download/v2.15.0/linux-armv7-filebrowser.tar.gz
wget https://github.91chi.fun/https://github.com//filebrowser/filebrowser/releases/download/v2.21.1/linux-armv7-filebrowser.tar.gz
wget https://github.com/filebrowser/filebrowser/releases/download/v2.21.1/linux-armv7-filebrowser.tar.gz
```

3.解压缩filebrowser

```
chmod 777 linux-armv7-filebrowser.tar.gz
tar xzf linux-armv7-filebrowser.tar.gz
```

4.将filebrowser 二进制文件移动到执行目录 及 执行filebrowser 测试

```bash
mv filebrowser /usr/bin/
filebrowser -a 0.0.0.0 -p 8080 -r /  
#filebrowser -a 0.0.0.0 -p 8080 -r /
-a 0.0.0.0 监听地址即所有
-p 8080 监听端口
-r / 管理目录 / 为所有
```

5.最后编辑 /etc/rc.load 加入开机自启动

```bash
filebrowser -a 0.0.0.0 -p 8080 -r / #在exit 0上一行加入
```

## 五、**aria2 (强大又轻巧的下载工具)**

1.SSH安装aria2及建立下载目录

```
opkg install aria2 luci-app-aria2
mkdir -p /mnt/sda1/aria2
```

2.配置aria2![img](https://www.right.com.cn/forum/data/attachment/forum/202105/14/090722id5x5cwwwfx5wcxj.png)

## 六、**ttyd**

```
opkg update && opkg install ttyd
```

2.运行及测试

```
ttyd -p 7681 login &
```

3.最后编辑 /etc/rc.load 加入开机自启动

```
ttyd -p 7681 login &  #在exit 0上一行加入
```

## 七、**简单导航页面**

1.将导航页面文件解压缩到SMB的Public分享目录

2.用记事本修改**index.html**内容的IP为自己赚钱宝的IP地址.

![img](https://www.right.com.cn/forum/data/attachment/forum/202105/14/091115gs0nno60qeje66qv.png)

3.在SSH查看目录路径是否正确及 在/www/目录建立软连结

```
ll /mnt/sda/www/dash/
ln -s /mnt/sda1/www/dash/ /www/

```

4.如之前安装aria2了, 也可一并建立Aria-Ng软连结.

```
ln -s /mnt/sda1/www/dash/Ng-Aria/ /www/
```

5.在浏览器输入赚钱宝的IP地址/dash/就可以开启导航页面

## 八、**CIFS 用户端**

1.SSH进入及安装

```
opkg install kmod-fs-cifs kmod-nls-base kmod-nls-utf8 kmod-crypto-hmac kmod-crypto-md5 kmod-crypto-misc cifsmount
```

2.挂载我NAS SMB分享目录![img](https://www.right.com.cn/forum/data/attachment/forum/202105/14/091158hcrqqlydjprqjl8s.png)

```bash
mount -t cifs //20.13.3.250/Backup /mnt/cifs -o username=guest,vers=1.0
df -h #username=guest 是因为设定了免密码连线
mount -t cifs //20.13.3.250/Backup /mnt/cifs -o username=admin,password=admin,vers=1.0 #username=使用者帐号
password=密码
```

3.最后编辑 /etc/rc.load 加入开机自启动

```bash
mount -t cifs //20.13.3.250/Backup /mnt/cifs -o username=guest,rw,vers=1.0  #在exit 0上一行加入
```

## 九、**NFS 用户端**

1.SSH登入及安装NFS 用户端

```
opkg update && opkg install nfs-utils kmod-fs-nfs kmod-fs-nfs-v4 kmod-fs-nfs-v3
```

2.DSM的NFS伺服器设定

![img](https://www.right.com.cn/forum/data/attachment/forum/202105/14/091251gkr7fvaxd6sfaakk.png)

3.挂载及测试

```bsah
mount -t nfs 20.13.3.13:/volume1/Public /mnt/nfs/ -o nolock
df -h
```

4.最后编辑 /etc/rc.load 加入开机自启动

```bash
mount -t nfs 20.13.3.13:/volume1/Public /mnt/nfs/ -o nolock  #mount -t nfs 20.13.3.13:/volume1/Public /mnt/nfs/ -o nolock
```

十、**AdGuard Home (官方脚本傻瓜安装)**

1.SSH登入, 执行官方脚本.

```
opkg install curl
curl -sSL https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh
```

```bash
wget https://static.adguard.com/adguardhome/edge/AdGuardHome_linux_armv7.tar.gz -O AdGuardHome.tar.gz

tar xvf AdGuardHome.tar.gz
mkdir /usr/local/AdGuard_Home

#移动文件
mkdir /usr/local/AdGuard_Home
mv AdGuardHome/AdGuardHome /usr/local/AdGuard_Home



```

**安装 AdGuard Home 到系统中**

```
cd /usr/local/AdGuard_Home
./AdGuardHome --service install
```

2.浏览器输入**赚钱宝的IP地址+端口3000** 开启ADGUARD设定

## 十一、**京东签到插件**

1.SSH登入安装依赖

```
opkg update
opkg install node wget lua
```

2.下载ipk 安装京东签到插件

```
wget --no-check-certificate https://github.com/jerrykuku/luci-app-jd-dailybonus/releases/download/v1.0.5/luci-app-jd-dailybonus_1.0.5-20210316_all.ipk
```

## 十二、**特别的小猫咪外游软件**

1、SSH登入, 安装依赖

```
opkg update && opkg install luciopkg luci-baseopkg iptablesopkg dnsmasq-fullopkg coreutilsopkg coreutils-nohupopkg bashopkg  curlopkg jsonfilteropkg ca-certificatesopkg ipsetopkg ip-fullopkg iptables-mod-tproxyopkg kmod-tunopkg luci-compat
```

2.下载及安装

```
wget --no-check-certificate https://github.com/vernesong/OpenClash/releases/download/v0.42.05-beta/luci-app-openclash_0.42.05-beta_all.ipk
opkg install luci-app-openclash_0.42.05-beta_all.ipk
```

3.下载 [Clash内核](https://github.com/vernesong/OpenClash/releases/tag/Clash)

```
wget --no-check-certificate https://github.com/vernesong/OpenClash/releases/download/Clash/clash-linux-armv7.tar.gz
```

4.解压到 /etc/openclash/core/文件夹

```
tar xzf clash-linux-armv7.tar.gz
mv clash /etc/opnclash/core/
ll /etc/openclash/co
ll /etc/openclash/core/
```

5.安装插件遇到Collected errors, 可根据内容移除dnsmasq再重新安装

```
opkg remove dnsmasq
opkg install luci-app-opnclash_0.42.05-beta_all.ipk
opkg install luci-app-opnclash_0.42.05-beta_all.ipk
```

![img](https://www.right.com.cn/forum/data/attachment/forum/202105/14/091752wc8lqwxppeonsrvs.png)

![img](https://www.right.com.cn/forum/data/attachment/forum/202105/14/091752ap4rn90ru7p1pp0s.png)

![img](https://www.right.com.cn/forum/data/attachment/forum/202105/14/091752ufvph9p105iz51vu.png)

![img](https://www.right.com.cn/forum/data/attachment/forum/202105/14/091752ngns8n8nmm74nicb.png)





# OpenWrt 更新所有已安装软件包命令