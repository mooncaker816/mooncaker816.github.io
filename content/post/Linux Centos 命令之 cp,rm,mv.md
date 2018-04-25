---
title: Linux Centos 命令之 cp,rm,mv
date: 2018-04-21 13:50:00
categories:
- Linux
tags:
- Centos
---

## cp - copy 拷贝目录，文件

<!--more-->

### Usage：

```bash
Usage: cp [OPTION]... [-T] SOURCE DEST
  or:  cp [OPTION]... SOURCE... DIRECTORY
  or:  cp [OPTION]... -t DIRECTORY SOURCE...
Copy SOURCE to DEST, or multiple SOURCE(s) to DIRECTORY.

Mandatory arguments to long options are mandatory for short options too.
  -a, --archive                same as -dR --preserve=all
      --attributes-only        don't copy the file data, just the attributes
      --backup[=CONTROL]       make a backup of each existing destination file
  -b                           like --backup but does not accept an argument
      --copy-contents          copy contents of special files when recursive
  -d                           same as --no-dereference --preserve=links
  -f, --force                  if an existing destination file cannot be
                                 opened, remove it and try again (this option
                                 is ignored when the -n option is also used)
  -i, --interactive            prompt before overwrite (overrides a previous -n
                                  option)
  -H                           follow command-line symbolic links in SOURCE
  -l, --link                   hard link files instead of copying
  -L, --dereference            always follow symbolic links in SOURCE
  -n, --no-clobber             do not overwrite an existing file (overrides
                                 a previous -i option)
  -P, --no-dereference         never follow symbolic links in SOURCE
  -p                           same as --preserve=mode,ownership,timestamps
      --preserve[=ATTR_LIST]   preserve the specified attributes (default:
                                 mode,ownership,timestamps), if possible
                                 additional attributes: context, links, xattr,
                                 all
  -c                           deprecated, same as --preserve=context
      --no-preserve=ATTR_LIST  don't preserve the specified attributes
      --parents                use full source file name under DIRECTORY
  -R, -r, --recursive          copy directories recursively
      --reflink[=WHEN]         control clone/CoW copies. See below
      --remove-destination     remove each existing destination file before
                                 attempting to open it (contrast with --force)
      --sparse=WHEN            control creation of sparse files. See below
      --strip-trailing-slashes  remove any trailing slashes from each SOURCE
                                 argument
  -s, --symbolic-link          make symbolic links instead of copying
  -S, --suffix=SUFFIX          override the usual backup suffix
  -t, --target-directory=DIRECTORY  copy all SOURCE arguments into DIRECTORY
  -T, --no-target-directory    treat DEST as a normal file
  -u, --update                 copy only when the SOURCE file is newer
                                 than the destination file or when the
                                 destination file is missing
  -v, --verbose                explain what is being done
  -x, --one-file-system        stay on this file system
  -Z                           set SELinux security context of destination
                                 file to default type
      --context[=CTX]          like -Z, or if CTX is specified then set the
                                 SELinux or SMACK security context to CTX
      --help     display this help and exit
      --version  output version information and exit

By default, sparse SOURCE files are detected by a crude heuristic and the
corresponding DEST file is made sparse as well.  That is the behavior
selected by --sparse=auto.  Specify --sparse=always to create a sparse DEST
file whenever the SOURCE file contains a long enough sequence of zero bytes.
Use --sparse=never to inhibit creation of sparse files.

When --reflink[=always] is specified, perform a lightweight copy, where the
data blocks are copied only when modified.  If this is not possible the copy
fails, or if --reflink=auto is specified, fall back to a standard copy.

The backup suffix is '~', unless set with --suffix or SIMPLE_BACKUP_SUFFIX.
The version control method may be selected via the --backup option or through
the VERSION_CONTROL environment variable.  Here are the values:

  none, off       never make backups (even if --backup is given)
  numbered, t     make numbered backups
  existing, nil   numbered if numbered backups exist, simple otherwise
  simple, never   always make simple backups

As a special case, cp makes a backup of SOURCE when the force and backup
options are given and SOURCE and DEST are the same name for an existing,
regular file.
```

