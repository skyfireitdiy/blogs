# CTF 工具学习笔记

* nmap 探测工具

常用命令：

```bash
nmap -sV 192.168.6.0/24
```

表示扫描192.168.6.0.子网下所有机器的开放端口。

* dirb 查看站点隐藏文件

常用命令：

```bash
dirb http://192.168.7.6/
```

* ssh 远程连接

常用命令：

```bash
ssh -i id_rsa root@192.168.7.6
```

表示使用私钥文件id_rsa、用户名root登录远程主机192.168.7.6

其中 id_rsa 文件需要可读可写权限：

```bash
chmod 600 id_rsa
```

* ssh2john 转换ssh私钥为john可识别的信息。

常用命令：

```bash
ssh2john id_rsa > isacrack
```

* john 解密 isacrack 信息

john工具用于解密isacrack文件对应的信息。

命令：

```bash
zcat /usr/share/wordlists/rockyou.txt.gz | john --pipe --rules isacrack
```

* find 查看具有执行权限的文件

```bash
find / -perm -4000 2>/dev/null
```

从根目录查找具有可执行权限的文件，标准错误重定向到/dev/null

**要多注意 /root 目录下的文件。**