<!--
 * @Author: jiyang Gui
 * @Date: 2022-04-10 19:58:28
 * @LastEditors: jiyang Gui
 * @LastEditTime: 2023-02-12 16:10:07
 * @Description: 
 * guijiyang@163.com
 * Copyright (c) 2022 by jiyang Gui/GuisGame, All Rights Reserved. 
-->

# Linux常用命令
## 最大文件打开数量
We knew the answer to this question once, back when the world was young and full of truth. Without hesitation, we’d have spouted “Just take the output of lsof | wc -l!” And it’s true enough, in a general sort of way.

But if you asked me the same question now, my answer would be: “Why do you want to know?”

Are you, for instance, attempting to determine whether the number of open files is exceeding the limit set in the kernel? Then the output of lsof will be practically useless.

Let’s back up…

On *nix systems, there is a limit set in the kernel on how many open file descriptors are allowed on the system. This may be compiled in, or it may be tunable on the fly. In Linux, the value of this parameter can be read from and written to the proc filesystem.

[root@srv-4 proc]# cat /proc/sys/fs/file-max 
52427
On this system 52,427 open file descriptors are permitted. We are unlikely to run out. If we wanted to change it, we’d do something like this:

[root@srv-4 proc]# echo "104854" > /proc/sys/fs/file-max 
[root@srv-4 proc]# cat /proc/sys/fs/file-max 
104854
(Warning, change kernel parameters on the fly only with great caution and good cause)

But how do we know how many file descriptors are being used?

[root@srv-4 proc]# cat /proc/sys/fs/file-nr
3391    969     52427
|	 |       |
|	 |       |
|        |       maximum open file descriptors
|        total free allocated file descriptors
total allocated file descriptors
(the number of file descriptors allocated since boot)
The number of open file descriptors is column 1 – column 2; 2325 in this case. (Note: we have read contradictory definitions of the second column in newsgroups. Some people say it is the number of used allocated file descriptors – just the opposite of what we’ve stated here. Luckily, we can prove that the second number is free descriptors. Just launch an xterm or any new process and watch the second number go down.)

What about the method we mentioned earlier, running lsof to get the number of open files?

[root@srv-4 proc]# lsof | wc -l
4140
Oh my. There is quite a discrepancy here. The lsof utility reports 4140 open files, while /proc/sys/fs/file-nr says, if our math is correct, 2325. So how many open files are there?

What is an open file?

Is an open file a file that is being used, or is it an open file descriptor? A file descriptor is a data structure used by a program to get a handle on a file, the most well know being 0,1,2 for standard in, standard out, and standard error.

The file-max kernel parameter refers to open file descriptors, and file-nr gives us the current number of open file descriptors. But lsof lists all open files, including files which are not using file descriptors – such as current working directories, memory mapped library files, and executable text files.

To illustrate, let’s examine the difference between the output of lsof for a given pid and the file descriptors listed for that pid in /proc.

Pick a PID, any PID

Let’s look at this process:

