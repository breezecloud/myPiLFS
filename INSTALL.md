#树莓派操作系统彻底定制
##一、为什么要用树莓派
	文章名字由来于网上有人对LFS项目的翻译，虽然感觉没有体现LFS的含义，但如果取《在树莓派进行LFS》这样的类似名字，估计很多人都不明白什么意思。所以先来科普一下什么是LFS。Linux From Scratch项目简称LFS，它提供具体的步骤、特定的补丁、必须的脚本，从而提供一个简便的创建Linux发行版的途径。简单来说提供了一个自己从网上下载所有源代码编译一个linux完整系统的指南。
	为什么要在树莓派上编译？首先是学习，当然如果在pc机上LFS也一样可以通过LFS学习linux，但我没有在裸机上安装linux，如果运行虚拟机的话估计速读比树莓派也快不了多少。而且手头有树莓派啊，LFS需要长时间开机，树莓派可以安静的工作没有任何噪音困扰，你可以用你的pc做其它事情，这样多惬意。再有我在预装了树莓派的raspbian系统后发现启动很慢，系统预装了很多我不需要的东西占用了大量的内存和TF卡空间，如果我想用树莓派做一个固定的用处，比如BT下载机，或者摄像机之类的，当然需要一个小巧，迅速的系统，所以需要定制。最后，我想我是喜欢树莓派吧。
	在我刚开始想用树莓派进行LFS时，感觉是一项大工程，因为以前我用pc参照LFS手册进行LFS，感觉这是一项复杂和需要勇气的工作，而且对于是否可以在树莓派上进行LFS实在心里没底。直到我在网上找到了一个网站（http://www.intestinate.com/pilfs/
），知道国外有人已经进行了尝试并且编制了相关的脚本，沿着前人的步伐，我发现在树莓派上进行LFS不是一项困难的工作，甚至感觉so easy。我借用了pilfs网站的脚本，在此十分感谢pilfs网站的帮助。
##二、前提条件
	1，需要一个树莓派及电源、键盘、鼠标、TF卡（16G）、U盘等，大概这是废话了，另外根据我的经验，在pi1上做LFS那是一件极其痛苦的事情，所以建议在pi3上完成LFS，pi2是否可以我没测试。
	2，一台可以上网的pc，pc作为辅助功能必不可少，有很多事情需要在电脑上完成。另外我在运行LFS时是通过pc联机到树莓派，这样可以记录下所有屏幕日志方便查看编译过程是否有问题。
	除了以上两点，你必须还要有足够的耐心和基本的linux知识。
##三、准备工作
	首先你应该阅读LFS手册，这是你学习linux的绝好机会，LFS官方网站http://www.linuxfromscratch.org/。目前有人已经对LFS手册进行了翻译，建议参考：
1、	http://www.jinbuguo.com/ 6.2版本非systemd版本
2、	中文参考文档version 7.7-systemd systemd版本下载：https://linux.cn/article-5797-rss.html 开源中文社区https://linux.cn/
3、	你同时也可以浏览一下pilfs网站的guide，该网站所指定的LFS指南在线版本http://www.linuxfromscratch.org/lfs/view/development/index.html
环境准备：
1、	安装raspbain
需要一张16G的TF卡作为LFS的环境，但先需要安装raspbain操作系统，我安装的版本是2016-05-10-raspbian-jessie，网上可以下载zip文件，然后解压之后是一个img文件。你可以用Win32DiskImager工具软件将img文件写入tf卡。有一个小问题，如果是用hdmi转vga的，可能无法正常显示，需要修改tf卡上的config.txt加上：
hdmi_force_hotplug=1
config_hdmi_boost=4
hdmi_group=2
hdmi_mode=9
hdmi_drive=2
hdmi_ignore_edid=0xa5000080
试试启动树莓派是否正常。
2、	通过PUTTY联机树莓派
在pc机上安装putty软件。但是如何联机树莓派？我是用360wifi将pc作为一个ap热点，然后用树莓派去联接pc，这样的好处是不用任何开销，而且树莓派可以通过pc访问互联网，事实证明也很稳定。当树莓派联上pc之后，启动putty试试是否正常，如果能正确联机，那么恭喜你已经迈出LFS的一小步了，另外别忘了设置putty将任何屏幕输出记录下来（putty软件的session/logging中All session output选项）。
3、	下载LFS需要的软件包
我已经整理好了LFS所有需要的软件包，请到xx下载。这是一个tar.gz的包，将其复制到u盘上备用。

##四、开始LFS
	好了，一切准备工作就绪，现在可以正式开始LFS了。
	1、创建目录和用户
	启动树莓派，用putty联上，用root登录。先查看一下时间是否正确（date命令），在编译时时间非常重要，如果时间不对执行date –s 11/30/16（重要！）设置时间。
	更新raspbain，确保树莓派能访问外网，有4个软件包需要更新：