### 常用选项：

-a / -p：连同文件属性一起复制（用户组属性受 id 权限限制）

-i（interactive）：覆盖询问

-r（recursive）： 用于目录的递归复制

-u （update）：source 比 destination 新才复制

- 覆盖询问

```bash
[root@78063f0fe2e8 ~]# cp ~/.bashrc /tmp/bashrc
[root@78063f0fe2e8 ~]# cp -i ~/.bashrc /tmp/bashrc
cp: overwrite ‘/tmp/bashrc’? y
```

- 连同文件属性一起复制

```bash
[root@78063f0fe2e8 ~]# cd /tmp
[root@78063f0fe2e8 tmp]# cp /var/log/wtmp .
[root@78063f0fe2e8 tmp]# ls -l /var/log/wtmp wtmp
-rw-rw-r-- 1 root utmp 0 Apr  2 18:38 /var/log/wtmp
-rw-r--r-- 1 root root 0 Apr 21 06:23 wtmp
[root@78063f0fe2e8 tmp]# cp -a /var/log/wtmp wtmp2
[root@78063f0fe2e8 tmp]# ls -l /var/log/wtmp wtmp2
-rw-rw-r-- 1 root utmp 0 Apr  2 18:38 /var/log/wtmp
-rw-rw-r-- 1 root utmp 0 Apr  2 18:38 wtmp2
```

- 用于目录的递归复制

```bash
[root@78063f0fe2e8 tmp]# cp /etc /tmp
cp: omitting directory ‘/etc’
[root@78063f0fe2e8 tmp]# cp -r /etc /tmp
```

{{< asciinema 1wQwTsxT1J3Xs3MjPk8A7wgLZ >}}

## rm - remove 删除目录或文件

### Usage：

```bash
Usage: rm [OPTION]... FILE...
Remove (unlink) the FILE(s).

  -f, --force           ignore nonexistent files and arguments, never prompt
  -i                    prompt before every removal
  -I                    prompt once before removing more than three files, or
                          when removing recursively; less intrusive than -i,
                          while still giving protection against most mistakes
      --interactive[=WHEN]  prompt according to WHEN: never, once (-I), or
                          always (-i); without WHEN, prompt always
      --one-file-system  when removing a hierarchy recursively, skip any
                          directory that is on a file system different from
                          that of the corresponding command line argument
      --no-preserve-root  do not treat '/' specially
      --preserve-root   do not remove '/' (default)
  -r, -R, --recursive   remove directories and their contents recursively
  -d, --dir             remove empty directories
  -v, --verbose         explain what is being done
      --help     display this help and exit
      --version  output version information and exit

By default, rm does not remove directories.  Use the --recursive (-r or -R)
option to remove each listed directory, too, along with all of its contents.

To remove a file whose name starts with a '-', for example '-foo',
use one of these commands:
  rm -- -foo

  rm ./-foo

Note that if you use rm to remove a file, it might be possible to recover
some of its contents, given sufficient expertise and/or time.  For greater
assurance that the contents are truly unrecoverable, consider using shred.
```

### 常用选项：

-f（force）：忽略不存在的文件

-i（interactive）：询问删除

-r（recursive）：递归删除

