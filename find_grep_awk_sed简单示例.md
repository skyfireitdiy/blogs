文章只是大致介绍一些常用的用法，还有很多事没有提到的，这几个工具在linux上的功能是很强大的，最好根据命令的官方文档完整的学习一遍。

# find

* 查找文件或者文件夹

```bash
find /usr/include -name "stdio.h"
```

输出：

```text
/usr/include/bits/stdio.h
/usr/include/wine/msvcrt/stdio.h
/usr/include/bsd/stdio.h
/usr/include/c++/9.2.0/tr1/stdio.h
/usr/include/stdio.h
```

* 使用通配符

```bash
find /usr/include -name "*c++"
```

输出：

```text
/usr/include/sigc++-2.0
/usr/include/sigc++-2.0/sigc++
/usr/include/sigc++-2.0/sigc++/sigc++.h
/usr/include/c++
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/stdtr1c++.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/stdc++.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/c++locale.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/c++allocator.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/c++io.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/c++config.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/extc++.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/stdtr1c++.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/stdc++.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/c++locale.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/c++allocator.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/c++io.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/c++config.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/extc++.h
/usr/include/c++/9.2.0/bits/c++0x_warning.h
```

* 只查找目录

```bash
find /usr/include -name "*c++*" -type d
```

输出

```text
/usr/include/sigc++-2.0
/usr/include/sigc++-2.0/sigc++
/usr/include/c++

```

* 只查找文件

```bash
find /usr/include -name "*c++*" -type f
```

输出：

```text
/usr/include/sigc++-2.0/sigc++/sigc++.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/stdtr1c++.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/stdc++.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/c++locale.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/c++allocator.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/c++io.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/c++config.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/extc++.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/stdtr1c++.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/stdc++.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/c++locale.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/c++allocator.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/c++io.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/c++config.h
/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/extc++.h
/usr/include/c++/9.2.0/bits/c++0x_warning.h

```

* 查找30天前修改的文件

```bash
find /usr/include -name "*c++*" -mtime +30
```

输出：

```text
/usr/include/sigc++-2.0
/usr/include/sigc++-2.0/sigc++
/usr/include/sigc++-2.0/sigc++/sigc++.h

```

* 查找今天修改的文件

```bash
find . -mtime -1
```

输出：

```text
.
./find_grep_sed_awk.md
```

* 对找到的每个文件做一些事情

```bash
find /usr/include -name "*c++*" -type f -exec cp -r {} ./ \;
```

输出：

```text
'/usr/include/sigc++-2.0/sigc++/sigc++.h' -> './sigc++.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/stdtr1c++.h' -> './stdtr1c++.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/stdc++.h' -> './stdc++.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/c++locale.h' -> './c++locale.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/c++allocator.h' -> './c++allocator.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/c++io.h' -> './c++io.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/c++config.h' -> './c++config.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/bits/extc++.h' -> './extc++.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/stdtr1c++.h' -> './stdtr1c++.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/stdc++.h' -> './stdc++.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/c++locale.h' -> './c++locale.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/c++allocator.h' -> './c++allocator.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/c++io.h' -> './c++io.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/c++config.h' -> './c++config.h'
'/usr/include/c++/9.2.0/x86_64-pc-linux-gnu/32/bits/extc++.h' -> './extc++.h'
'/usr/include/c++/9.2.0/bits/c++0x_warning.h' -> './c++0x_warning.h'

```

```bash
find . -name "*c++*" -exec rm -v {} \;
```

输出：

```text
已删除 './c++config.h'
已删除 './c++io.h'
已删除 './extc++.h'
已删除 './c++0x_warning.h'
已删除 './stdtr1c++.h'
已删除 './c++allocator.h'
已删除 './c++locale.h'
已删除 './sigc++.h'
已删除 './stdc++.h'

```

* 查找大小超过500k的文件并显示一下行数

```bash
find /usr/include -type f -size +500k -exec wc -l {} \;
```

输出：

```text
11857 /usr/include/sqlite3.h
18827 /usr/include/ruby-2.6.0/x86_64-linux/rb_mjit_min_header-2.6.5.h
28987 /usr/include/wine/windows/mshtml.idl
23864 /usr/include/wine/windows/msxml2.h
11856 /usr/include/wine/windows/dwrite_3.h
79293 /usr/include/wine/windows/mshtml.h
16182 /usr/include/wine/windows/shobjidl.h
23255 /usr/include/wine/windows/msxml6.h
19634 /usr/include/epoxy/gl_generated.h
19478 /usr/include/qt/QtOpenGLExtensions/qopenglextensions.h
12264 /usr/include/qt/QtGui/qopenglext.h
8789 /usr/include/qt/QtCore/5.13.1/QtCore/private/qlocale_data_p.h
3533 /usr/include/clang/Basic/DiagnosticSemaKinds.inc
11347 /usr/include/clang/Sema/Sema.h
11177 /usr/include/SDL2/SDL_opengl_glext.h
23686 /usr/include/GL/glew.h
12763 /usr/include/GL/glext.h
11784 /usr/include/dlang/ldc/std/datetime/systime.d
11081 /usr/include/dlang/ldc/std/internal/unicode_tables.d
11784 /usr/include/dlang/dmd/std/datetime/systime.d
11081 /usr/include/dlang/dmd/std/internal/unicode_tables.d
11080 /usr/include/dlang/gdc/std/internal/unicode_tables.d

```