sudo su
apt-get update
apt-get install bison gawk m4 texinfo
建立相关目录
mkdir -pv /lfs;-p如果目录存在不报错
mkdir –v /lfs/sources
chmod -v a+wt /lfs/sources
mkdir -v /lfs/tools
ln -sv /lfs/tools /  ;在根目录下建立链接
;增加交换文件
dd if=/dev/zero of=/swapfile bs=1M count=512
mkswap /swapfile
swapon -v /swapfile

将下载xx的u盘插入树莓派，解压文件到/mnt/lfs/sources
mkdir /mnt/tmp
mount /dev/sda1 /mnt/tmp
tar –zxvf /mnt/tmp/xxx /lfs/sources

添加lfs用户

groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
passwd lfs
chown -v lfs /lfs/tools
chown -v lfs /lfs/sources
su – lfs
export LFS=/lfs

	2、执行脚本ch5-build.sh
以lfs用户执行4_4_set_env.sh，执行完之后退出lfs用户重新登录。一定确认以下环境变量$LFS(/mnt/lfs)和$LFS_TGT(armv7l-lfs-linux-gnueabihf)正确。
cd /lfs/sources
./4_4_set_env.sh
exit
su - lfs 
cd /lfs/sources
./ch5-buildum.sh
此过程大概有8个小时，在结束之后，可以通过查看putty的截屏查看是否有错误。如果顺利通过，那么应该说已经成功了一大半了，后面的步骤不会遇到大的问题。编译完成的软件安装在/tools目录下，修改文件属性，以root用户执行：
exit
chown -R root:root /lfs/tools

3、进chroot

	以下开始已root身份执行命令，执行s6.2.sh 确认一下/mnt/lfs/dev/consul ，/mnt/lfs/dev/null是否生成
export LFS=/lfs
cd /lfs/sources
./s6.2.sh
进入chroot（bash s.6.4_chroot.sh），但需要root用户执行，否则失败。
./S6.4_chroot.sh
下面命令如果机器重启之后要执行s6.2r.sh。

4、执行脚本ch6-build.sh
先执行s6.5_6.sh（生成相关目录、passwd、group文件）

cd /sources
./s6.5_6.sh
exec /tools/bin/bash --login +h 
touch /var/log/{btmp,lastlog,wtmp} 
chgrp -v utmp /var/log/lastlog 
chmod -v 664 /var/log/lastlog 
chmod -v 600 /var/log/btmp
./ch6-build.sh
ch6-binuld.sh脚本大概执行有5个小时，最后有3个问题，这三个问题如果选择YES，则分别执行：
1，	cp -rv /sources/firmware-master/hardfp/opt/vc /opt
2，	cp -rv /sources/firmware-master/modules /lib
3，	mount /dev/mmcblk0p1 /boot && cp -rv /sources/firmware-master/boot / && umount /boot;
特别是第三步，会覆盖了原来tf卡上的启动文件，重新启动会以新的内核启动，建议原来的boot分区文件备份一份，以防止无法启动时在pc上重新复制。

5、清除无用内容及基本配置信息
	清除调试信息及/tools目录，此时/tools目录已经不需要可以删除。当然如果你想再次LFS，可以备份/tools目录，这样下次第一阶段（ch5-binuld.sh）就不需要执行了。
Logout
cd /lfs/sources
./s6.71_chroot.sh
/tools/bin/find /{,usr/}{bin,lib,sbin} -type f -exec /tools/bin/strip --strip-debug '{}' ';'
rm -rf /tmp/*
rm -rf /tools
cd /sources
./s7.2_9.sh （bash /lib/udev/init-net-rules.sh）是否正确？mount一下/sys是否mount？
cp fstab /etc
	从现在开始如果重启了树莓派，进入chroot 环境执行(root用户)：
	export LFS=/lfs
	s6.2r.sh
s6.71_crhoor
	6、准备启动
	如果你已经走到这里的，那么应该恭喜你已经完成了LFS的99%，离完成LFS就差一步之遥了。下面的操作会删除原来的系统只保留编译出来的lfs目录，如果你怕误操作破坏tf卡上的内容，可以先备份一下（可以用Win32DiskImager，或者tar）。
	以下步骤需要将TF卡取下，使用另外的linux系统进行操作。如果你有另外的树莓派系统盘，这时候可以插入另外一张TF卡启动树莓派，将编译有LFS那张TF卡用一个读卡器接入树莓派，然后执行：
mount /dev/sda2 /mnt
cd /mnt
shopt -s extglob
rm -rf !(lfs)
mv /mnt/lfs/* /mnt
如果在执行ch6-binuld.sh之后的第三个问题你回答了YES，那么这整个LFS已经完成了，你可以umount之后插入这张完成的TF卡试试启动（如果有HDMI转VGA，同样别忘了修改config.txt）。如果之前没有覆盖boot文件，可以将下载到的boot目录通过u盘复制到boot分区，命令参考如下。

mkdir /mnt/lfs
mkdir /mnt/tmp
mount /dev/sda1 /mnt/lfs
mount /dev/sdb1 /mnt/tmp
cp –R /mnt/tmp/xx /mnt/lfs
umount /mnt/tmp
umount /mnt/lfs
Hardware name：BCM2708
not syncing:Attempted to kill init!exitcode=0x00000004




