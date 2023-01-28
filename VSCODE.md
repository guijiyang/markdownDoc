# <center>VSCode 配置</center>
<!-- @import "my-style.less" -->
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [VSCode 配置](#-centervscode-配置center-)
  - [常用快捷键](#-常用快捷键-)
    - [通用](#-通用-)
    - [基本编辑](#-基本编辑-)
    - [导航](#-导航-)
    - [查找和替换](#-查找和替换-)
    - [多光标选择](#-多光标选择-)
    - [富文本编辑](#-富文本编辑-)
    - [编辑器管理](#-编辑器管理-)
    - [文件管理](#-文件管理-)
    - [显示](#-显示-)
    - [调试](#-调试-)
    - [终端](#-终端-)
    - [插件快捷键](#-插件快捷键-)

<!-- /code_chunk_output -->

## 常用快捷键

### 通用

|快捷键|作用|
|:-|:-|
|Ctrl+Shift+P|显示所有命令|
|Ctrl+Shift+N|打开新的窗口(实例)|
|Ctrl+Shift+W|关闭窗口(实例)|
|Ctrl+,|打开用户设置|
|Ctrl+K Ctrl+S|打开快捷键设置|
|Esc|退出，常用操作|

### 基本编辑

|快捷键|作用|
|:-|:-|
|Ctrl+X|剪贴|
|Ctrl+C|复制|
|Ctrl+V|粘贴|
|Shift+Alt+Up(Down)|向上(下)行复制当前行|
|Ctrl+Shift+Left(Right)|光标按词左(右)选|
|Shift+Alt+Right(Left)|展开(收起)选择|
|Shift+Left(Right)|光标按单元格左(右)选|
|Shift+Up(Down)|从光标位置进行行选择|
|Ctrl+Shift+K|删除行|
|Ctrl+Shift+Enter|向当前行上插入行|
|Ctrl+Enter|向当前行下插入行|
|Ctrl+Shift+\ |跳转到匹配的括号|
|Ctrl+[(])|行左(右)缩进|
|Home(End)|转到行首(尾)|
|Ctrl+Home(End)|转到文首(尾)|
|Ctrl+Up(Down)|编辑区按行向上(下)滚动|
|PgUp(PgDn)|切换到上(下)页|
|Ctrl+Shift+[(])|折叠(展开)片段|
|Ctrl+K Ctrl+[(])|折叠(展开)所有子片段|
|Ctrl+K Ctrl+0(J)|折叠(展开)所有片段|
|Ctrl+K Ctrl+C|行注释|
|Ctrl+K Ctrl+U|行注释取消|
|Ctrl+/|切换行注释|
|Shift+Alt+A|切换选中块注释|
|Ctrl+U|转换成大写|
|Ctrl+Shift+U|转换成小写|

### 导航

|快捷键|作用|
|:-:|:-|
|Ctrl+T|弹出工作区中选中符号的所有位置，随后可以跳转到指定的位置|
|Ctrl+G|转到指定行|
|Ctrl+P|转到文件|
|Ctrl+Shift+O|弹出当前编辑器中的所有符号，随后可以选择其中的符号跳转到该位置|
|F8|跳转到当前编辑器中的下一个错误或者警告|
|Shift+F8|跳转到当前编辑器中的上一个错误或者警告|
|Ctrl+Shift+Tab|快速打开编辑器组中最近使用频率最低的编辑器|
|Alt+Left(Right)|后退(前进)|
|Ctrl+M|切换TAB移动焦点|

### 查找和替换

|快捷键|作用|
|:-:|:-|
|Ctrl+F|查找|
|Ctrl+H|替换|
|F3|查找下一个|
|Shift+F3|查找上一个|
|Ctrl+D|未选中时，可以选中光标处匹配项；选中时，可以增加选中匹配项，可以多次操作|
|Ctrl+K Ctrl+D|将当前选中的对象光标移动到下一个相同的对象|

### 多光标选择

|快捷键|作用|
|:-:|:-|
|Ctrl+Alt+Up(Down)|向上(下)插入光标|
|Ctrl+L|选中当前行|
|Ctrl+Shift+L|选中当前光标所处的对象和所有匹配对象|
|Ctrl+F2|选中当前词和所有匹配词|
|Shift+Alt+(drag Mouse)|列框选|
|Ctrl+Shift+Alt+(arrow key)|列框选|
|Ctrl+Shift+Alt+PgUp(PgDn)|页框选|
|Shift+Alt+Left(Right)|缩选(扩选)|

### 富文本编辑

|快捷键|作用|
|:-|:-|
|Ctrl+Space, Ctrl+I|触发提示|
|Shift+Alt+F|格式化文档|
|F12|跳转到定义或声明(C/C++)|
|Ctrl+K F12|在侧边编辑器打开定义|
|Ctrl+.|快速修复|
|Shift+F12|弹窗显示所有引用|
|Shift+Alt+F12|查找所用引用项，在侧边栏引用页显示|
|F2|重命名符号|
|Ctrl+K M|改变文件编程语言|

### 编辑器管理

|快捷键|作用|
|:-|:-|
|Ctrl+F4, Ctrl+W|关闭当前的编辑文本|
|Ctrl+K F|关闭工作区|
|Ctrl+\ |拆分编辑器，增加一个编辑组|
|Ctrl+1，2，3|聚焦第1、2、3编辑组|
|Ctrl+K Ctrl+Left(Right)|聚焦到前(后)一个编辑组|
|Ctrl+Shift+PgUp(PgDn)|在当前编辑器组中将编辑器向前(后)移动|
|Ctrl+K Left(Right), Ctrl+Alt+Left(Right)|将当前编辑器组向前(后)移动|

### 文件管理

|快捷键|作用|
|:-|:-|
|Ctrl+N|新建文本|
|Ctrl+O|打开文件|
|Ctrl+S|保存当前编辑器文件|
|Ctrl+Shift+S|保存当前所有打开的编辑器文件|
|Ctrl+Shift+T|重新打开已关闭的编辑器|
|Ctrl+K Enter|保留预览模式的编辑器常开|
|Ctrl+(Shift)+Tab|切换编辑器|
|Ctrl+K P|复制当前编辑器文本路径|
|Ctrl+K R|打开当前编辑器文件的文件夹|
|Ctrl+K O|在新的窗口(实例)打开当前编辑器文件|

### 显示

|作用|快捷键|
|:-|:-|
|F11|切换全屏|
|Ctrl+=(-)|放大(缩小)|
|Ctrl+B|切换侧边栏显隐|
|Ctrl+Shift+E|显示资源管理器、切换焦点|
|Ctrl+Shift+F|显示搜索栏|
|Ctrl+Shift+J|切换搜索栏细节设置|
|Ctrl+Shift+G|显示资源控制栏|
|Ctrl+Shift+D|显示调试栏|
|Ctrl+Shift+H|显示替换栏|
|Ctrl+Shift+V|显示markdown预览|
|Ctrl+K V|新开编辑器组，显示markdown预览|
|Ctrl+K Z|禅模式|

### 调试

|快捷键|作用|
|:-|:-|
|F9|开关断点|
|F5|开启、继续调试|
|Shift+F5|停止调试|
|F11/Shift+F11|步入、步出|
|F10|跳过|
|Ctrl+K Ctrl+I|显示悬停|

### 终端

|快捷键|作用|
|:-|:-|
|Ctrl+`|切换集成终端|
|Ctrl+Shift+`|创建新终端|
|Ctrl+Shift+Y|切换调试控制台|
|Ctrl+Shift+M|展示问题面板|

### 插件快捷键

|快捷键|作用|
|:-|:-|
|Ctrl+Shift+Alt+t|google翻译|
|Shift+Alt+t|google翻译候选结果|