```bash
sh-4.2# cd /tmp
sh-4.2# ls
bashrc		  mingle1  tmpxq2sylvo-ascii.cast  wtmp2
ks-script-hE5IPf  test1    wtmp			   yum.log
sh-4.2# rm -i bashrc
rm: remove regular file ‘bashrc’? y
sh-4.2# ls
ks-script-hE5IPf  mingle1  test1  tmpxq2sylvo-ascii.cast  wtmp	wtmp2  yum.log
sh-4.2# rmdir mingle1
sh-4.2# rmdir test1
rmdir: failed to remove ‘test1’: Directory not empty
sh-4.2# rm -rf test1
sh-4.2# ls
ks-script-hE5IPf  tmpxq2sylvo-ascii.cast  wtmp	wtmp2  yum.log
sh-4.2# touch ./-aaa-
sh-4.2# ls
-aaa-  ks-script-hE5IPf  tmpxq2sylvo-ascii.cast  wtmp  wtmp2  yum.log
sh-4.2# rm -f -aaa-
rm: invalid option -- 'a'
Try 'rm ./-aaa-' to remove the file ‘-aaa-’.
Try 'rm --help' for more information.
sh-4.2# rm -f ./-aaa-
sh-4.2# ls
ks-script-hE5IPf  tmpxq2sylvo-ascii.cast  wtmp	wtmp2  yum.log
sh-4.2# rm -rf wtmp*
sh-4.2# ls
ks-script-hE5IPf  tmpxq2sylvo-ascii.cast  yum.log
```

{{< asciinema eZnFGqsqST6IJDVts4f7MONRt >}}

## mv - move 移动文件或目录，更名

### Usage：

```bash
Usage: mv [OPTION]... [-T] SOURCE DEST
  or:  mv [OPTION]... SOURCE... DIRECTORY
  or:  mv [OPTION]... -t DIRECTORY SOURCE...
Rename SOURCE to DEST, or move SOURCE(s) to DIRECTORY.

Mandatory arguments to long options are mandatory for short options too.
      --backup[=CONTROL]       make a backup of each existing destination file
  -b                           like --backup but does not accept an argument
  -f, --force                  do not prompt before overwriting
  -i, --interactive            prompt before overwrite
  -n, --no-clobber             do not overwrite an existing file
If you specify more than one of -i, -f, -n, only the final one takes effect.
      --strip-trailing-slashes  remove any trailing slashes from each SOURCE
                                 argument
  -S, --suffix=SUFFIX          override the usual backup suffix
  -t, --target-directory=DIRECTORY  move all SOURCE arguments into DIRECTORY
  -T, --no-target-directory    treat DEST as a normal file
  -u, --update                 move only when the SOURCE file is newer
                                 than the destination file or when the
                                 destination file is missing
  -v, --verbose                explain what is being done
  -Z, --context                set SELinux security context of destination
                                 file to default type
      --help     display this help and exit
      --version  output version information and exit

The backup suffix is '~', unless set with --suffix or SIMPLE_BACKUP_SUFFIX.
The version control method may be selected via the --backup option or through
the VERSION_CONTROL environment variable.  Here are the values:

  none, off       never make backups (even if --backup is given)
  numbered, t     make numbered backups
  existing, nil   numbered if numbered backups exist, simple otherwise
  simple, never   always make simple backups
```

### 常用选项：

-f（force）：不询问直接覆盖

-i（ interactive）： 询问覆盖

-u（update）：source 新于 destination 才会 move

- 普通移动

```bash
sh-4.2# cd /tmp
sh-4.2# cp ~/.bashrc bashrc1
sh-4.2# cp ~/.bashrc bashrc2
sh-4.2# ls
bashrc1  bashrc2  ks-script-hE5IPf  tmp8u627nsx-ascii.cast  yum.log
sh-4.2# mkdir mvtest
sh-4.2# ls
bashrc1  bashrc2  ks-script-hE5IPf  mvtest  tmp8u627nsx-ascii.cast  yum.log
sh-4.2# mv bashrc1 bashrc2 mvtest
sh-4.2# ls
ks-script-hE5IPf  mvtest  tmp8u627nsx-ascii.cast  yum.log
sh-4.2# cd mvtest
sh-4.2# ls
bashrc1  bashrc2
```

- 重命名

```bash
sh-4.2# cd ..
sh-4.2# ls
ks-script-hE5IPf  mvtest  tmp8u627nsx-ascii.cast  yum.log
sh-4.2# mv mvtest mvtest2
sh-4.2# ls
ks-script-hE5IPf  mvtest2  tmp8u627nsx-ascii.cast  yum.log
```

{{< asciinema CyAOm5kpvEzi75HFNowZdIV23 >}}