* 查找指定权限的文件

```bash
find /usr/include -type f -perm 755 -exec ls -l {} \;
```

输出：

```text
-rwxr-xr-x 1 root root 18981  4月 21 02:15 /usr/include/http_parser.h
-rwxr-xr-x 1 root root 181547  7月  5 16:28 /usr/include/va/va.h
-rwxr-xr-x 1 root root 56026  7月  5 16:28 /usr/include/va/va_vpp.h
-rwxr-xr-x 1 root root 1966 11月 10  2018 /usr/include/libnethogs.h

```

# grep

* 查找文件中的指定内容

```bash
grep "root" /etc/passwd
```

输出：

```text
root:x:0:0::/root:/usr/bin/fish
```

* 高亮

```bash
grep "root" /etc/passwd --color
```

这里看不到，所以就不输出了。

* 显示行号

```bash
grep -n "root" /etc/passwd
```

输出：

```text
1:root:x:0:0::/root:/usr/bin/fish

```

* 查找正则表达式

```bash
grep "sh\$" /etc/passwd
```

输出：

```text
root:x:0:0::/root:/usr/bin/fish
skyfire:x:1000:1001:skyfire:/home/skyfire:/usr/bin/fish
postgres:x:964:964:PostgreSQL user:/var/lib/postgres:/bin/bash

```

* 反选，除去不满足条件的

```bash
grep "sh\$" /etc/passwd | grep -v "bash\$" 
```

查找所有以sh结尾，但是不以bash结尾的行：

输出：

```text
root:x:0:0::/root:/usr/bin/fish
skyfire:x:1000:1001:skyfire:/home/skyfire:/usr/bin/fish

```

* 统计查找到的次数

```bash
grep "sh\$" /etc/passwd -c
```

输出：

```text
3

```

* 忽略大小写

```bash
grep "SH\$" /etc/passwd -i
```

输出：

```text
root:x:0:0::/root:/usr/bin/fish
skyfire:x:1000:1001:skyfire:/home/skyfire:/usr/bin/fish
postgres:x:964:964:PostgreSQL user:/var/lib/postgres:/bin/bash

```

 * 使用正则表达式

 输出ifconfig命令输出的所有ip地址：

 ```bash
 ifconfig | egrep "([0-9]{1,3}\.){3}[0-9]{1,3}" --color
 ```

 输出：

```text
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet 192.168.42.240  netmask 255.255.255.0  broadcast 192.168.42.255
        inet 127.0.0.1  netmask 255.0.0.0

```

# awk

awk 脚本的基本结构：

```bash
awk 'BEGIN { print "start"} pattern {commands } END {print "end"}' file
```

* 列出/etc/passwd文件的第一列，文件使用:分割：


```bash
awk -F: '{print $1}' /etc/passwd
```
输出：

```text
root
nobody
dbus
bin
daemon
mail
ftp
http
systemd-journal-remote
systemd-coredump
uuidd
dnsmasq
rpc
avahi
colord
cups
deepin_anything_server
deepin-sound-player
deepin-daemon
deluge
git
lightdm
nm-openconnect
nm-openvpn
ntp
polkitd
rtkit
usbmux
skyfire
systemd-network
systemd-resolve
systemd-timesync
deepin-anything-server
geoclue
lantern
postgres

```

* 给`hello world`加上双引号

```bash
echo hello world | awk 'BEGIN{print "Start"}{print "\""$1"\"" "\""$2"\""}END{print "End"}' 
```

输出：

```text
Start
"hello""world"
End

```

* 给文本加上行号

```bash
echo -e "hello\nworld\n666" | awk '{print NR " - " $1}'
```

输出：

```text
1 - hello
2 - world
3 - 666

```

* 统计文本字段数量

```bash
echo hello world 666 | awk '{print "文本字段：" NF}' 
```

输出

```text
文本字段：3

```

* 传递外部变量

```bash
echo hello | awk -v SUFFIX="world" '{print $1 "," SUFFIX}' 
```

或者：

```bash
echo hello | awk '{print $1 "," SUFFIX}' SUFFIX="world"
```