usr-4    2381  0.0  0.5  5168 2748 pts/14   S    14:42   0:01 vim openfiles.html
[root@srv-4 usr-4]# lsof | grep 2381
vim        2381 usr-4  cwd    DIR        3,8    4096   2621517 /n
vim        2381 usr-4  rtd    DIR        3,5    4096         2 /
vim        2381 usr-4  txt    REG        3,2 2160520     34239 /usr/bin/vim
vim        2381 usr-4  mem    REG        3,5   85420    144496 /lib/ld-2.2.5.so
vim        2381 usr-4  mem    REG        3,2     371     20974 /usr/lib/locale/LC_IDENTIFICATION
vim        2381 usr-4  mem    REG        3,2   20666    192622 /usr/lib/gconv/gconv-modules.cache
vim        2381 usr-4  mem    REG        3,2      29     20975 /usr/lib/locale/LC_MEASUREMENT
vim        2381 usr-4  mem    REG        3,2      65     20979 /usr/lib/locale/LC_TELEPHONE
vim        2381 usr-4  mem    REG        3,2     161     19742 /usr/lib/locale/LC_ADDRESS
vim        2381 usr-4  mem    REG        3,2      83     20977 /usr/lib/locale/LC_NAME
vim        2381 usr-4  mem    REG        3,2      40     20978 /usr/lib/locale/LC_PAPER
vim        2381 usr-4  mem    REG        3,2      58     51851 /usr/lib/locale/LC_MESSAGES/SYSL
vim        2381 usr-4  mem    REG        3,2     292     20976 /usr/lib/locale/LC_MONETARY
vim        2381 usr-4  mem    REG        3,2   22592     99819 /usr/lib/locale/LC_COLLATE
vim        2381 usr-4  mem    REG        3,2    2457     20980 /usr/lib/locale/LC_TIME
vim        2381 usr-4  mem    REG        3,2      60     35062 /usr/lib/locale/LC_NUMERIC
vim        2381 usr-4  mem    REG        3,2  290511     64237 /usr/lib/libncurses.so.5.2
vim        2381 usr-4  mem    REG        3,2   24565     64273 /usr/lib/libgpm.so.1.18.0
vim        2381 usr-4  mem    REG        3,5   11728    144511 /lib/libdl-2.2.5.so
vim        2381 usr-4  mem    REG        3,5   22645    144299 /lib/libcrypt-2.2.5.so
vim        2381 usr-4  mem    REG        3,5   10982    144339 /lib/libutil-2.2.5.so
vim        2381 usr-4  mem    REG        3,5  105945    144516 /lib/libpthread-0.9.so
vim        2381 usr-4  mem    REG        3,5  169581    144512 /lib/libm-2.2.5.so
vim        2381 usr-4  mem    REG        3,5 1344152    144297 /lib/libc-2.2.5.so
vim        2381 usr-4  mem    REG        3,2  173680    112269 /usr/lib/locale/LC_CTYPE
vim        2381 usr-4  mem    REG        3,5   42897    144321 /lib/libnss_files-2.2.5.so
vim        2381 usr-4    0u   CHR     136,14                16 /dev/pts/14
vim        2381 usr-4    1u   CHR     136,14                16 /dev/pts/14
vim        2381 usr-4    2u   CHR     136,14                16 /dev/pts/14
vim        2381 usr-4    4u   REG        3,8   12288   2621444 /n/.openfiles.html.swp
[root@srv-4 usr-4]# lsof | grep 2381 | wc -l
30
[root@srv-4 fd]# ls -l /proc/2381/fd/
total 0
lrwx------    1 usr-4   usr-4         64 Jul 30 15:16 0 -> /dev/pts/14
lrwx------    1 usr-4   usr-4         64 Jul 30 15:16 1 -> /dev/pts/14
lrwx------    1 usr-4   usr-4         64 Jul 30 15:16 2 -> /dev/pts/14
lrwx------    1 usr-4   usr-4         64 Jul 30 15:16 4 -> /n/.openfiles.html.swp
Quite a difference. This process has only four open file descriptors, but there are thirty open files associated with it. Some of the open files which are not using file descriptors: library files, the program itself (executable text), and so on as listed above.

These files are accounted for elsewhere in the kernel data structures (cat /proc/PID/maps to see the libraries, for instance), but they are not using file descriptors and therefore do not exhaust the kernel’s file descriptor maximum.

## ldd
ldd命令可以查看库或可执行文件的所有依赖库的情况：有哪些库，是否缺失。
```shell
$ ldd concept_template
linux-vdso.so.1 (0x00007fffa01fd000)
libc++.so.1 => /usr/local/llvm-14.0.0/lib/x86_64-linux-gnu/libc++.so.1 (0x00007fbb3488c000)
libc++abi.so.1 => /usr/local/llvm-14.0.0/lib/x86_64-linux-gnu/libc++abi.so.1 (0x00007fbb3484b000)
libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007fbb346fc000)
libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007fbb346e1000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fbb344ef000)
libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fbb344cc000)
librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007fbb344c0000)
libatomic.so.1 => /lib/x86_64-linux-gnu/libatomic.so.1 (0x00007fbb344b6000)
/lib64/ld-linux-x86-64.so.2 (0x00007fbb3498d000)

```
## 设置库链接路径
通过在`/etc/ld.so.conf`下添加路径，可以增加运行时依赖库查找的路径，注意这不同于编译时候的链接。
```shell
# 这是我自己添加的查找依赖库的路径
$ cat /etc/ld.so.conf.d/llvm.conf 
/usr/local/llvm-14.0.0/lib
/usr/local/llvm-14.0.0/lib/x86_64-linux-gnu
```
Linux 的先辈 Unix 还有一个环境变量 - LD_LIBRARY_PATH 来处理非标准路经的共享库。ld.so 加载共享库的时候，也会查找这个变量所设置的路经。但是，有不少声音主张要避免使用 LD_LIBRARY_PATH 变量，尤其是作为全局变量。
## 设置编译器链接库路径
这是设置编译器在链接时候的库搜素路径
```shell
# 当前终端生效
export LIBRARY_PATH=libPath
# 永久生效，修改/etc/profile或者~/.bashrc文件
export LIBRARY_PATH=libPath:$LIBRARY_PATH
source /etc/profile or ~/.bashrc
#编译命令中添加查找库
g++ -LlibPath
```

## find

在指定路径查找文件：
```shell
find <path> -name <doc>
```

### objdump

查看文件信息
```shell
objdump <option> <name>
```

### readelf
查看elf文件(executable and linkable format)

```shell
readelf <option> <name>
```

### c++filt还原编译后的c++函数名
```shell
c++filt _ZN5ATestC2Ev
```
输出：
```shell
ATest::ATest()
```