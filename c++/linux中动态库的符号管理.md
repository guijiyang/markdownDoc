<!--
 * @Author: jiyang Gui
 * @Date: 2021-03-03 23:29:25
 * @LastEditors: jiyang Gui
 * @LastEditTime: 2023-02-11 18:56:08
 * @Description: 
 * guijiyang@163.com
 * Copyright (c) 2023 by jiyang Gui/GuisGame, All Rights Reserved. 
-->
#<center>linux中动态库的符号管理</center>

管理GNU/Linux符号导出的方式主要是以下几种:
- 源码注释/装饰:
- - 方法1:` -fvisibility=hidden` 和`__attribute__((visibility("default")))`配合使用
- - 方法2: `#pragma GCC visibility`
- 版本脚本:
- - 方法3:版本脚本(aka ''symbol maps")传递给链接器(eg,` -Wl,--version-script=<version script file>`)
- static函数.

编译时添加参数` -fvisibility=hidden`将所有符号隐藏,不会出现在动态符号表`.dynsym`中,但还是留在符号表`.symtab`中用于静态链接.在需要对外的符号前添加`__attribute__((visibility("default")))`属性保证符号对外可见.
也可以反过来将` -fvisibility=default`或不设置这个属性,默认都是对外可见的,然后在不对外可见的符号前添加`__attribute__((visibility("hidden")))`来隐藏该符号.