输出

```text
hello,world

```

* 输出行号小于5的行

```bash
awk 'NR < 5' /etc/passwd
```

输出：

```text
root:x:0:0::/root:/usr/bin/fish
nobody:x:65534:65534:Nobody:/:/sbin/nologin
dbus:x:81:81:System Message Bus:/:/sbin/nologin
bin:x:1:1::/:/sbin/nologin

```

* 行号在3到6之间的行

```bash
awk 'NR==3,NR==6' /etc/passwd
```

输出
```text
dbus:x:81:81:System Message Bus:/:/sbin/nologin
bin:x:1:1::/:/sbin/nologin
daemon:x:2:2::/:/sbin/nologin
mail:x:8:12::/var/spool/mail:/sbin/nologin

```

* 包含root的行

```bash
awk '/root/' /etc/passwd
```

输出：

```text
root:x:0:0::/root:/usr/bin/fish

```

* 不包含root的行

```bash
awk '!/root/' /etc/passwd
```

输出

```text
nobody:x:65534:65534:Nobody:/:/sbin/nologin
dbus:x:81:81:System Message Bus:/:/sbin/nologin
bin:x:1:1::/:/sbin/nologin
daemon:x:2:2::/:/sbin/nologin
mail:x:8:12::/var/spool/mail:/sbin/nologin
ftp:x:14:11::/srv/ftp:/sbin/nologin
http:x:33:33::/srv/http:/sbin/nologin
systemd-journal-remote:x:982:982:systemd Journal Remote:/:/sbin/nologin
systemd-coredump:x:981:981:systemd Core Dumper:/:/sbin/nologin
uuidd:x:68:68::/:/sbin/nologin
dnsmasq:x:980:980:dnsmasq daemon:/:/sbin/nologin
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
avahi:x:979:979:Avahi mDNS/DNS-SD daemon:/:/sbin/nologin
colord:x:978:978:Color management daemon:/var/lib/colord:/sbin/nologin
cups:x:209:209:cups helper user:/:/sbin/nologin
deepin_anything_server:x:977:977:Deepin Anything Server:/:/sbin/nologin
deepin-sound-player:x:976:976:Deepin Sound Player:/:/sbin/nologin
deepin-daemon:x:975:975:Deepin Daemon:/:/sbin/nologin
deluge:x:974:974:Deluge BitTorrent daemon:/srv/deluge:/sbin/nologin
git:x:973:973:git daemon user:/:/usr/bin/git-shell
lightdm:x:620:620:Light Display Manager:/var/lib/lightdm:/sbin/nologin
nm-openconnect:x:972:972:NetworkManager OpenConnect:/:/sbin/nologin
nm-openvpn:x:971:971:NetworkManager OpenVPN:/:/sbin/nologin
ntp:x:87:87:Network Time Protocol:/var/lib/ntp:/bin/false
polkitd:x:102:102:PolicyKit daemon:/:/sbin/nologin
rtkit:x:133:133:RealtimeKit:/proc:/sbin/nologin
usbmux:x:140:140:usbmux user:/:/sbin/nologin
skyfire:x:1000:1001:skyfire:/home/skyfire:/usr/bin/fish
systemd-network:x:970:970:systemd Network Management:/:/sbin/nologin
systemd-resolve:x:969:969:systemd Resolver:/:/sbin/nologin
systemd-timesync:x:968:968:systemd Time Synchronization:/:/sbin/nologin
deepin-anything-server:x:967:967:Deepin Anthing Server:/:/sbin/nologin
geoclue:x:966:966:Geoinformation service:/var/lib/geoclue:/sbin/nologin
lantern:x:619:619::/var/lib/lantern:/bin/false
postgres:x:964:964:PostgreSQL user:/var/lib/postgres:/bin/bash

```

* 循环

```bash
echo | awk '{for(i=0;i<10;i++){print i}}'
```

输出：

```text
0
1
2
3
4
5
6
7
8
9

```

# sed

* 替换字符串

```bash
echo hello world | sed 's/world/skyfire/'
```

输出：

```text
hello skyfire

```

* 替换全部

```bash
echo hello world | sed 's/o/O/g' 
```

输出

```text
hellO wOrld

```

* 删除匹配行

```bash
echo -e "hello\nworld\n666" | sed '/h/d' 
```

输出

```text
world
666

```

* 替换正则表达式

```bash
echo hello world | sed 's/l\{2\}/LL/g' 
```

输出

```text
heLLo world

```

* 已匹配字符串标记

```bash
echo hello world | sed 's/he\(l\+\)o/& \1/g'
```

输出

```text
hello ll world

```

* 组合多个表达式

```bash
echo hello world | sed 's/l/o/g;s/o\+/t/g'
```

输出：

```text
het wtrtd

```

