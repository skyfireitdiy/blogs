# git普通库与裸库

创建一个git仓库的时候，有两种方式：

```bash
git init example
```

和

```bash
git init --bare example
```

这两个有什么区别呢？

使用`git init`创建的仓库被称为普通库，使用`git init --bare`创建的库被称为裸库。

普通库不仅包含了版本控制信息，还含了项目所有的源文件，而裸库只包含了版本控制信息。

举例如下：

* 普通库

在/tmp目录下创建一个普通库 test

```bash
cd /tmp
git init test
```

查看文件：

```bash
cd test
ls -a
```
结果：

```text
./  ../  .git/
```

查看.git中的文件：

```bash
cd .git 
ls -a
```
结果：

```text
./  ../  branches/  config  description  HEAD  hooks/  info/  objects/  refs/
```

创建一个文件，加入版本控制，提交：
**(普通库是可以直接操作的)**

```bash
cd /tmp/test
touch a 
git add a
git commit -m "first commit"
```

将仓库克隆到/tmp/test2

```bash
git clone skyfire@localhost:/tmp/test /tmp/test2
```

在test2下建立一个新文件，加入版本控制，并且提交：

```bash
cd /tmp/test2
touch b
git add b 
git commit -m "add b"
```

尝试推送：

```bash
git push
```

此时会报出如下错误：

```text
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 226 bytes | 226.00 KiB/s, done.
Total 2 (delta 0), reused 0 (delta 0)
remote: error: refusing to update checked out branch: refs/heads/master
remote: error: By default, updating the current branch in a non-bare repository
remote: is denied, because it will make the index and work tree inconsistent
remote: with what you pushed, and will require 'git reset --hard' to match
remote: the work tree to HEAD.
remote: 
remote: You can set the 'receive.denyCurrentBranch' configuration variable
remote: to 'ignore' or 'warn' in the remote repository to allow pushing into
remote: its current branch; however, this is not recommended unless you
remote: arranged to update its work tree to match what you pushed in some
remote: other way.
remote: 
remote: To squelch this message and still keep the default behaviour, set
remote: 'receive.denyCurrentBranch' configuration variable to 'refuse'.
To localhost:/tmp/test
 ! [remote rejected] master -> master (branch is currently checked out)
error: failed to push some refs to 'skyfire@localhost:/tmp/test'
```

因为remote仓库当前也在master分支，并且是可以直接操作的，所以如果此时向remote的当前分支push代码，可能会将正在修改的代码覆盖掉。因此只能向非当前分支push修改：

```bash
git branch dev
git push origin dev
```

这样就可以正常push了。如果实在需要push到master怎么办呢？可以将远程的当前分支切换到非master分支：

```bash
cd /tmp/test
git checkout dev
cd /tmp/test2
git checkout master
git push origin master
```

* 裸库

接下来说说裸库的情况。

先将刚才建立的仓库删除：

```bash
cd /tmp
rm -rf test test2
```
创建裸库：

```bash
git init --bare test
```

查看文件：

```bash
cd test
ls -a
```

结果：

```text
./  ../  branches/  config  description  HEAD  hooks/  info/  objects/  refs/
```

可以看到，test下的文件与普通库test/.git下的文件一样。

尝试建立一个文件，并将其加入版本控制：

```bash
touch a 
git add a
```

此时会报错：

```bash
fatal: this operation must be run in a work tree
```

说明裸库并不具备直接操作的能力。

将仓库克隆到/tmp/test2

```bash
git clone skyfire@localhost:/tmp/test /tmp/test2
```

在test2下建立一个新文件，加入版本控制，并且提交：

```bash
cd /tmp/test2
touch b
git add b 
git commit -m "add b"
```

尝试推送：

```bash
git push
```

推送成功。

## 总结

裸库与普通库的总结如下：

|  | 裸库 | 普通库 |
|:--|:--|:--|
|结构|版本控制文件| 源代码+裸库（.git）|
|直接操作|不可以|可以|
|推送|可直接推送，因为远端没有当前分支| 不可以推送到远端的当前分支|
|适用范围| 通常用于本地版本控制| 通常用于多人协作，版本库放在远程服务器|