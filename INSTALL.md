#树莓派操作系统彻底定制
##一、为什么要用树莓派  
  文章名字由来于网上有人对LFS项目的翻译，虽然感觉没有体现LFS的内涵，但如果叫《在树莓派进行LFS》这样的类似名字，估计很多人都不明白什么意思。先来科普一下什么是LFS。Linux From Scratch项目简称LFS，它提供具体的步骤、特定的补丁、必须的脚本，从而提供一个简便的创建Linux发行版的途径。简单来说提供了一个自己从网上下载所有源代码编译一个linux完整系统的指南，就好比自己去买电子零件安装一台收音机。  
  为什么要在树莓派上编译？首先是学习，当然如果在pc机上LFS也一样可以通过LFS学习linux，但我不想在裸机上安装linux，如果运行虚拟机的话估计速度比树莓派也快不了多少。而且手头有树莓派啊。LFS需要长时间开机，树莓派可以安静的工作没有任何噪音困扰，你可以用你的pc做其它事情，这样多惬意。另外我在预装了树莓派的raspbian系统后感觉启动很慢，系统预装了很多我不需要的东西占用了大量的内存和TF卡空间，如果我想用树莓派做一个固定的用处，比如BT下载机，或者摄像机之类的，当然需要一个小巧，启动迅速的系统，所以需要定制。最后，我想我是喜欢树莓派吧。  
  在我刚开始想用树莓派进行LFS时，感觉是一项大工程，因为以前我用pc参照LFS手册进行LFS，感觉这是一项复杂和需要勇气的工作，而且对于是否可以在树莓派上进行LFS实在心里没底。直到我在网上找到了一个网站<http://www.intestinate.com/pilfs/>，知道国外有人已经进行了尝试并且编制了相关的脚本，沿着前人的步伐，我发现在树莓派上进行LFS不是一项困难的工作，甚至感觉so easy。我借用了pilfs网站的脚本，在此十分感谢pilfs网站的帮助。
##二、前提条件
  需要一个树莓派及电源、键盘、鼠标、TF卡（16G）、U盘等附件，大概这是废话了，另外根据我的经历，在pi1上做LFS那是一件极其痛苦的事情（第一阶段需要50小时左右），所以建议在pi3上完成LFS，pi2是否可以我没测试过。 
  一台可以上网的pc。pc作为辅助功能必不可少，有很多事情需要在电脑上完成。另外我在运行LFS时是通过pc联机到树莓派，这样可以记录下所有屏幕日志方便查看编译过程是否有问题。
   除了以上两点，你必须还要有足够的耐心和基本的linux知识。
##三、准备工作
  首先你应该阅读LFS手册，这是你学习linux知识的绝佳机会，LFS官方网站<http://www.linuxfromscratch.org/>。目前有人已经对LFS手册进行了翻译，建议参考： 

1. <http://www.jinbuguo.com/> 6.2版本非systemd版本
2. 中文参考文档version 7.7-systemd systemd版本下载：<https://linux.cn/article-5797-rss.html> 开源中文社区<https://linux.cn/>
3. 你同时也可以浏览一下pilfs网站的guide，该网站所指定的LFS指南在线版本<http://www.linuxfromscratch.org/lfs/view/development/index.html>
###环境准备：
1. 安装raspbain

      需要一张16G的TF卡作为LFS的环境，但先需要安装raspbain操作系统，我安装的版本是2016-05-10-raspbian-jessie，网上可以下载zip文件(或者我的百度云盘)，然后解压之后是一个img文件。你可以用Win32DiskImager工具软件将img文件写入tf卡。有一个小问题，如果是用hdmi转vga的，可能无法正常显示，需要在写入img文件之后，用pc机插入tf卡，在boot分区的config.txt文件加上：

       hdmi_force_hotplug=1
       config_hdmi_boost=4
       hdmi_group=2
       hdmi_mode=9
       hdmi_drive=2
       hdmi_ignore_edid=0xa5000080
   最后试试启动树莓派是否正常。

2. 通过PUTTY联机树莓派  

      在pc机上安装putty软件。但是如何联机树莓派？我是用360wifi将pc作为一个ap热点，然后用树莓派去联接pc，这样的好处是不用任何开销而且设置简单，而且树莓派可以通过pc访问互联网，事实证明也很稳定。当然你也可以将树莓派直接联上可以上网的ap。当设置树莓派联上pc之后，启动putty试试是否正常，如果能正确联机，那么恭喜你已经迈出LFS的一小步了。此时可以配置树莓派启动到命令行就可以了（LFS过程不需要启动xwindow，这样可以节约宝贵的RAM空间，别告诉我你不会设置，请教度娘吧）另外别忘了设置putty将任何屏幕输出记录下来（putty软件的session/logging中All session output选项）。

   3、下载LFS需要的软件包
   我已经整理好了LFS所有需要的软件包，请到我的百度云盘下载（见附录）。将lfs_reselse下的sources全部下载下来，并复制到一个u盘上备用。将几个脚本文件从git服务器上下载下来，并拷贝到sources目录。

