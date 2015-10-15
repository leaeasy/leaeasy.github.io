---
layout: post
title:  "Linux下的时间机器 -- Deepin实战BTRFS子卷"
date:   2015-10-15 10:41:01
categories: deepin system
tags: deepin
image: /assets/article_images/2015-10-15-btrfs-time-machine/header.jpg
---
基于btrfs的子卷等特性，我们可以轻松实现系统快速恢复、类似与OSX的时间机器等有趣实用的功能。

Btrfs 被称为是下一代 Linux 文件系统,目前btrfs已集成入内核开发主线。

本教程将针对btrfs字卷及压缩进行一些系统功能实现。

## 安装deepin系统

以下是全新安装deepin系统

### 分区选择
下载deepin系统按如下图分区方式进行安装

**由于grub引导限制，需要设置/boot分区**

![分区选择](/assets/article_images/2015-10-15-btrfs-time-machine/01-partition-options.png)


### 正常安装过程
![安装Deepin](/assets/article_images/2015-10-15-btrfs-time-machine/02-installing-deepin.png)

### 安装完成后正常进入系统

![Grub选择](/assets/article_images/2015-10-15-btrfs-time-machine/03-deepin-interface-grub.png)
![Deepin界面](/assets/article_images/2015-10-15-btrfs-time-machine/03-deepin-interface-desktop.png)
![挂载截图](/assets/article_images/2015-10-15-btrfs-time-machine/03-deepin-interface-mount.png)

## 已安装deepin系统
如果你已安装deepin系统,想要对你系统上的根分区进行转换，你首先需要使用Live CD启动。

在启动后，打开终端，使用下面的命令来转换文件系统。

{% highlight bash %}
btrfs-convert /dev/sda2 #这里假设/dev/sda2 为root分区
{% endhighlight %}

> 注意： 假如没有单独的/boot分区，还是需要重新建立一个单独的/boot分区

## 将deepin系统转移到btrfs子卷
由于目前deepin-installer无法支持直接安装到某个字卷，我们只能通过手动方式进行。

1. 将根分区挂载至某个目录（这里用/mnt），并进入该目录
```
mount /dev/sda2 /mnt
```

2. 将deepin系统迁移到字卷内部
```
btrfs subvolume snapshot /mnt /mnt/deepin-1012
```
 
3. 至此，deepin系统便迁移到字卷内部了。
 
4. 修改grub.cfg等文件
找到Deepin启动项目那行，在内核参数添加`rootflags=subvolid=xxxx`
> subvolid=xxxx中xxxx为subvolid值，通过`btrfs subvolume list .`可以得到，第二步可以看到deepin-1012字卷的ID为263

如下为grub.cfg的其中一段
<pre><code>
menuentry 'Deepin GNU/Linux 8.0 (jessie) GNU/Linux Stable' --class deepin --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-9db18def-ad74-4197-8303-ca912a56f316' {
    load_video
    insmod gzio
    	if [ x$grub_platform = xxen ]; then insmod xzio; insmod lzopio; fi
	insmod part_msdos
	insmod ext2
	set root='hd0,msdos1'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1  6b5865da-87d4-4d0d-8444-0181f759c886
	else
	  search --no-floppy --fs-uuid --set=root 6b5865da-87d4-4d0d-8444-0181f759c886
	fi
	echo	'载入 Linux 4.2.0-1-amd64 ...'
	linux	/vmlinuz-4.2.0-1-amd64 root=UUID=9db18def-ad74-4197-8303-ca912a56f316  rootflags=subvolid=263 ro  splash quiet
	echo	'载入初始化内存盘...'
	initrd	/initrd.img-4.2.0-1-amd64
}
</code></pre>

 由于grub.cfg文件指定了root分区信息，我们也可以注释掉/etc/fstab中根分区挂载条目，否则也需要在挂载参数中添加subvolid字段
 
重启选择修改后的grub启动项进入系统，能够成功进入系统后，便可以将以前不在字卷中的文件夹及文件删除了。
 
## deepin系统在btrfs字卷的新玩法
这里开始才是这篇教程真正要讲的内容。
以下是一个具体的例子。

**假如我们希望开启experimental源安装dde-next新桌面，但是又想保留2015旧版桌面，那我们该如何操作？**

1. 将根分区挂载至/mnt下
```bash
mount /dev/sda2 /mnt
```
2. 将当前系统快照一份备份
```bash
btrfs subvolume snapshot deepin-1012 deepin-stable
```
3. 安装传统方式添加experimental源安装dde-next桌面

4. 这里我们操作完后，就可以通过`btrfs subvolume list. `命令得到新字卷ID，然后添加到grub.cfg文件中

 ![子卷信息](/assets/article_images/2015-10-15-btrfs-time-machine/04-subvolume-list.png)
 
> 这里是grub.cfg文件的一部分
 <pre><code>
 menuentry 'Deepin GNU/Linux 8.0 (jessie) GNU/Linux Stable' --class deepin --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-9db18def-ad74-4197-8303-ca912a56f316' {
	load_video
	insmod gzio
	if [ x$grub_platform = xxen ]; then insmod xzio; insmod lzopio; fi
	insmod part_msdos
	insmod ext2
	set root='hd0,msdos1'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1  6b5865da-87d4-4d0d-8444-0181f759c886
	else
	  search --no-floppy --fs-uuid --set=root 6b5865da-87d4-4d0d-8444-0181f759c886
	fi
	echo	'载入 Linux 4.2.0-1-amd64 ...'
	linux	/vmlinuz-4.2.0-1-amd64 root=UUID=9db18def-ad74-4197-8303-ca912a56f316  rootflags=subvolid=269 ro  splash quiet
	echo	'载入初始化内存盘...'
	initrd	/initrd.img-4.2.0-1-amd64
}
menuentry 'Deepin GNU/Linux 8.0 (jessie) GNU/Linux Experimental' --class deepin --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-9db18def-ad74-4197-8303-ca912a56f316' {
	load_video
	insmod gzio
	if [ x$grub_platform = xxen ]; then insmod xzio; insmod lzopio; fi
	insmod part_msdos
	insmod ext2
	set root='hd0,msdos1'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1  6b5865da-87d4-4d0d-8444-0181f759c886
	else
	  search --no-floppy --fs-uuid --set=root 6b5865da-87d4-4d0d-8444-0181f759c886
	fi
	echo	'载入 Linux 4.2.0-1-amd64 ...'
	linux	/vmlinuz-4.2.0-1-amd64 root=UUID=9db18def-ad74-4197-8303-ca912a56f316  rootflags=subvolid=270 ro  splash quiet
	echo	'载入初始化内存盘...'
	initrd	/initrd.img-4.2.0-1-amd64
}
</code></pre>

以后我们就可以在选择菜单中选择进入dde-next系统，或者stable系统了

![子卷选择](/assets/article_images/2015-10-15-btrfs-time-machine/05-deepin-subvolume-grub.png)
 
### 一些解释
我们在执行安装experimental之前，将当前系统快照到一个新子卷，就类似相当于将当前系统生成一份备份，由于btrfs文件系统实现快照速度非常快，几秒之内就可以完成。
通过类似的操作，我们可以在需要测试软件、删除测试等非常影响系统情况下，操作后无需担心系统损坏等问题，只需要操作前生成一份快照便可放心安装测试，出现问题也只需要重新启动选择备份快照重新引导进入系统即可。

## 其他
btrfs也可以设置默认挂载子卷
```bash
 btrfs subvolume set-default 256 /
```
这样就无需修改grub.cfg文件添加字卷id参数，也可以默认将id为256的子卷默认挂载。


