[CMake教程](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id1)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#cmake-tutorial "固定到该标题")
======================================================================================================================================================================

内容

*   [CMake教程](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#cmake-tutorial)
    
    *   [基本起点（步骤1）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#a-basic-starting-point-step-1)
        
        *   [添加版本号和配置的头文件](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-a-version-number-and-configured-header-file)
            
        *   [指定C ++标准](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#specify-the-c-standard)
            
        *   [构建和测试](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#build-and-test)
            
    *   [添加库（步骤2）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-a-library-step-2)
        
    *   [添加库的使用要求（步骤3）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-usage-requirements-for-library-step-3)
        
    *   [安装和测试（步骤4）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#installing-and-testing-step-4)
        
        *   [安装规则](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#install-rules)
            
        *   [测试支持](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#testing-support)
            
    *   [添加系统自检（步骤5）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-system-introspection-step-5)
        
        *   [指定编译定义](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#specify-compile-definition)
            
    *   [添加自定义命令和生成的文件（步骤6）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-a-custom-command-and-generated-file-step-6)
        
    *   [生成安装程序（步骤7）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#building-an-installer-step-7)
        
    *   [添加对仪表板的支持（步骤8）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-support-for-a-dashboard-step-8)
        
    *   [混合静态和共享（步骤9）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#mixing-static-and-shared-step-9)
        
    *   [添加生成器表达式（步骤10）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-generator-expressions-step-10)
        
    *   [添加导出配置（步骤11）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-export-configuration-step-11)
        
    *   [打包调试和发布（步骤12）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#packaging-debug-and-release-step-12)
        

CMake教程提供了逐步指南，涵盖了CMake可以解决的常见构建系统问题。了解示例项目中各个主题如何协同工作将非常有帮助。教程文档和示例的源代码可以`Help/guide/tutorial`在CMake源代码树的目录中找到 。每个步骤都有其自己的子目录，其中包含可以用作起点的代码。教程示例是渐进式的，因此每个步骤都为上一步提供了完整的解决方案。

[基本起点（步骤1）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id2)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#a-basic-starting-point-step-1 "固定到该标题")
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

最基本的项目是从源代码文件构建的可执行文件。对于简单项目，只需要三行`CMakeLists.txt`文件。这将是本教程的起点。`CMakeLists.txt`在`Step1`目录中创建一个 文件，如下所示：
```makefile
cmake_minimum_required(VERSION  3.10)

# set the project name
project(Tutorial)

# add the executable
add_executable(Tutorial  tutorial.cxx)
```

请注意，此示例在`CMakeLists.txt`文件中使用小写命令。CMake支持大写，小写和大小写混合命令。目录中`tutorial.cxx`提供的源代码，`Step1`可用于计算数字的平方根。

### [添加一个版本号和配置的头文件](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id3)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-a-version-number-and-configured-header-file "固定到该标题")

我们将添加的第一个功能是为我们的可执行文件和项目提供版本号。虽然我们可以在源代码中专门执行此操作，但是使用 `CMakeLists.txt`可以提供更大的灵活性。

首先，修改`CMakeLists.txt`文件以使用[`project()`](https://cmake.org/cmake/help/latest/command/project.html#command:project "项目") 命令设置项目名称和版本号。

cmake\_minimum\_required(VERSION  3.10)

\# set the project name and version
project(Tutorial  VERSION  1.0)

然后，配置头文件以将版本号传递给源代码：

configure_file(TutorialConfig.h.in  TutorialConfig.h)

由于配置的文件将被写入二进制树，因此我们必须将该目录添加到路径列表中以搜索包含文件。将以下行添加到`CMakeLists.txt`文件的末尾：

target\_include\_directories(Tutorial  PUBLIC
  "${PROJECT\_BINARY\_DIR}"
  )

使用您喜欢的编辑器，`TutorialConfig.h.in`在源目录中创建以下内容：

//  the  configured  options  and  settings  for  Tutorial
#define Tutorial\_VERSION\_MAJOR @Tutorial\_VERSION\_MAJOR@
#define Tutorial\_VERSION\_MINOR @Tutorial\_VERSION\_MINOR@

当CMake配置此头文件时，`@Tutorial_VERSION_MAJOR@`和的值 `@Tutorial_VERSION_MINOR@`将被替换。

接下来修改`tutorial.cxx`以包括配置的头文件 `TutorialConfig.h`。

最后，通过`tutorial.cxx`如下更新来打印出版本号：

  if (argc < 2) {
    // report version
    std::cout << argv\[0\] << " Version " << Tutorial\_VERSION\_MAJOR << "."
              << Tutorial\_VERSION\_MINOR << std::endl;
    std::cout << "Usage: " << argv\[0\] << " number" << std::endl;
    return 1;
  }

### [指定C ++标准](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id4)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#specify-the-c-standard "固定到该标题")

接下来，我们`atof`用 `std::stod`in 替换为我们的项目添加一些C ++ 11功能`tutorial.cxx`。同时，删除 。`#include <cstdlib>`

  const double inputValue = std::stod(argv\[1\]);

我们将需要在CMake代码中明确声明应使用正确的标志。在CMake中启用对特定C ++标准的支持的最简单方法是使用[`CMAKE_CXX_STANDARD`](https://cmake.org/cmake/help/latest/variable/CMAKE_CXX_STANDARD.html#variable:CMAKE_CXX_STANDARD "CMAKE_CXX_STANDARD")变量。对于本教程，请设置[`CMAKE_CXX_STANDARD`](https://cmake.org/cmake/help/latest/variable/CMAKE_CXX_STANDARD.html#variable:CMAKE_CXX_STANDARD "CMAKE_CXX_STANDARD")将`CMakeLists.txt`文件中的变量设置 为11并[`CMAKE_CXX_STANDARD_REQUIRED`](https://cmake.org/cmake/help/latest/variable/CMAKE_CXX_STANDARD_REQUIRED.html#variable:CMAKE_CXX_STANDARD_REQUIRED "CMAKE_CXX_STANDARD_REQUIRED") 改为True：

cmake\_minimum\_required(VERSION  3.10)

\# set the project name and version
project(Tutorial  VERSION  1.0)

\# specify the C++ standard
set(CMAKE\_CXX\_STANDARD  11)
set(CMAKE\_CXX\_STANDARD_REQUIRED  True)

### [构建和测试](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id5)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#build-and-test "固定到该标题")

跑过 [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）") 配置项目，然后使用您选择的构建工具进行构建。

例如，从命令行我们可以导航到`Help/guide/tutorial`CMake源代码树的 目录并运行以下命令：

mkdir Step1_build
cd Step1_build
cmake ../Step1
cmake --build .

导航到构建Tutorial的目录（可能是make目录或Debug或Release构建配置子目录），然后运行以下命令：

Tutorial 4294967296
Tutorial 10
Tutorial

[添加一个库（第二步）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id6)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-a-library-step-2 "固定到该标题")
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

现在，我们将库添加到我们的项目中。该库将包含我们自己的实现，用于计算数字的平方根。然后可执行文件可以使用此库而不是编译器提供的标准平方根函数。

在本教程中，我们将库放入名为的子目录中`MathFunctions`。该目录已经包含一个头文件 `MathFunctions.h`和一个源文件`mysqrt.cxx`。源文件具有一个称为的`mysqrt`功能，该功能提供与编译器`sqrt`功能相似的功能。

将以下一个行`CMakeLists.txt`文件添加到`MathFunctions` 目录：

add_library(MathFunctions  mysqrt.cxx)

为了利用新库，我们将添加一个 [`add_subdirectory()`](https://cmake.org/cmake/help/latest/command/add_subdirectory.html#command:add_subdirectory "add_sub目录") 调用顶级`CMakeLists.txt`文件，以便构建库。我们将新库添加到可执行文件，并添加`MathFunctions`为包含目录，以便`mqsqrt.h`可以找到头文件。现在，顶级`CMakeLists.txt`文件的最后几行应如下所示：

\# add the MathFunctions library
add_subdirectory(MathFunctions)

\# add the executable
add_executable(Tutorial  tutorial.cxx)

target\_link\_libraries(Tutorial  PUBLIC  MathFunctions)

\# add the binary tree to the search path for include files
\# so that we will find TutorialConfig.h
target\_include\_directories(Tutorial  PUBLIC
  "${PROJECT\_BINARY\_DIR}"
  "${PROJECT\_SOURCE\_DIR}/MathFunctions"
  )

现在让我们将MathFunctions库设为可选。虽然对于本教程而言确实没有任何必要，但是对于较大的项目，这是常见的情况。第一步是向顶层`CMakeLists.txt`文件添加一个选项 。

option(USE_MYMATH  "Use tutorial provided math implementation"  ON)

\# configure a header file to pass some of the CMake settings
\# to the source code
configure_file(TutorialConfig.h.in  TutorialConfig.h)

此选项将显示在 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）") 和 [`ccmake`](https://cmake.org/cmake/help/latest/manual/ccmake.1.html#manual:ccmake(1) "ccmake（1）") 用户可以更改的默认值ON。此设置将存储在缓存中，因此用户无需在每次在构建目录上运行CMake时都设置该值。

下一个更改是使建立和链接MathFunctions库成为条件。为此，我们将顶级`CMakeLists.txt` 文件的末尾更改为如下所示：

if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND  EXTRA_LIBS  MathFunctions)
  list(APPEND  EXTRA_INCLUDES  "${PROJECT\_SOURCE\_DIR}/MathFunctions")
endif()

\# add the executable
add_executable(Tutorial  tutorial.cxx)

target\_link\_libraries(Tutorial  PUBLIC  ${EXTRA_LIBS})

\# add the binary tree to the search path for include files
\# so that we will find TutorialConfig.h
target\_include\_directories(Tutorial  PUBLIC
  "${PROJECT\_BINARY\_DIR}"
  ${EXTRA_INCLUDES}
  )

请注意，使用变量`EXTRA_LIBS`来收集所有可选库，以便以后链接到可执行文件中。该变量 `EXTRA_INCLUDES`类似地用于可选的头文件。在处理许多可选组件时，这是一种经典方法，我们将在下一步中介绍现代方法。

对源代码的相应更改非常简单。首先，如果需要，请在`tutorial.cxx`中包含`MathFunctions.h`标头：

#ifdef USE_MYMATH
\#  include "MathFunctions.h"
#endif

然后，在同一文件中，`USE_MYMATH`控制使用哪个平方根函数：

#ifdef USE_MYMATH
  const double outputValue = mysqrt(inputValue);
#else
  const double outputValue = sqrt(inputValue);
#endif

由于源代码现在需要，`USE_MYMATH`我们可以`TutorialConfig.h.in`使用以下行将其添加到 其中：

#cmakedefine USE_MYMATH

**练习**：为什么`TutorialConfig.h.in` 在选项之后进行配置很重要`USE_MYMATH`？如果我们将两者倒置会怎样？

跑过 [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）")配置项目，然后使用您选择的构建工具进行构建。然后运行构建的Tutorial可执行文件。

使用 [`ccmake`](https://cmake.org/cmake/help/latest/manual/ccmake.1.html#manual:ccmake(1) "ccmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）") 更新的值`USE_MYMATH`。重新生成并再次运行本教程。sqrt或mysqrt哪个函数可提供更好的结果？

[添加库的使用要求（步骤3）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id7)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-usage-requirements-for-library-step-3 "固定到该标题")
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

使用要求可以更好地控制库或可执行文件的链接并包含行，同时还可以更好地控制CMake内部目标的传递属性。利用使用需求的主要命令是：

> *   [`target_compile_definitions()`](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions "target_compile_definitions")
>     
> *   [`target_compile_options()`](https://cmake.org/cmake/help/latest/command/target_compile_options.html#command:target_compile_options "target_compile_options")
>     
> *   [`target_include_directories()`](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories "target_include_directories")
>     
> *   [`target_link_libraries()`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries")
>     

让我们从[添加库（第2步）中](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-a-library-step-2)重构代码，以使用使用率需求的现代CMake方法。我们首先声明，链接到MathFunctions的任何人都需要包括当前源目录，而MathFunctions本身不需要。因此这可能成为`INTERFACE`使用要求。

记住`INTERFACE`是指消费者需要的东西，而生产者则不需要。将以下行添加到的末尾 `MathFunctions/CMakeLists.txt`：

target\_include\_directories(MathFunctions
  INTERFACE  ${CMAKE\_CURRENT\_SOURCE_DIR}
  )

既然我们已经指定了MathFunction的使用要求，我们就可以安全地`EXTRA_INCLUDES`从顶级`CMakeLists.txt`（这里）删除对变量 的使用：

if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND  EXTRA_LIBS  MathFunctions)
endif()

和这里：

target\_include\_directories(Tutorial  PUBLIC
  "${PROJECT\_BINARY\_DIR}"
  )

完成后，运行 [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）")配置项目，然后使用您选择的构建工具或通过构建目录进行构建。`cmake --build .`

[安装和测试（步骤4）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id8)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#installing-and-testing-step-4 "固定到该标题")
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

现在，我们可以开始为项目添加安装规则和测试支持。

### [安装规则](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id9)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#install-rules "固定到该标题")

安装规则非常简单：对于MathFunctions，我们要安装库和头文件，对于应用程序，我们要安装可执行文件和已配置的头文件。

因此，`MathFunctions/CMakeLists.txt`我们最后添加：

install(TARGETS  MathFunctions  DESTINATION  lib)
install(FILES  MathFunctions.h  DESTINATION  include)

并在顶层末尾`CMakeLists.txt`添加：

install(TARGETS  Tutorial  DESTINATION  bin)
install(FILES  "${PROJECT\_BINARY\_DIR}/TutorialConfig.h"
  DESTINATION  include
  )

这是创建本教程的基本本地安装所需的全部。

跑过 [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）")配置项目，然后使用您选择的构建工具进行构建。使用以下`install` 选项运行安装步骤[`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）")命令（在3.15中引入，较早版本的CMake必须使用），或者从IDE 构建目标。这将安装适当的头文件，库和可执行文件。`make install``INSTALL`

CMake变量 [`CMAKE_INSTALL_PREFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html#variable:CMAKE_INSTALL_PREFIX "CMAKE_INSTALL_PREFIX")用于确定文件的安装根目录。如果使用定制安装目录，则可以通过参数指定。对于多配置工具，请使用参数指定配置。`cmake --install``--prefix``--config`

验证已安装的教程正在运行。

### [测试支持](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id10)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#testing-support "固定到该标题")

接下来让我们测试我们的应用程序。在顶级`CMakeLists.txt` 文件的末尾，我们可以启用测试，然后添加一些基本测试以验证应用程序是否正常运行。

enable_testing()

\# does the application run
add_test(NAME  Runs  COMMAND  Tutorial  25)

\# does the usage message work?
add_test(NAME  Usage  COMMAND  Tutorial)
set\_tests\_properties(Usage
  PROPERTIES  PASS\_REGULAR\_EXPRESSION  "Usage:.*number"
  )

\# define a function to simplify adding tests
function(do_test  target  arg  result)
  add_test(NAME  Comp${arg}  COMMAND  ${target}  ${arg})
  set\_tests\_properties(Comp${arg}
  PROPERTIES  PASS\_REGULAR\_EXPRESSION  ${result}
  )
endfunction(do_test)

\# do a bunch of result based tests
do_test(Tutorial  4  "4 is 2")
do_test(Tutorial  9  "9 is 3")
do_test(Tutorial  5  "5 is 2.236")
do_test(Tutorial  7  "7 is 2.645")
do_test(Tutorial  25  "25 is 5")
do_test(Tutorial  -25  "-25 is \[-nan|nan|0\]")
do_test(Tutorial  0.0001  "0.0001 is 0.01")

第一个测试只是验证应用程序正在运行，没有段错误或其他崩溃，并且返回值为零。这是CTest测试的基本形式。

下一个测试利用 [`PASS_REGULAR_EXPRESSION`](https://cmake.org/cmake/help/latest/prop_test/PASS_REGULAR_EXPRESSION.html#prop_test:PASS_REGULAR_EXPRESSION "PASS_REGULAR_EXPRESSION")测试属性，以验证测试的输出包含某些字符串。在这种情况下，请验证在提供了错误数量的参数时是否打印了用法消息。

最后，我们有一个名为的函数`do_test`，用于运行应用程序，并验证所计算的平方根对于给定输入是否正确。对于的每次调用`do_test`，都会根据传递的参数将另一个测试与名称，输入和预期结果一起添加到项目中。

重建应用程序，然后CD到二进制目录并运行 [`ctest`](https://cmake.org/cmake/help/latest/manual/ctest.1.html#manual:ctest(1) "ctest（1）")可执行文件：和。对于多配置生成器（例如Visual Studio），必须指定配置类型。例如，要在“调试”模式下运行测试，请使用 build目录（而非Debug子目录！）。或者，从IDE 构建目标。`ctest -N``ctest -VV``ctest -C Debug -VV``RUN_TESTS`

[添加系统自检（步骤5）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id11)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-system-introspection-step-5 "固定到该标题")
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

让我们考虑将一些代码添加到我们的项目中，这取决于目标平台可能不具备的功能。对于此示例，我们将添加一些代码，具体取决于目标平台是否具有`log`和`exp` 功能。当然，几乎每个平台都具有这些功能，但对于本教程而言，它们并不常见。

如果平台具有`log`和`exp`函数,则我们将使用它们来计算函数中的平方根`mysqrt`。我们首先使用 `CMakeLists.txt`顶层模块[`CheckSymbolExists`](https://cmake.org/cmake/help/latest/module/CheckSymbolExists.html#module:CheckSymbolExists "CheckSymbolExists")测试这些函数的可用性。我们将在`TutorialConfig.h.in`中使用新的定义 ，因此请确保在配置该文件之前进行设置。

include(CheckSymbolExists)
set(CMAKE\_REQUIRED\_LIBRARIES  "m")
check\_symbol\_exists(log  "math.h"  HAVE_LOG)
check\_symbol\_exists(exp  "math.h"  HAVE_EXP)

现在让我们添加这些定义，`TutorialConfig.h.in`以便我们可以从中使用它们`mysqrt.cxx`：

// does the platform provide exp and log functions?
#cmakedefine HAVE_LOG
#cmakedefine HAVE_EXP

如果`log`和`exp`在系统上可用，那么我们将使用它们来计算函数中的平方根`mysqrt`。将以下代码添加到中的`mysqrt`函数中`MathFunctions/mysqrt.cxx`（`#endif`返回结果前不要忘了 ！）：

#if defined(HAVE\_LOG) && defined(HAVE\_EXP)
  double result = exp(log(x) * 0.5);
  std::cout << "Computing sqrt of " << x << " to be " << result
            << " using log and exp" << std::endl;
#else
  double result = x;

我们还需要修改`mysqrt.cxx`为包括`cmath`。

#include <cmath>

跑过 [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）") 配置项目，然后使用所选的构建工具进行构建，并运行Tutorial可执行文件。

您会注意到我们没有使用`log`和`exp`，即使我们认为它们应该可用。我们应该很快认识到，我们都忘记了，包括`TutorialConfig.h`在`mysqrt.cxx`。

我们还需要更新，`MathFunctions/CMakeLists.txt`以便`mysqrt.cxx` 知道此文件的位置：

target\_include\_directories(MathFunctions
  INTERFACE  ${CMAKE\_CURRENT\_SOURCE_DIR}
  PRIVATE  ${CMAKE\_BINARY\_DIR}
  )

进行此更新之后，继续并再次构建项目并运行已构建的Tutorial可执行文件。如果`log`和`exp`仍未使用，请`TutorialConfig.h`从构建目录打开生成的文件。也许它们在当前系统上不可用？

sqrt或mysqrt哪个函数现在可以提供更好的结果？

### [指定编译定义](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id12)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#specify-compile-definition "固定到该标题")

除了in之外，还有更好的地方供我们保存`HAVE_LOG`和`HAVE_EXP`值`TutorialConfig.h`吗？让我们尝试使用 [`target_compile_definitions()`](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions "target_compile_definitions")。

首先，从中删除定义`TutorialConfig.h.in`。我们不再需要包含`TutorialConfig.h`from `mysqrt.cxx`或多余的include in `MathFunctions/CMakeLists.txt`。

接下来，我们可以将check `HAVE_LOG`和`HAVE_EXP`移至 `MathFunctions/CMakeLists.txt`，然后将这些值指定为`PRIVATE` 编译定义。

include(CheckSymbolExists)
set(CMAKE\_REQUIRED\_LIBRARIES  "m")
check\_symbol\_exists(log  "math.h"  HAVE_LOG)
check\_symbol\_exists(exp  "math.h"  HAVE_EXP)

if(HAVE_LOG  AND  HAVE_EXP)
  target\_compile\_definitions(MathFunctions
  PRIVATE  "HAVE_LOG"  "HAVE_EXP")
endif()

完成这些更新后，继续并重新构建项目。运行内置的Tutorial可执行文件，并验证结果与本步骤前面的内容相同。

[添加自定义命令和生成的文件（步骤6）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id13)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-a-custom-command-and-generated-file-step-6 "固定到该标题")
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

假设，出于本教程的目的，我们决定我们不想使用平台`log`和`exp`函数，而是想生成一个在`mysqrt`函数中使用的预计算值表。在本节中，我们将在构建过程中创建表，然后将该表编译到我们的应用程序中。

首先，让我们删除中对`log`和`exp`功能 的检查`MathFunctions/CMakeLists.txt`。然后取出支票`HAVE_LOG`和 `HAVE_EXP`从`mysqrt.cxx`。同时，我们可以删除 。`#include <cmath>`

在`MathFunctions`子目录中，提供了一个名为的新源文件 `MakeTable.cxx`来生成表。

查看完文件后，我们可以看到该表是作为有效的C ++代码生成的，并且输出文件名作为参数传入。

下一步是将适当的命令添加到 `MathFunctions/CMakeLists.txt`文件中以构建MakeTable可执行文件，然后在构建过程中运行它。需要一些命令来完成此操作。

首先，在的顶部`MathFunctions/CMakeLists.txt`，`MakeTable`添加的可执行文件， 就像添加任何其他可执行文件一样。

add_executable(MakeTable  MakeTable.cxx)

然后，我们添加一个自定义命令，该命令指定如何`Table.h` 通过运行MakeTable 进行生产。

add\_custom\_command(
  OUTPUT  ${CMAKE\_CURRENT\_BINARY_DIR}/Table.h
  COMMAND  MakeTable  ${CMAKE\_CURRENT\_BINARY_DIR}/Table.h
  DEPENDS  MakeTable
  )

接下来，我们必须让CMake知道这`mysqrt.cxx`取决于生成的文件`Table.h`。这是通过将生成的添加`Table.h`到库MathFunctions的源列表中来完成的。

add_library(MathFunctions
  mysqrt.cxx
  ${CMAKE\_CURRENT\_BINARY_DIR}/Table.h
  )

我们还必须将当前的二进制目录添加到包含目录的列表中，以便`Table.h`可以找到和包含它`mysqrt.cxx`。

target\_include\_directories(MathFunctions
          INTERFACE ${CMAKE\_CURRENT\_SOURCE_DIR}
          PRIVATE ${CMAKE\_CURRENT\_BINARY_DIR}
          )

现在让我们使用生成的表。首先，修改`mysqrt.cxx`为 `Table.h`。接下来，我们可以重写mysqrt函数以使用该表：

double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }

  // use the table to help find an initial value
  double result = x;
  if (x >= 1 && x < 10) {
    std::cout << "Use the table to help find an initial value " << std::endl;
    result = sqrtTable\[static_cast<int>(x)\];
  }

  // do ten iterations
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }

  return result;
}

跑过 [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）") 配置项目，然后使用您选择的构建工具进行构建。

构建此项目时，它将首先构建`MakeTable`可执行文件。然后它将运行`MakeTable`产生`Table.h`。最后，它将进行编译`mysqrt.cxx`，包括`Table.h`生成MathFunctions库。

运行Tutorial可执行文件，并验证它是否正在使用该表。

[构建安装程序（步骤7）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id14)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#building-an-installer-step-7 "固定到该标题")
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

接下来，假设我们要将项目分发给其他人，以便他们可以使用它。我们希望在各种平台上提供二进制和源代码分发。这与我们之前在“ [安装和测试”（第4步）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#installing-and-testing-step-4)中进行的安装有些不同，在[安装和测试中](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#installing-and-testing-step-4)，我们正在安装根据源代码构建的二进制文件。在此示例中，我们将构建支持二进制安装和软件包管理功能的安装软件包。为此，我们将使用CPack创建平台特定的安装程序。具体来说，我们需要在顶级`CMakeLists.txt`文件的底部添加几行。

include(InstallRequiredSystemLibraries)
set(CPACK\_RESOURCE\_FILE_LICENSE  "${CMAKE\_CURRENT\_SOURCE_DIR}/License.txt")
set(CPACK\_PACKAGE\_VERSION_MAJOR  "${Tutorial\_VERSION\_MAJOR}")
set(CPACK\_PACKAGE\_VERSION_MINOR  "${Tutorial\_VERSION\_MINOR}")
include(CPack)

这就是全部。我们首先包括 [`InstallRequiredSystemLibraries`](https://cmake.org/cmake/help/latest/module/InstallRequiredSystemLibraries.html#module:InstallRequiredSystemLibraries "InstallRequiredSystemLibraries")。该模块将包括项目当前平台所需的任何运行时库。接下来，我们将一些CPack变量设置为存储该项目的许可证和版本信息的位置。版本信息是在本教程的前面设置的，并且`license.txt`已包含在此步骤的顶级源目录中。

最后，我们将 [`CPack module`](https://cmake.org/cmake/help/latest/module/CPack.html#module:CPack "包") 它将使用这些变量和当前系统的其他一些属性来设置安装程序。

下一步是按照通常的方式构建项目，然后运行 [`cpack`](https://cmake.org/cmake/help/latest/manual/cpack.1.html#manual:cpack(1) "cpack（1）")可执行文件。要构建二进制发行版，请从二进制目录运行：

cpack

要指定生成器，请使用`-G`选项。对于多配置版本，用于 `-C`指定配置。例如：

cpack -G ZIP -C Debug

要创建源分发，您可以输入：

cpack --config CPackSourceConfig.cmake

或者，从IDE 运行或右键单击目标 。`make package``Package``Build Project`

运行在二进制目录中找到的安装程序。然后运行已安装的可执行文件，并验证其是否有效。

[添加对仪表盘的支持（步骤8）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id15)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-support-for-a-dashboard-step-8 "固定到该标题")
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

添加将测试结果提交到仪表板的支持非常简单。我们已经在“ [测试支持”中](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#testing-support)为我们的项目定义了许多[测试](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#testing-support)。现在，我们只需要运行这些测试并将其提交到仪表板即可。为了包括对仪表板的支持，我们将[`CTest`](https://cmake.org/cmake/help/latest/module/CTest.html#module:CTest "测试")顶层模块 `CMakeLists.txt`。

更换：

\# enable testing
enable_testing()

带有：

\# enable dashboard scripting
include(CTest)

的 [`CTest`](https://cmake.org/cmake/help/latest/module/CTest.html#module:CTest "测试")模块将自动调用`enable_testing()`，因此我们可以将其从CMake文件中删除。

我们还需要`CTestConfig.cmake`在顶级目录中创建一个文件，在其中可以指定项目的名称以及提交仪表板的位置。

set(CTEST\_PROJECT\_NAME  "CMakeTutorial")
set(CTEST\_NIGHTLY\_START_TIME  "00:00:00 EST")

set(CTEST\_DROP\_METHOD  "http")
set(CTEST\_DROP\_SITE  "my.cdash.org")
set(CTEST\_DROP\_LOCATION  "/submit.php?project=CMakeTutorial")
set(CTEST\_DROP\_SITE_CDASH  TRUE)

的 [`ctest`](https://cmake.org/cmake/help/latest/manual/ctest.1.html#manual:ctest(1) "ctest（1）")可执行文件将在运行时读入此文件。要创建一个简单的仪表板，您可以运行[`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）")配置项目，但尚未构建。而是，将目录更改为二进制树，然后运行：

> ctest \[-VV\] -D实验

请记住，对于多配置生成器（例如Visual Studio），必须指定配置类型：

ctest \[-VV\] -C Debug -D Experimental

或者，从IDE中构建`Experimental`目标。

的 [`ctest`](https://cmake.org/cmake/help/latest/manual/ctest.1.html#manual:ctest(1) "ctest（1）")可执行文件将构建并测试项目，并将结果提交到Kitware的公共仪表板：[https](https://my.cdash.org/index.php?project=CMakeTutorial) ://my.cdash.org/index.php?project [=](https://my.cdash.org/index.php?project=CMakeTutorial) CMakeTutorial 。

[混合静态和共享（步骤9）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id16)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#mixing-static-and-shared-step-9 "固定到该标题")
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

在本节中，我们将展示如何 [`BUILD_SHARED_LIBS`](https://cmake.org/cmake/help/latest/variable/BUILD_SHARED_LIBS.html#variable:BUILD_SHARED_LIBS "BUILD_SHARED_LIBS") 变量可用于控制的默认行为 [`add_library()`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "add_library")，并允许在如何没有显式类型库（控制`STATIC`， `SHARED`，`MODULE`或`OBJECT`）构建的。

为此，我们需要添加 [`BUILD_SHARED_LIBS`](https://cmake.org/cmake/help/latest/variable/BUILD_SHARED_LIBS.html#variable:BUILD_SHARED_LIBS "BUILD_SHARED_LIBS")到顶层`CMakeLists.txt`。我们使用[`option()`](https://cmake.org/cmake/help/latest/command/option.html#command:option "选项") 命令，因为它允许用户选择该值是ON还是OFF。

接下来，我们将重构MathFunctions使其成为使用`mysqrt`或封装的真实库`sqrt`，而不是要求调用代码执行此逻辑。这也将意味着`USE_MYMATH`它将不控制构建MathFunction，而将控制该库的行为。

第一步是将顶层的开始部分更新为 `CMakeLists.txt`：

cmake\_minimum\_required(VERSION  3.10)

\# set the project name and version
project(Tutorial  VERSION  1.0)

\# specify the C++ standard
set(CMAKE\_CXX\_STANDARD  11)
set(CMAKE\_CXX\_STANDARD_REQUIRED  True)

\# control where the static and shared libraries are built so that on windows
\# we don't need to tinker with the path to run the executable
set(CMAKE\_ARCHIVE\_OUTPUT_DIRECTORY  "${PROJECT\_BINARY\_DIR}")
set(CMAKE\_LIBRARY\_OUTPUT_DIRECTORY  "${PROJECT\_BINARY\_DIR}")
set(CMAKE\_RUNTIME\_OUTPUT_DIRECTORY  "${PROJECT\_BINARY\_DIR}")

option(BUILD\_SHARED\_LIBS  "Build using shared libraries"  ON)

\# configure a header file to pass the version number only
configure_file(TutorialConfig.h.in  TutorialConfig.h)

\# add the MathFunctions library
add_subdirectory(MathFunctions)

\# add the executable
add_executable(Tutorial  tutorial.cxx)
target\_link\_libraries(Tutorial  PUBLIC  MathFunctions)

既然我们已经使MathFunctions始终被使用，我们将需要更新该库的逻辑。因此，`MathFunctions/CMakeLists.txt`我们需要创建一个SqrtLibrary，`USE_MYMATH`启用后将有条件地对其进行构建。现在，由于这是一个教程，我们将明确要求静态地构建SqrtLibrary。

最终结果`MathFunctions/CMakeLists.txt`应该是：

\# add the library that runs
add_library(MathFunctions  MathFunctions.cxx)

\# state that anybody linking to us needs to include the current source dir
\# to find MathFunctions.h, while we don't.
target\_include\_directories(MathFunctions
  INTERFACE  ${CMAKE\_CURRENT\_SOURCE_DIR}
  )

\# should we use our own math functions
option(USE_MYMATH  "Use tutorial provided math implementation"  ON)
if(USE_MYMATH)

  target\_compile\_definitions(MathFunctions  PRIVATE  "USE_MYMATH")

  \# first we add the executable that generates the table
  add_executable(MakeTable  MakeTable.cxx)

  \# add the command to generate the source code
  add\_custom\_command(
  OUTPUT  ${CMAKE\_CURRENT\_BINARY_DIR}/Table.h
  COMMAND  MakeTable  ${CMAKE\_CURRENT\_BINARY_DIR}/Table.h
  DEPENDS  MakeTable
  )

  \# library that just does sqrt
  add_library(SqrtLibrary  STATIC
  mysqrt.cxx
  ${CMAKE\_CURRENT\_BINARY_DIR}/Table.h
  )

  \# state that we depend on our binary dir to find Table.h
  target\_include\_directories(SqrtLibrary  PRIVATE
  ${CMAKE\_CURRENT\_BINARY_DIR}
  )

  target\_link\_libraries(MathFunctions  PRIVATE  SqrtLibrary)
endif()

\# define the symbol stating we are using the declspec(dllexport) when
\# building on windows
target\_compile\_definitions(MathFunctions  PRIVATE  "EXPORTING_MYMATH")

\# install rules
install(TARGETS  MathFunctions  DESTINATION  lib)
install(FILES  MathFunctions.h  DESTINATION  include)

接下来，更新`MathFunctions/mysqrt.cxx`为使用`mathfunctions`和 `detail`名称空间：

#include <iostream>

#include "MathFunctions.h"

// include the generated table
#include "Table.h"

namespace mathfunctions {
namespace detail {
// a hack square root calculation using simple operations
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }

  // use the table to help find an initial value
  double result = x;
  if (x >= 1 && x < 10) {
    std::cout << "Use the table to help find an initial value " << std::endl;
    result = sqrtTable\[static_cast<int>(x)\];
  }

  // do ten iterations
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }

  return result;
}
}
}

我们还需要对中进行一些更改`tutorial.cxx`，以使其不再使用`USE_MYMATH`：

1.  一律包括 `MathFunctions.h`
    
2.  一律使用 `mathfunctions::sqrt`
    
3.  不包括cmath
    

最后，更新`MathFunctions/MathFunctions.h`为使用dll导出定义：

#if defined(_WIN32)
\#  if defined(EXPORTING_MYMATH)
\#    define DECLSPEC __declspec(dllexport)
\#  else
\#    define DECLSPEC __declspec(dllimport)
\#  endif
#else // non windows
\#  define DECLSPEC
#endif

namespace mathfunctions {
double DECLSPEC sqrt(double x);
}

此时，如果您构建了所有内容，则会注意到链接失败，因为我们将没有位置独立代码的静态库与具有位置独立代码的库组合在一起。解决方案是明确设置[`POSITION_INDEPENDENT_CODE`](https://cmake.org/cmake/help/latest/prop_tgt/POSITION_INDEPENDENT_CODE.html#prop_tgt:POSITION_INDEPENDENT_CODE "POSITION_INDEPENDENT_CODE") 无论构建类型如何，SqrtLibrary的target属性都应为True。

  \# state that SqrtLibrary need PIC when the default is shared libraries
  set\_target\_properties(SqrtLibrary  PROPERTIES
  POSITION\_INDEPENDENT\_CODE  ${BUILD\_SHARED\_LIBS}
  )

  target\_link\_libraries(MathFunctions  PRIVATE  SqrtLibrary)

**练习**：我们修改`MathFunctions.h`为使用dll导出定义。使用CMake文档，您可以找到一个帮助器模块来简化此过程吗？

[添加生成器表达式（步骤10）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id17)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-generator-expressions-step-10 "固定到该标题")
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[`Generator expressions`](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions（7）") 在生成系统的过程中进行评估，以生成特定于每个生成配置的信息。

[`Generator expressions`](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions（7）") 在许多目标属性的上下文中是允许的，例如 [`LINK_LIBRARIES`](https://cmake.org/cmake/help/latest/prop_tgt/LINK_LIBRARIES.html#prop_tgt:LINK_LIBRARIES "LINK_LIBRARIES")， [`INCLUDE_DIRECTORIES`](https://cmake.org/cmake/help/latest/prop_tgt/INCLUDE_DIRECTORIES.html#prop_tgt:INCLUDE_DIRECTORIES "INCLUDE_DIRECTORIES")， [`COMPILE_DEFINITIONS`](https://cmake.org/cmake/help/latest/prop_tgt/COMPILE_DEFINITIONS.html#prop_tgt:COMPILE_DEFINITIONS "COMPILE_DEFINITIONS")和别的。在使用命令填充这些属性时，也可以使用它们，例如 [`target_link_libraries()`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries")， [`target_include_directories()`](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories "target_include_directories")， [`target_compile_definitions()`](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions "target_compile_definitions") 和别的。

[`Generator expressions`](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions（7）") 可能用于启用条件链接，编译时使用的条件定义，条件包含目录等。条件可以基于构建配置，目标属性，平台信息或任何其他可查询信息。

有不同类型的 [`generator expressions`](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions（7）") 包括逻辑，信息和输出表达式。

逻辑表达式用于创建条件输出。基本表达式是0和1表达式。一个`$<0:...>`导致空字符串，而`<1:...>`导致的内容“...”。它们也可以嵌套。

的常见用法 [`generator expressions`](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions（7）")是有条件地添加编译器标志，例如用于语言级别或警告的标志。一个不错的模式是将该信息与`INTERFACE` 目标关联，以允许该信息传播。让我们从构造一个 `INTERFACE`目标并指定所需的C ++标准级别开始，`11` 而不是使用[`CMAKE_CXX_STANDARD`](https://cmake.org/cmake/help/latest/variable/CMAKE_CXX_STANDARD.html#variable:CMAKE_CXX_STANDARD "CMAKE_CXX_STANDARD")。

所以下面的代码：

\# specify the C++ standard
set(CMAKE\_CXX\_STANDARD  11)
set(CMAKE\_CXX\_STANDARD_REQUIRED  True)

将被替换为：

add_library(tutorial\_compiler\_flags  INTERFACE)
target\_compile\_features(tutorial\_compiler\_flags  INTERFACE  cxx\_std\_11)

接下来，我们为项目添加所需的编译器警告标志。由于警告标志根据编译器的不同而不同，因此我们使用`COMPILE_LANG_AND_ID` 生成器表达式来控制在给定一种语言和一组编译器ID的情况下应用哪些标志，如下所示：

set(gcc\_like\_cxx  "$<COMPILE\_LANG\_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU>")
set(msvc_cxx  "$<COMPILE\_LANG\_AND_ID:CXX,MSVC>")
target\_compile\_options(tutorial\_compiler\_flags  INTERFACE
  "$<${gcc\_like\_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc\_cxx}:$<BUILD\_INTERFACE:-W3>>"
)

看着这个，我们看到警告标志被封装在一个 `BUILD_INTERFACE`条件中。这样做是为了使我们已安装项目的使用者不会继承我们的警告标志。

**练习**：修改`MathFunctions/CMakeLists.txt`以使所有目标都有一个[`target_link_libraries()`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries")致电至`tutorial_compiler_flags`。

[添加导出配置（步骤11）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id18)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-export-configuration-step-11 "固定到该标题")
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

在本教程的“ [安装和测试”（第4步）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#installing-and-testing-step-4)中，我们添加了CMake的功能，以安装项目的库和头文件。在 [构建安装程序（第7步）期间，](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#building-an-installer-step-7)我们添加了打包此信息的功能，以便可以将其分发给其他人。

下一步是添加必要的信息，以便其他CMake项目可以使用我们的项目，无论是从构建目录，本地安装还是打包时。

第一步是更新我们的 [`install(TARGETS)`](https://cmake.org/cmake/help/latest/command/install.html#command:install "安装")命令不仅指定a `DESTINATION`而且还指定`EXPORT`。该`EXPORT`关键字生成并安装包含代码导入从安装树安装命令列出的所有目标的CMake的文件。因此，让我们继续进行操作，并`EXPORT`通过更新以下`install` 命令`MathFunctions/CMakeLists.txt`来显式地显示MathFunctions库：

install(TARGETS  MathFunctions  tutorial\_compiler\_flags
  DESTINATION  lib
  EXPORT  MathFunctionsTargets)
install(FILES  MathFunctions.h  DESTINATION  include)

现在我们已经导出了MathFunction，我们还需要显式安装生成的`MathFunctionsTargets.cmake`文件。这是通过在顶层底部添加以下内容来完成的`CMakeLists.txt`：

install(EXPORT  MathFunctionsTargets
  FILE  MathFunctionsTargets.cmake
  DESTINATION  lib/cmake/MathFunctions
)

此时，您应该尝试运行CMake。如果一切设置正确，您将看到CMake将生成如下错误：

Target "MathFunctions" INTERFACE\_INCLUDE\_DIRECTORIES property contains
path:

 "/Users/robert/Documents/CMakeClass/Tutorial/Step11/MathFunctions"

which is prefixed in the source directory.

CMake试图说的是，在生成导出信息的过程中，它将导出与当前机器固有联系的路径，并且在其他机器上无效。解决方案是更新MathFunctions[`target_include_directories()`](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories "target_include_directories")了解`INTERFACE`在构建目录和安装/软件包中使用它时需要在不同的位置。这意味着转换 [`target_include_directories()`](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories "target_include_directories") 要求MathFunctions看起来像：

target\_include\_directories(MathFunctions
  INTERFACE
  $<BUILD_INTERFACE:${CMAKE\_CURRENT\_SOURCE_DIR}>
  $<INSTALL_INTERFACE:include>
  )

更新后，我们可以重新运行CMake并确认它不再发出警告。

在这一点上，我们已经正确地包装了CMake所需的目标信息，但是我们仍然需要生成一个`MathFunctionsConfig.cmake`使CMake[`find_package()`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package "find_package")命令可以找到我们的项目。因此，我们继续将新文件添加到项目的顶层，该文件 `Config.cmake.in`具有以下内容：

@PACKAGE_INIT@

include ( "${CMAKE\_CURRENT\_LIST_DIR}/MathFunctionsTargets.cmake" )

然后，要正确配置和安装该文件，请将以下内容添加到顶层的底部`CMakeLists.txt`：

install(EXPORT  MathFunctionsTargets
  FILE  MathFunctionsTargets.cmake
  DESTINATION  lib/cmake/MathFunctions
)

include(CMakePackageConfigHelpers)
\# generate the config file that is includes the exports
configure\_package\_config_file(${CMAKE\_CURRENT\_SOURCE_DIR}/Config.cmake.in
  "${CMAKE\_CURRENT\_BINARY_DIR}/MathFunctionsConfig.cmake"
  INSTALL_DESTINATION  "lib/cmake/example"
  NO\_SET\_AND\_CHECK\_MACRO
  NO\_CHECK\_REQUIRED\_COMPONENTS\_MACRO
  )
\# generate the version file for the config file
write\_basic\_package\_version\_file(
  "${CMAKE\_CURRENT\_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION  "${Tutorial\_VERSION\_MAJOR}.${Tutorial\_VERSION\_MINOR}"
  COMPATIBILITY  AnyNewerVersion
)

\# install the configuration file
install(FILES
  ${CMAKE\_CURRENT\_BINARY_DIR}/MathFunctionsConfig.cmake
  DESTINATION  lib/cmake/MathFunctions
  )

至此，我们为项目生成了可重定位的CMake配置，可在安装或打包项目后使用它。如果我们也希望从构建目录中使用我们的项目，则只需将以下内容添加到顶层的底部`CMakeLists.txt`：

export(EXPORT  MathFunctionsTargets
  FILE  "${CMAKE\_CURRENT\_BINARY_DIR}/MathFunctionsTargets.cmake"
)

通过此导出调用，我们现在生成一个`Targets.cmake`，允许`MathFunctionsConfig.cmake`构建目录中的配置供其他项目使用，而无需安装它。

[打包调试和发布（步骤12）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id19)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#packaging-debug-and-release-step-12 "固定到该标题")
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**注意：**此示例对单配置生成器有效，而对多配置生成器（例如Visual Studio）无效。

默认情况下，CMake的模型是构建目录仅包含一个配置，可以是Debug，Release，MinSizeRel或RelWithDebInfo。但是，可以将CPack设置为捆绑多个构建目录，并构建一个包含同一项目的多个配置的软件包。

首先，我们要确保调试版本和发行版本对要安装的可执行文件和库使用不同的名称。让我们使用d作为调试可执行文件和库的后缀。

组 [`CMAKE_DEBUG_POSTFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_DEBUG_POSTFIX.html#variable:CMAKE_DEBUG_POSTFIX "CMAKE_DEBUG_POSTFIX")在顶级`CMakeLists.txt`文件开头附近 ：

set(CMAKE\_DEBUG\_POSTFIX  d)

add_library(tutorial\_compiler\_flags  INTERFACE)

和 [`DEBUG_POSTFIX`](https://cmake.org/cmake/help/latest/prop_tgt/DEBUG_POSTFIX.html#prop_tgt:DEBUG_POSTFIX "DEBUG_POSTFIX") 教程可执行文件上的属性：

add_executable(Tutorial  tutorial.cxx)
set\_target\_properties(Tutorial  PROPERTIES  DEBUG_POSTFIX  ${CMAKE\_DEBUG\_POSTFIX})

target\_link\_libraries(Tutorial  PUBLIC  MathFunctions)

让我们还将版本编号添加到MathFunctions库中。在中 `MathFunctions/CMakeLists.txt`，设置[`VERSION`](https://cmake.org/cmake/help/latest/prop_tgt/VERSION.html#prop_tgt:VERSION "版") 和 [`SOVERSION`](https://cmake.org/cmake/help/latest/prop_tgt/SOVERSION.html#prop_tgt:SOVERSION "覆盖") 特性：

set_property(TARGET  MathFunctions  PROPERTY  VERSION  "1.0.0")
set_property(TARGET  MathFunctions  PROPERTY  SOVERSION  "1")

在`Step12`目录中，创建`debug`和`release` 子目录。布局将如下所示：

\- Step12
   └── debug
   └── release

现在我们需要设置调试和发布版本。我们可以用 [`CMAKE_BUILD_TYPE`](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html#variable:CMAKE_BUILD_TYPE "CMAKE_BUILD_TYPE") 设置配置类型：

cd debug
cmake -DCMAKE\_BUILD\_TYPE=Debug ..
cmake --build .
cd ../release
cmake -DCMAKE\_BUILD\_TYPE=Release ..
cmake --build .

既然调试和发布版本均已完成，我们就可以使用自定义配置文件将两个版本打包到一个版本中。在 `Step12`目录中，创建一个名为的文件`MultiCPackConfig.cmake`。在此文件中，首先包括由配置文件创建的默认配置文件。 [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") 可执行文件。

接下来，使用`CPACK_INSTALL_CMAKE_PROJECTS`变量指定要安装的项目。在这种情况下，我们要同时安装调试和发布。

include("release/CPackConfig.cmake")

set(CPACK\_INSTALL\_CMAKE_PROJECTS
  "debug;Tutorial;ALL;/"
  "release;Tutorial;ALL;/"
  )

从`Step12`目录运行[`cpack`](https://cmake.org/cmake/help/latest/manual/cpack.1.html#manual:cpack(1) "cpack（1）")使用以下`config`选项指定我们的自定义配置文件：

cpack --config MultiCPackConfig.cmake

### [目录](https://cmake.org/cmake/help/latest/index.html)

*   [CMake教程](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#)
    *   [基本起点（步骤1）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#a-basic-starting-point-step-1)
        *   [添加版本号和配置的头文件](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-a-version-number-and-configured-header-file)
        *   [指定C ++标准](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#specify-the-c-standard)
        *   [构建和测试](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#build-and-test)
    *   [添加库（步骤2）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-a-library-step-2)
    *   [添加库的使用要求（步骤3）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-usage-requirements-for-library-step-3)
    *   [安装和测试（步骤4）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#installing-and-testing-step-4)
        *   [安装规则](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#install-rules)
        *   [测试支持](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#testing-support)
    *   [添加系统自检（步骤5）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-system-introspection-step-5)
        *   [指定编译定义](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#specify-compile-definition)
    *   [添加自定义命令和生成的文件（步骤6）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-a-custom-command-and-generated-file-step-6)
    *   [生成安装程序（步骤7）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#building-an-installer-step-7)
    *   [添加对仪表板的支持（步骤8）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-support-for-a-dashboard-step-8)
    *   [混合静态和共享（步骤9）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#mixing-static-and-shared-step-9)
    *   [添加生成器表达式（步骤10）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-generator-expressions-step-10)
    *   [添加导出配置（步骤11）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-export-configuration-step-11)
    *   [打包调试和发布（步骤12）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#packaging-debug-and-release-step-12)

#### 上一主题

[CPack WIX生成器](https://cmake.org/cmake/help/latest/cpack_gen/wix.html "前一章")

#### 下一个话题

[用户互动指南](https://cmake.org/cmake/help/latest/guide/user-interaction/index.html "下一章")

### 这一页

*   [显示来源](https://cmake.org/cmake/help/latest/_sources/guide/tutorial/index.rst.txt)

### 快速搜索

 

$('#searchbox').show(0);

### 导航

*   [指数](https://cmake.org/cmake/help/latest/genindex.html "综合指数")
*   [下一个](https://cmake.org/cmake/help/latest/guide/user-interaction/index.html "用户互动指南") |
*   [上一个](https://cmake.org/cmake/help/latest/cpack_gen/wix.html "CPack WIX生成器") |
*   ![](./CMake教程— CMake 3.17.0-rc1文档_files/cmake-logo-16.png)
*   [CMake](https://cmake.org/) »
*   git-stagegit-master最新版本（3.17.0-rc1）3.173.163.153.143.133.123.113.103.93.83.73.63.53.43.33.23.13.0 [说明文件](https://cmake.org/cmake/help/latest/index.html) »

©版权所有2000-2020 Kitware，Inc.和贡献者。使用[Sphinx](http://sphinx-doc.org/) 2.3.1 创建。

var gaJsHost = (("https:" == document.location.protocol) ? "https://ssl." : "http://www."); document.write(unescape("%3Cscript src='" + gaJsHost + "google-analytics.com/ga.js' type='text/javascript'%3E%3C/script%3E")); try { var pageTracker = \_gat.\_getTracker("UA-6042509-4"); pageTracker._trackPageview(); } catch(err) {}

![Google 翻译](./CMake教程— CMake 3.17.0-rc1文档_files/translate_24dp.png)

原文
==

提供更好的翻译建议

* * *