##四、开始LFS

好了，一切准备工作就绪，现在可以正式开始LFS了。

1. 创建目录和用户

      启动树莓派，用putty联上，用root登录。先查看一下时间是否正确（date命令），在编译时时间非常重要，如果时间不对执行date –s 11/30/16（重要！）设置时间。
   更新raspbain，确保树莓派能访问外网，有4个软件包需要更新：

        sudo su
        apt-get update
        apt-get install bison gawk m4 texinfo

2. 建立相关目录

        mkdir -pv /lfs;-p如果目录存在不报错
        mkdir –v /lfs/sources
        chmod -v a+wt /lfs/sources
        mkdir -v /lfs/tools
        ln -sv /lfs/tools /  ;在根目录下建立链接
        ;增加交换文件
        dd if=/dev/zero of=/swapfile bs=1M count=512
        mkswap /swapfile
        swapon -v /swapfile

     将u盘插入树莓派，将sources拷贝到/lfs/

        mount /dev/sda1 /mnt
        cp -R /mnt/sources/* /lfs/sources

3. 添加lfs用户

        groupadd lfs
        useradd -s /bin/bash -g lfs -m -k /dev/null lfs
        passwd lfs
        chown -v lfs /lfs/tools
        chown -v lfs /lfs/sources
        su – lfs
        export LFS=/lfs

4. 执行脚本ch5-build.sh

    &emsp;&emsp;以lfs用户执行4_4_set_env.sh，执行完之后退出lfs用户重新登录。确认一下环境变量$LFS(/mnt/lfs)和$LFS_TGT(armv7l-lfs-linux-gnueabihf)正确。

        cd /lfs/sources
        ./4_4_set_env.sh
        exit
        su - lfs 
        cd /lfs/sources
        ./ch5-build.sh
      此过程大概有8个小时（pi1大概需要50小时，另外建议树莓派1在config.txt文件在加上gpu_mem=16，以最大化memory）。在结束之后，可以通过查看putty的截屏查看是否有错误。如果顺利通过，那么应该说已经成功了一大半了，后面的步骤应该不会遇到大的问题。编译完成的软件安装在/tools目录下，修改文件属性，以root用户执行：

        exit
        chown -R root:root /lfs/tools

5. 进入chroot

    &emsp;&emsp;以下开始已root身份执行命令，执行s6.2.sh 确认一下/mnt/lfs/dev/consul ，/mnt/lfs/dev/null是否生成

        export LFS=/lfs
        cd /lfs/sources
        ./s6.2.sh
        ./S6.4_chroot.sh
      进入chroot（bash s.6.4_chroot.sh），但需要root用户执行，否则失败。下面命令如果机器重启之后要先执行s6.2r.sh在chroot。

6. 执行脚本ch6-build.sh
    &emsp;&emsp;先执行s6.5_6.sh（生成相关目录、passwd、group文件）

        cd /sources
        ./s6.5_6.sh
        exec /tools/bin/bash --login +h 
        touch /var/log/{btmp,lastlog,wtmp} 
        chgrp -v utmp /var/log/lastlog 
        chmod -v 664 /var/log/lastlog 
        chmod -v 600 /var/log/btmp
        ./ch6-build.sh

      ch6-binuld.sh脚本大概执行有5个小时（在树莓派1上大概需要38小时）。最后有3个问题，这三个问题如果选择YES，则脚本会执行：

    &emsp;&emsp;1，	cp -rv /sources/firmware-master/hardfp/opt/vc /opt 

    &emsp;&emsp;2，	cp -rv /sources/firmware-master/modules /lib 

    &emsp;&emsp;3，	mount /dev/mmcblk0p1 /boot && cp -rv /sources/firmware-master/boot / && umount /boot; 

    &emsp;&emsp;特别是第三步，会覆盖了原来tf卡上的启动文件，重新启动会以新的内核启动，建议原来的boot分区文件备份一份，以防止无法启动时在pc上重新复制。

7. 清除无用内容及基本配置信息

    &emsp;&emsp;清除调试信息及/tools目录，此时/tools目录已经不需要可以删除。当然如果你想再次LFS，可以备份/tools目录，这样下次第一阶段（ch5-binuld.sh）就不需要执行了。

        logout
        cd /lfs/sources
        ./s6.71_chroot.sh
        /tools/bin/find /{,usr/}{bin,lib,sbin} -type f -exec /tools/bin/strip --strip-debug '{}' ';'
        rm -rf /tmp/*
        rm -rf /tools
        cd /sources
        ./s7.2_9.sh 
        cp fstab /etc
        cd /
        rm -rf /sources
      完成之后可以执行bash /lib/udev/init-net-rules.sh看是否正确，mount一下/sys是否已经mount。从现在开始如果重启了树莓派，进入chroot 环境执行(以root用户，别忘了sources目录还没删除的时候可以执行)：

       export LFS=/lfs
       s6.2r.sh
        s6.71_crhoor

8. 准备启动

      如果你已经走到这里的，那么应该恭喜你已经完成了LFS的99%，离完成LFS就差一步之遥了。下面的操作会删除原来的系统只保留编译出来的lfs目录，如果你怕误操作破坏tf卡上的内容，可以先备份一下（可以用Win32DiskImager，或者tar命令）。
   以下步骤需要将TF卡取下，使用另外的linux系统进行操作。如果你有另外的树莓派系统盘，这时候可以插入另外一张TF卡启动树莓派，将编译有LFS那张TF卡用一个读卡器接入树莓派，然后执行：

        mount /dev/sda2 /mnt
        cd /mnt
        shopt -s extglob
        rm -rf !(lfs)
        mv /mnt/lfs/* /mnt

    &emsp;&emsp;如果在执行ch6-binuld.sh之后的第三个问题你回答了YES，那么这整个LFS已经完成了，你可以umount之后插入这张完成的TF卡试试启动（如果有HDMI转VGA，同样别忘了修改config.txt）。如果之前没有覆盖boot文件，可以将下载到的boot目录通过u盘复制到boot分区，命令参考如下：

        mkdir /mnt/lfs
        mkdir /mnt/tmp
        mount /dev/sda1 /mnt/lfs
        mount /dev/sdb1 /mnt/tmp
        cp –R /mnt/tmp/sources/boot_lfs/* /mnt/lfs
        umount /mnt/tmp
        umount /mnt/lfs
## 结束语

  到这里，恭喜你的定制树莓派操作系统完成了，系统预安装的软件参考LFS手册或者ch6-build.sh脚本。不过我故意遗留了一个问题，如何将此系统做成一个镜像文件？当然你可以用Win32DiskImager制作，但这样做出来的镜像文件大小是和你的TF卡容量一样大小的，而且以我的经历，16G的镜像文件要恢复还不是一件容易成功的事情。大家可以到pilfs网站上去查查制作一个1G镜像文件的方法。另外还有一个问题，就算1G镜像文件完成了，如果在一张TF卡上恢复了，恢复后的系统大小也是1G的，需要手工扩展分区到TF卡原来的大小，如何做？欢迎和我交流。

##附录

本文相关文件的下载链接：

1，关于我的百度云盘。 我没有合适的地方可以免费上传大文件，只能存放在百度云盘。
链接是<https://pan.baidu.com/s/1mhVb9NE>

pilfs_orig_rpi3_20161226.tar.gz是已经编译好的lfs，是一个镜像文件，但恢复之后只有1g大小，需要resize。

pilfs_orig_rpi1_20161230.tar.gz是已经编译好的lfs，是一个镜像文件，但恢复之后只有1g大小，需要resize。

pilfs_tools_rpi3_20161220.tar.gz是编译的tools目录（基于rpi3），这样可以不用执行ch5-build.sh，直接进入第二遍编译

pilfs_tools_rpi1_20161220.tar.gz是编译的tools目录（基于rpi1），这样可以不用执行ch5-build.sh，直接进入第二遍编译

2016-05-10-raspbian-jessie.zip是raspbian树莓派操作系统

sources目录，所有的源代码包

2，关于自己的脚本及本手册
本手册和脚本作为github的一个项目，放在github.com/breezecloud/myPiLFS,可以使用
git clone git://github.com/breezecloud/myPiLFS下载，也可以直接用浏览器<https://github.com/breezecloud/myPiLFS>

3，关于我
喜欢折腾，财力有限；年纪不小，空闲不多；爱好甚广，精通寥寥。
邮箱：luping@shtel.com.cn或者luping@189.cn欢迎交流
欢迎加入我的个人公众号，本人以后所有的原创文章均会发布在此公众号，公众号可以通过搜索electronic_computer加入，或者扫描二维码加入。

[id]: https://mmbiz.qlogo.cn/mmbiz_jpg/HkMWDzhKWAhHfV6Jleicm9l8O8qTLrlG0ZT0pZkLkM8ZS72TxJAm4TN4ScbbSyQMTBI3IYicW9HicnBq2ACRQuOOg/0?wx_fmt=jpeg "Title"


