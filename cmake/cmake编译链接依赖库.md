<!--
 * @Author: jiyang Gui
 * @Date: 2022-10-24 10:49:28
 * @LastEditors: jiyang Gui
 * @LastEditTime: 2022-10-24 11:10:12
 * @Description: 
 * guijiyang@163.com
 * Copyright (c) 2022 by jiyang Gui/GuisGame, All Rights Reserved. 
-->
# cmake指定编译和链接所需的依赖库
比如使用llvm编译器并使用libc++ 替代标准库libstdc++
## 指定target的情况下

```cmake
target_compile_options(-stdlib=libc++)
target_link_options(--stdlib=libc++)
```

## 全局的情况下

```cmake
add_compile_options(--stdlib=libc++)
add_link_options(--stdlib=libc++)