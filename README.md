# Some useful scripts

bbr.sh
======
- Description: Auto install latest kernel for TCP BBR
- Tutorial: http://www.iyu.co/share/tcp-bbr/

Copyright (C) 2017 Simon Huang <www.iyu.co>




去年下半年谷歌推出了新的 TCP-BBR 开源算法，并且已经提交到Linux新的内核，可以起到单边加速 TCP 连接的效果，也就是不用客户端的配合，用来替代收费的锐速再合适不过，毕竟只要更新kernel内核到最新版就可以开启使用。爱鱼客（www.iyu.co）在第一时间试用了一下BBR协议，TCP-BBR算法的目的和执行原理是要尽量跑满带宽，并且尽量不要有排队的情况，TCP-BBR和锐速相比，个人实际效果TCP-BBR要好很多。

Ubuntu 部署起来很方便，爱鱼客这边也已经为大家提供了 CentOS 的一键安装包了，使用 CentOS的同学们可以直接看下面的一键安装包，前面的内容都可以无视了。Ubuntu14、CentOS 6~7 下已经测试成功！

注意BBR需要更改内核和全虚拟化的VPS，因此和锐速一样不能用在 OpenVZ 的VPS机器上，所以搬瓦工等是用不了的。
另外现在有可以让OpenVZ（如搬瓦工）机器使用BBR的方法：【VPS加速】OpenVZ下TCP-BBR一键安装及影梭速度优化。但是因为OpenVZ的特性，虽然OpenVZ虚拟化效果很好，但是所以这边并不建议OpenVZ的机器安装TCP-BBR，如果需要了解便宜的KVM梯子主机服务器，可以直接问我，根据你使用的线路不同，我这边也可以给你推荐一些不错的主机商。

谷歌TCP-BBR拥堵算法

使用OpenVZ（如搬瓦工等）的同学可以参考爱鱼客的这一篇教程来加速你的VPS：OpenVZ 下 TCP-BBR一键安装教程及影梭速度优化方案

TCP-BBR项目github主页：https://github.com/google/bbr
TCP-BBR项目开发论坛（英文）：https://groups.google.com/forum/#!forum/bbr-dev
TCP-BBR快速使用手册（英文）：https://github.com/google/bbr/blob/master/Documentation/bbr-quick-start.md


 
TCP-BBR 目前已经在 YouTube 服务器和 Google 跨数据中心的内部广域网（B4）上部署，由此可见出该TCP加速算法前途一片光明啊！TCP-BBR的目标就是最大化利用网络上瓶颈链路的带宽。打个比方，一条网络就像一条水管，要想最大化利用这条水管，最好的方法就是将这条水管灌满水。

GOOGLE TCP-BBR

虽然目前网上对BBR的褒贬不一，有很多人表示BBR协议算法适用于数据中心等大型网络枢纽，一般的VPS或者服务器并不会有什么效果，并且还可能占满系统资源。在这里爱鱼客对此不做评价，你大可以照着下面介绍的方法在自己的服务器上部署一下试试看能不能达到你所预期的效果，如果不行的话就不要用。你也可以到 TCP-BBR 的官方论坛上面去吐槽（学术讨论当然是英文），不过我觉得我整理这一篇教程的目的不是为了站边说BBR好还是不好，而是为了让更多的人看到 BBR 所可能有的潜力，让需要它的人能真正利用到。

服务端部署
(2017年3月14日已更新为bbr加速一键安装脚本，请往下拉)

更新kernel最新内核
1. 下载4.9最新内核

wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.9/linux-image-4.9.0-040900-generic_4.9.0-040900.201612111631_amd64.deb
2. 安装内核

dpkg -i linux-image-4.9.0*.deb
3. 删除旧内核

 dpkg -l|grep linux-image
 apt-get purge 旧内核ID（此处为旧内核ID，不懂可以看下图）
开启TCP-BBR拥塞控制算法，与锐速相媲美的单边加速器

