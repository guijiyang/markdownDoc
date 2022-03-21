#<center>macro和function</center>

## 相同点
macro的形式如下:
```php
macro(<name> [arg1 [arg2 [arg3 ...]]])
  COMMAND1(ARGS ...)
  COMMAND2(ARGS ...)
  ...
endmacro(<name>)
```

function的形式如下:
```php
function(<name> [arg1 [arg2 [arg3 ...]]])
  COMMAND1(ARGS ...)
  COMMAND2(ARGS ...)
  ...
function(<name>)
```
定义一个名为name的宏(函数),arg1...是传入的参数.
宏和函数都支持动态参数:
|变量|说明|
|:-:|:-:|
|ARGV0,ARGV1,...|顺序代表传入的参数|
|ARGC|传入的参数个数|
|ARGV|所有参数的list|
|ARGN|声明的参数之后的所有参数|

## 不同点

macro的动态参数不是通常CMake意义上的变量.它们是字符串替换,很像c++的宏,所以必须写成如下形式:
```php
if(${ARGV1})
if(DEFINED ${ARGV2})
if(${ARGC} GREATER 2)
foreach(loop_var IN LISTS ${ARGN})
or
set(list_var "${ARGN}")
foreach(loop_var IN LISTS list_var)
 ```
 function 中的变量不需要加"{}".