---
title: Linux Centos 命令之 mkdir,rmdir,pwd,cd
date: 2018-04-21 12:50:00
categories:
- Linux
tags:
- Centos
---

## mkdir - make directory 创建目录

<!--more-->

### Usage：

```bash
Usage: mkdir [OPTION]... DIRECTORY...
Create the DIRECTORY(ies), if they do not already exist.

Mandatory arguments to long options are mandatory for short options too.
  -m, --mode=MODE   set file mode (as in chmod), not a=rwx - umask
  -p, --parents     no error if existing, make parent directories as needed
  -v, --verbose     print a message for each created directory
  -Z                   set SELinux security context of each created directory
                         to the default type
      --context[=CTX]  like -Z, or if CTX is specified then set the SELinux
                         or SMACK security context to CTX
      --help     display this help and exit
      --version  output version information and exit
```

### 常用选项：

-m（mode）：按指定权限创建目录，若没有该选项则按 umask 的默认设置创建

-p（parent）：递归创建所有目录

-v（verbose）： 打印出每个创建的目录

- 递归创建目录

```bash
[root@78063f0fe2e8 ~]# mkdir -pv /tmp/test1/test2/test3
mkdir: created directory ‘/tmp/test1’
mkdir: created directory ‘/tmp/test1/test2’
mkdir: created directory ‘/tmp/test1/test2/test3’
```

- 目录的权限由 umask 决定

```bash
[root@78063f0fe2e8 ~]# umask
0022
[root@78063f0fe2e8 ~]# umask -S
u=rwx,g=rx,o=rx
[root@78063f0fe2e8 ~]# ls /tmp/test1 -l
total 4
drwxr-xr-x 3 root root 4096 Apr 21 05:01 test2
```

022是指对应三个组别分别要减去的权限分数，777-022=755=（rwxr-xr-x）

- 指定创建权限为711的目录

```bash
[root@78063f0fe2e8 ~]# mkdir /tmp/mingle -m 711
[root@78063f0fe2e8 ~]# ls /tmp/mingle -ld
drwx--x--x 2 root root 4096 Apr 21 05:10 /tmp/mingle
```

{{< asciinema IfLUdKAKL7a0b0p84Ye0ZgbGg >}}

## rmdir - remove empty directory 删除空目录

### Usage：

```bash
Usage: rmdir [OPTION]... DIRECTORY...
Remove the DIRECTORY(ies), if they are empty.

      --ignore-fail-on-non-empty
                  ignore each failure that is solely because a directory
                    is non-empty
  -p, --parents   remove DIRECTORY and its ancestors; e.g., 'rmdir -p a/b/c' is
                    similar to 'rmdir a/b/c a/b a'
  -v, --verbose   output a diagnostic for every directory processed
      --help     display this help and exit
      --version  output version information and exit
```

### 常用选项：

-p（parent）：递归删除空目录

-v（verbose）：打印出每个删除的目录

```bash
[root@78063f0fe2e8 ~]# mkdir -p /tmp/test1/test2/test3
[root@78063f0fe2e8 ~]# rmdir -pv /tmp/test1/test2/test3
rmdir: removing directory, ‘/tmp/test1/test2/test3’
rmdir: removing directory, ‘/tmp/test1/test2’
rmdir: removing directory, ‘/tmp/test1’
rmdir: removing directory, ‘/tmp’
rmdir: failed to remove directory ‘/tmp’: Directory not empty
```

{{< asciinema Cn6E7LzNtzxz21NeFaOUN0U3t >}}

## pwd - print working directory 打印当前路径

### Usage：

```bash
pwd [-LP]
```

### 常用选项：

-L（link）：若为软链，则显示软链路径(默认)

-P（）：若为软链，则显示为实际指向路径

```bash
[root@78063f0fe2e8 mail]# ls -ld /var/mail
lrwxrwxrwx 1 root root 10 Apr  2 18:38 /var/mail -> spool/mail
[root@78063f0fe2e8 mail]# pwd
/var/mail
[root@78063f0fe2e8 mail]# pwd -P
/var/spool/mail
[root@78063f0fe2e8 mail]# pwd -L
/var/mail
```

{{< asciinema olfTBQ4qihCn5XdssoKBj9Mn9 >}}

## cd - change directory 切换当前路径

### 常用目录：

"." ： 当前目录

".." ：上级目录

"-" ：前一个工作目录

"~"," " ：当前用户所在的主目录
"~account" ：account 这个用户所在的主目录

```bash
[root@78063f0fe2e8 mail]# cd
[root@78063f0fe2e8 ~]#
[root@78063f0fe2e8 ~]# cd /tmp
[root@78063f0fe2e8 tmp]# cd
[root@78063f0fe2e8 ~]# cd ~root
[root@78063f0fe2e8 ~]# cd -
/root
[root@78063f0fe2e8 ~]# cd ..
[root@78063f0fe2e8 /]# cd -
/root
[root@78063f0fe2e8 ~]# cd /var/mail
[root@78063f0fe2e8 mail]#
```

{{< asciinema tQtEI0jB0jXDClx5NryyKLD6i >}}

