# NFS（网络文件系统）配置（Ubunut/Deepin等系统）

[**网络文件系统**（**NFS**）](https://en.wikipedia.org/wiki/Network_File_System)是一种[分布式文件系统](https://en.wikipedia.org/wiki/Distributed_file_system)协议，最初由[Sun Microsystems](https://en.wikipedia.org/wiki/Sun_Microsystems)于1984年开发，允许客户端计算机上的用户通过[计算机网络](https://en.wikipedia.org/wiki/Computer_network)访问文件，就像访问本地存储一样。与许多其他协议一样，NFS建立在[开放网络计算远程过程调用](https://en.wikipedia.org/wiki/Open_Network_Computing_Remote_Procedure_Call)（ONC RPC）系统之上。NFS是[Request for Comments](https://en.wikipedia.org/wiki/Request_for_Comments)（RFC）中定义的开放标准，允许任何人实现该协议。

## 服务器配置

### 安装nfs服务

```shell
sudo apt install nfs-kernel-server
```

### 创建共享目录

在共享前创建需要共享的目录，如果要共享已有的目录，这个步骤可省略。

```shell
mkdir /tmp/nfs
```

创建目录`/tmp/nfs`

### 修改配置文件

```shell
sudo nano /etc/exports
```

配置文件格式为：

```
/srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
```

其中`/srv/homs`表示本地要共享的目录，`hostname1`表示本地允许远程连接的主机（格式可以为主机名、IP段等）

`rw`、`sync`等表示一些参数，完整的参数列表及含义请使用`man`命令查看：

```shell
man exports
```

本次参数文件如下：`cat /etc/exports`

```shell
# /etc/exports: the access control list for filesystems which may be exported
#		to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/tmp/nfs 192.168.6.0/24(rw,sync,root_squash,subtree_check)
```

以上参数表示共享本地的`/tmp/nfs`,允许远程访问的网段为`192.168.6.x`，权限为同步读写。

### 重新加载配置文件

```shell
sudo exportfs -r
```

至此，`nfs`服务段就配置完成了。

## 客户端连接

### 安装`nfs`

客户端只需要安装`nfs`的客户端组件就可以了：

```shell
sudo apt install nfs-common
```

### 查看远程主机中共享的`nfs`文件夹

```shell
showmount -e 192.168.6.217
```

`192.168.6.217`为服务器`ip`

如果正常，应该返回类似以下结果：

```shell
Export list for 192.168.6.217:
/tmp/nfs 192.168.6.0/24
```

### 确认自身是否在允许的列表中

可使用`ifconfig`查看自身`ip`信息，确保自身的`ip`在允许的列表中。

### 创建挂载点

使用`mkdir`命令创建一个共享目录的挂载点，如果已经有挂载点，可省略此步骤。

本次创建挂载点命令为：

```shell
mkdir /tmp/nfs-mount-point
```

### 挂载

使用`mount`命令挂载：

```shell
sudo mount -t nfs 192.168.6.217:/tmp/nfs /tmp/nfs-mount-point
```

表示挂载`192.168.6.217`主机下`/tmp/nfs`目录到本地`/tmp/nfs-mount-point`

### 验证

此时如果没有什么报错，就应该创建成功了。可以创建一个文件验证。

在服务器端执行：

```shell
echo 'Welcome to NFS server' > /tmp/nfs/test
```

在客户端执行

```shell
cat /tmp/nfs-mount-point/test
```

可以看到

```
Welcome to NFS server
```

表示网络文件系统已经成功搭建。