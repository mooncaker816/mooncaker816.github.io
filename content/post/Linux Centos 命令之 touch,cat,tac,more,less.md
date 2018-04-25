---
title: Linux Centos 命令之 touch,cat,tac,more,less
date: 2018-04-21 13:50:00
categories:
- Linux
tags:
- Centos
---

## touch -  创建空文件，修改时间

<!--more-->

### Usage：

```bash
Usage: touch [OPTION]... FILE...
Update the access and modification times of each FILE to the current time.

A FILE argument that does not exist is created empty, unless -c or -h
is supplied.

A FILE argument string of - is handled specially and causes touch to
change the times of the file associated with standard output.

Mandatory arguments to long options are mandatory for short options too.
  -a                     change only the access time
  -c, --no-create        do not create any files
  -d, --date=STRING      parse STRING and use it instead of current time
  -f                     (ignored)
  -h, --no-dereference   affect each symbolic link instead of any referenced
                         file (useful only on systems that can change the
                         timestamps of a symlink)
  -m                     change only the modification time
  -r, --reference=FILE   use this file's times instead of current time
  -t STAMP               use [[CC]YY]MMDDhhmm[.ss] instead of current time
      --time=WORD        change the specified time:
                           WORD is access, atime, or use: equivalent to -a
                           WORD is modify or mtime: equivalent to -m
      --help     display this help and exit
      --version  output version information and exit

Note that the -d and -t options accept different time-date formats.
```

mtime ：modification time （默认）

文件的内容被修改时会更新

ctime ：status time

文件的属性或权限被修改时会更新

atime ： access time

文件的内容被读取时会更新

```bash
[root@78063f0fe2e8 etc]# ls -l login.defs
-rw-r--r-- 1 root root 2028 Nov  4  2016 login.defs
[root@78063f0fe2e8 etc]# ls -l login.defs --time=atime
-rw-r--r-- 1 root root 2028 Nov  4  2016 login.defs
[root@78063f0fe2e8 etc]# ls -l login.defs --time=ctime
-rw-r--r-- 1 root root 2028 Apr 20 14:19 login.defs
```

### 常用选项：

-a（access） ：仅修改访问时间

-c（）：仅修改文件时间，若文件不存在则**不**创建新文件

-m（modification）： 仅修改 mtime

-t （time）：想要修改的时间[YYMMDDhhmm]

-d（date）：修改的日期

- 新建空文件

```bash
sh-4.2# cd /tmp
sh-4.2# touch testtouch
sh-4.2# ls -l testtouch
-rw-r--r-- 1 root root 0 Apr 21 07:52 testtouch
sh-4.2# ls -l testtouch --time=atime
-rw-r--r-- 1 root root 0 Apr 21 07:52 testtouch
sh-4.2# ls -l testtouch --time=ctime
-rw-r--r-- 1 root root 0 Apr 21 07:52 testtouch
```

- 修改时间

```bash
sh-4.2# cp -a ~/.bashrc bashrc
sh-4.2# ls -l bashrc
-rw-r--r-- 1 root root 176 Dec 29  2013 bashrc
sh-4.2# ls -l bashrc --time=atime
-rw-r--r-- 1 root root 176 Dec 29  2013 bashrc
sh-4.2# ls -l bashrc --time=ctime
-rw-r--r-- 1 root root 176 Apr 21 07:53 bashrc
sh-4.2# touch -d "2 days ago" bashrc
sh-4.2# ls -l bashrc
-rw-r--r-- 1 root root 176 Apr 19 07:55 bashrc
sh-4.2# ls -l bashrc --time=atime
-rw-r--r-- 1 root root 176 Apr 19 07:55 bashrc
sh-4.2# ls -l bashrc --time=ctime
-rw-r--r-- 1 root root 176 Apr 21 07:55 bashrc
sh-4.2# touch -t 1801011000 bashrc
sh-4.2# ls -l bashrc
-rw-r--r-- 1 root root 176 Jan  1 10:00 bashrc
sh-4.2# ls -l bashrc --time=ctime
-rw-r--r-- 1 root root 176 Apr 21 07:57 bashrc
sh-4.2# ls -l bashrc --time=atime
-rw-r--r-- 1 root root 176 Jan  1 10:00 bashrc
```

{{< asciinema EK9paXYyASszuUBvO9OOpzIfa >}}

## cat - concatenate 浏览文件

### Usage：

```bash
Usage: cat [OPTION]... [FILE]...
Concatenate FILE(s), or standard input, to standard output.

  -A, --show-all           equivalent to -vET
  -b, --number-nonblank    number nonempty output lines, overrides -n
  -e                       equivalent to -vE
  -E, --show-ends          display $ at end of each line
  -n, --number             number all output lines
  -s, --squeeze-blank      suppress repeated empty output lines
  -t                       equivalent to -vT
  -T, --show-tabs          display TAB characters as ^I
  -u                       (ignored)
  -v, --show-nonprinting   use ^ and M- notation, except for LFD and TAB
      --help     display this help and exit
      --version  output version information and exit

With no FILE, or when FILE is -, read standard input.

Examples:
  cat f - g  Output f's contents, then standard input, then g's contents.
  cat        Copy standard input to standard output.
```

