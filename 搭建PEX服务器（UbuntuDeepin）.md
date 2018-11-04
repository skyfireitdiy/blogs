# 搭建PEX服务器（Ubuntu/Deepin）

## 工具准备

1. `dnsmasq`

   ```shell
   sudo apt install dnsmasq
   ```

2. 下载网络引导文件

   在`http://cn.archive.ubuntu.com/ubuntu/dists`下找到对应系统的网络引导文件，如`xenial/main/installer-amd64/current/images/netboot/netboot.tar.gz`,下载备用。

## 配置


1. 修改`/etc/dnsmasq.conf`

   ```shell
   sudo nano /etc/dnsmasq.conf
   ```

   修改为：

   ```
   interface=enp0s3
   dhcp-range=10.0.2.101,10.0.2.200,infinite
   dhcp-host=08:00:27:39:7c:07:16,10.0.2.15,svr1,infinite
   dhcp-boot=pxelinux/pxelinux.0
   enable-tftp
   tftp-root=/var/lib/tftpboot
   ```

   配置参数解释：

   `interface`

   	表示仅在指定网卡接口上侦听传入的客户端请求。

   `dhcp-range`

   	表示`dhcp`分配的网络地址范围和租期时间，`infinite`表示不限定时间。

   `dhcp-host`

   	表示`dhcp`主机（本机），配置了本机网卡的`MAC`地址、`IP`等信息。

   `dhcp-boot`

   	指定PXE客户端所需的引导加载程序文件的位置。此示例支持基于BIOS的PXE客户端。支持基于UEFI的客户端的条目可能采用以下形式：

   ```
   dhcp-boot=EFI/BOOTX64.efi
   ```

   `enable-tftp`

   	启用`dnsmasq`提供的`TFTP`服务。

   `tftp-root`

   	指定TFTP服务的文件的根目录。为防止客户端访问主机上的任何文件，dnsmasq拒绝指定`..`为路径元素的请求 。


2. 创建`tftp`目录

   ```shell
   sudo mkdir -p /var/lib/tftpboot/pxelinux
   ```

3. 解压网络引导文件到`tftp`目录（假设`netboot.tar.gz`位于`/tmp`下）

   ```shell
   cd /var/lib/tftpboot/pxelinux
   sudo tar xvf /tmp/netboot.tar.gz 
   ```

4. 重新启动`dnsmasq`服务。

至此，`PXE`服务器配置完成。

## 客户端从`PXE`启动

客户端与服务器在同一子网，从`PXE`启动就可以进入安装系统的流程了。