比如上图圈起来的部分就是需要卸载的旧内核，那么命令用该是这样的：

apt-get purge linux-image-4.4.0-53-generic linux-image-extra-4.4.0-53-generic 
4. 更新 grub 系统引导文件并重启

update-grub 
reboot
CentOS 6
1. 更换内核

rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-6-6.el6.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install kernel-ml -y
2. 查看内核是否安装成功

rpm -qa | grep kernel
3. 更新 grub 系统引导文件并重启

sed -i 's:default=.*:default=0:g' /etc/grub.conf
reboot
Tips：开不了机的打开vps后台控制面板的vnc, 开机卡在 grub 引导, 只需要手动选择内核就可以了。

CentOS 7
1. 更换内核

 rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
 rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
 yum --enablerepo=elrepo-kernel install kernel-ml -y
2. 查看内核是否安装成功

 rpm -qa | grep kernel
正常会如下所示:

[root@centos-512mb-sfo1-01 ~]# rpm -qa | grep kernel
 kernel-firmware-2.6.32-642.11.1.el6.noarch
 kernel-headers-2.6.32-642.11.1.el6.x86_64
 dracut-kernel-004-409.el6_8.2.noarch
 kernel-2.6.32-642.11.1.el6.x86_64
 kernel-devel-2.6.32-642.11.1.el6.x86_64
 kernel-ml-4.9.0-1.el6.elrepo.x86_64 #这就是我们安装的新内核
3. 更新 grub 系统引导文件并重启

 egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
 grub2-set-default 0 #default 0表示第一个内核设置为默认运行, 选择最新内核就对了
 reboot
【更新】安装TCP-BBR一键安装脚本（2017年3月14日）
使用root用户登录，运行以下命令：

wget --no-check-certificate https://github.com/iyuco/scripts/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
安装完成后，脚本会提示需要重启 VPS，

Info: The system needs to be restart. Do you want to reboot? [y/n]
输入 y 并回车后重启。重启完成后，重新登录进入 VPS，验证一下最新内核是否安装成功，输入以下命令：

uname -r
查看内核版本，若包含有 4.9及以上就表示 OK 了，现在应该已经更新到 4.11 了。

检测是否完全生效：

sysctl net.ipv4.tcp_available_congestion_control
返回值若为：net.ipv4.tcp_available_congestion_control = bbr cubic reno 即可。

接着输入：

sysctl net.ipv4.tcp_congestion_control
返回值一般为：
net.ipv4.tcp_congestion_control = bbr

sysctl net.core.default_qdisc
返回值一般为：net.core.default_qdisc = fq

最后再输入：

lsmod | grep bbr
返回值有 tcp_bbr 模块即说明bbr已启动。

这样大家按照爱鱼客的教程就已经为我们的VPS部署好了谷歌的 TCP-BBR 算法，感受一下飞的感觉吧！加速后的速度基本上是锐速效果的5-20倍，当然网上也有同学说用了比不用还慢的，或者说用了BBR遇到很多问题的，所以各位还是自己多多测试吧。

以下是Simon的VPS安装TCP-BBR前后 YouTube视频的对比，而且安装了BBR之后打开的是1080p的视频哦，大家可以看看差别还是蛮大的。

before-bbr

after-bbr

安装新梯子服务器建议的顺序为：TCP-BBR -> ss等服务（测试一下速度） -> 其他要装的东西。由于开启BBR之后服务器资源会占用得比较厉害，所以小内存的机器不建议使用，如果还想要在加速梯子上放几个网站的同学要小心服务器资源使用率哦！

另外我在CentOS6低版本上测试的时候遇到过kernel卡死错误的问题当然也不是所有的VPS都会有这个问题，所以如果遇到说kernel无法引导启动的话可以尝试使用CentOS7以上的系统，另外建议BBR在装好系统之后第一个安装，这样至少不会出现卡死需要重开VPS导致损失全部数据的风险。如果要卸载BBR请在评论区留言，或者直接重装系统。如果有其他问题可以在评论区留言！