### 常用选项：

-A（All）：显示所有，包括特殊字符，等价于-vET

-v（verbose）：列出看不出来的特殊字符

-T（Tab）：将 Tab 按键以^I 显示出来

-b（blank）：列出行号，空白行不标号

-n（number）：列出行号，空白行也有

-E（End）：将结尾的断行字符$显示出来

```bash
sh-4.2# cat /etc/issue
\S
Kernel \r on an \m

sh-4.2# cat -n /etc/issue
     1	\S
     2	Kernel \r on an \m
     3
sh-4.2# cat -b /etc/issue
     1	\S
     2	Kernel \r on an \m

sh-4.2# cd /tmp
sh-4.2# ls
bashrc		  mvtest2    testtouch		     yum.log
ks-script-hE5IPf  test.conf  tmpk3saal44-ascii.cast
sh-4.2# cat -A test.conf
^Ihello$
this is a test file only...$
sh-4.2# cat -An test.conf
     1	^Ihello$
     2	this is a test file only...$
```

{{< asciinema JqE1p5EPWbGntVdOAXNfnoY1O >}}

##  tac - 反向 cat

### Usage：

```bash
Usage: tac [OPTION]... [FILE]...
Write each FILE to standard output, last line first.
With no FILE, or when FILE is -, read standard input.

Mandatory arguments to long options are mandatory for short options too.
  -b, --before             attach the separator before instead of after
  -r, --regex              interpret the separator as a regular expression
  -s, --separator=STRING   use STRING as the separator instead of newline
      --help     display this help and exit
      --version  output version information and exit
```

tac 没有 -n 选项

```bash
[root@78063f0fe2e8 ~]# tac /etc/issue

Kernel \r on an \m
\S
```

## nl - 添加行号显示文件 

### Usage：

```bash
Usage: nl [OPTION]... [FILE]...
Write each FILE to standard output, with line numbers added.
With no FILE, or when FILE is -, read standard input.

Mandatory arguments to long options are mandatory for short options too.
  -b, --body-numbering=STYLE      use STYLE for numbering body lines
  -d, --section-delimiter=CC      use CC for separating logical pages
  -f, --footer-numbering=STYLE    use STYLE for numbering footer lines
  -h, --header-numbering=STYLE    use STYLE for numbering header lines
  -i, --line-increment=NUMBER     line number increment at each line
  -l, --join-blank-lines=NUMBER   group of NUMBER empty lines counted as one
  -n, --number-format=FORMAT      insert line numbers according to FORMAT
  -p, --no-renumber               do not reset line numbers at logical pages
  -s, --number-separator=STRING   add STRING after (possible) line number
  -v, --starting-line-number=NUMBER  first line number on each logical page
  -w, --number-width=NUMBER       use NUMBER columns for line numbers
      --help     display this help and exit
      --version  output version information and exit

By default, selects -v1 -i1 -l1 -sTAB -w6 -nrn -hn -bt -fn.  CC are
two delimiter characters for separating logical pages, a missing
second character implies :.  Type \\ for \.  STYLE is one of:

  a         number all lines
  t         number only nonempty lines
  n         number no lines
  pBRE      number only lines that contain a match for the basic regular
            expression, BRE

FORMAT is one of:

  ln   left justified, no leading zeros
  rn   right justified, no leading zeros
  rz   right justified, leading zeros
```

```bash
[root@78063f0fe2e8 ~]# nl /etc/issue
     1	\S
     2	Kernel \r on an \m

[root@78063f0fe2e8 ~]# nl /etc/issue -b a
     1	\S
     2	Kernel \r on an \m
     3
[root@78063f0fe2e8 ~]# nl /etc/issue -b a -n rz
000001	\S
000002	Kernel \r on an \m
000003
[root@78063f0fe2e8 ~]# nl /etc/issue -b a -n ln
1     	\S
2     	Kernel \r on an \m
3
[root@78063f0fe2e8 ~]# nl /etc/issue -b a -n rn
     1	\S
     2	Kernel \r on an \m
     3
[root@78063f0fe2e8 ~]# nl /etc/issue -b a -n rn -w 3
  1	\S
  2	Kernel \r on an \m
  3
[root@78063f0fe2e8 ~]# nl /etc/issue -b a -n rz -w 3
001	\S
002	Kernel \r on an \m
003
```

## more - 一页一页显示文件内容

### Usage：

```bash
more: unknown option -help
Usage: more [options] file...

Options:
  -d        display help instead of ring bell
  -f        count logical, rather than screen lines
  -l        suppress pause after form feed
  -p        do not scroll, clean screen and display text
  -c        do not scroll, display text and clean line ends
  -u        suppress underlining
  -s        squeeze multiple blank lines into one
  -NUM      specify the number of lines per screenful
  +NUM      display file beginning from line number NUM
  +/STRING  display file beginning from search string match
  -V        output version information and exit
```

空格：向下翻一页

Enter：向下一行

/字符串：向下查询字符串

q：退出查看

b：往回翻页

## less - more 的进阶

略

