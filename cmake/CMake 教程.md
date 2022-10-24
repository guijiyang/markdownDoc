
CMake 教程提供了逐步指南，涵盖了 CMake 可以解决的常见构建系统问题。了解示例项目中各个主题如何协同工作将非常有帮助。教程文档和示例的源代码可以`Help/guide/tutorial`在 CMake 源代码树的目录中找到 。每个步骤都有其自己的子目录，其中包含可以用作起点的代码。教程示例是渐进式的，因此每个步骤都为上一步提供了完整的解决方案。

[基本起点（步骤 1）](#id2)[¶](#a-basic-starting-point-step-1 "固定到该标题")
--------------------------------------------------------------

最基本的项目是从源代码文件构建的可执行文件。对于简单项目，只需要三行`CMakeLists.txt`文件。这将是本教程的起点。`CMakeLists.txt`在`Step1`目录中创建一个 文件，如下所示：

```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name
project(Tutorial)

# add the executable
add_executable(Tutorial tutorial.cxx)


```

请注意，此示例在`CMakeLists.txt`文件中使用小写命令。CMake 支持大写，小写和大小写混合命令。目录中`tutorial.cxx`提供的源代码，`Step1`可用于计算数字的平方根。

### [添加一个版本号和配置的头文件](#id3)  [¶](#adding-a-version-number-and-configured-header-file "固定到该标题")

我们将添加的第一个功能是为我们的可执行文件和项目提供版本号。虽然我们可以在源代码中专门执行此操作，但是使用 `CMakeLists.txt`可以提供更大的灵活性。

首先，修改`CMakeLists.txt`文件以使用 [`project()`](https://cmake.org/cmake/help/latest/command/project.html#command:project "项目") 命令设置项目名称和版本号。

```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)


```

然后，配置头文件以将版本号传递给源代码：

```cmake
configure_file(TutorialConfig.h.in TutorialConfig.h)


```

由于配置的文件将被写入二进制树，因此我们必须将该目录添加到路径列表中以搜索包含文件。将以下行添加到`CMakeLists.txt`文件的末尾：

```cmake
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```

使用您喜欢的编辑器，`TutorialConfig.h.in`在源目录中创建以下内容：

```cmake
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

当 CMake 配置此头文件时，`@Tutorial_VERSION_MAJOR@`和的值 `@Tutorial_VERSION_MINOR@`将被替换。

接下来修改`tutorial.cxx`以包括配置的头文件 `TutorialConfig.h`。

最后，通过`tutorial.cxx`如下更新来打印出版本号：

```cmake
  if (argc < 2) {
    // report version
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
```

### [指定 C ++ 标准](#id4)  [¶](#specify-the-c-standard "固定到该标题")

接下来，我们`atof`用 `std::stod`in 替换为我们的项目添加一些 C ++ 11 功能`tutorial.cxx`。同时，删除 。`#include <cstdlib>`

```c++
  const double inputValue = std::stod(argv[1]);


```

我们将需要在 CMake 代码中明确声明应使用正确的标志。在 CMake 中启用对特定 C ++ 标准的支持的最简单方法是使用 [`CMAKE_CXX_STANDARD`](https://cmake.org/cmake/help/latest/variable/CMAKE_CXX_STANDARD.html#variable:CMAKE_CXX_STANDARD "CMAKE_CXX_STANDARD")变量。对于本教程，请设置 [`CMAKE_CXX_STANDARD`](https://cmake.org/cmake/help/latest/variable/CMAKE_CXX_STANDARD.html#variable:CMAKE_CXX_STANDARD "CMAKE_CXX_STANDARD")将`CMakeLists.txt`文件中的变量设置 为 11 并 [`CMAKE_CXX_STANDARD_REQUIRED`](https://cmake.org/cmake/help/latest/variable/CMAKE_CXX_STANDARD_REQUIRED.html#variable:CMAKE_CXX_STANDARD_REQUIRED "CMAKE_CXX_STANDARD_REQUIRED") 改为 True：

```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)


```

### [构建和测试](#id5)  [¶](#build-and-test "固定到该标题")

跑过 [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）") 配置项目，然后使用您选择的构建工具进行构建。

例如，从命令行我们可以导航到`Help/guide/tutorial`CMake 源代码树的 目录并运行以下命令：

```shell
mkdir Step1_build
cd Step1_build
cmake ../Step1
cmake --build .
```

导航到构建 Tutorial 的目录（可能是 make 目录或 Debug 或 Release 构建配置子目录），然后运行以下命令：

```shell
Tutorial 4294967296
Tutorial 10
Tutorial


```

[添加一个库（第二步）](#id6)[¶](#adding-a-library-step-2 "固定到该标题")
--------------------------------------------------------

现在，我们将库添加到我们的项目中。该库将包含我们自己的实现，用于计算数字的平方根。然后可执行文件可以使用此库而不是编译器提供的标准平方根函数。

在本教程中，我们将库放入名为的子目录中`MathFunctions`。该目录已经包含一个头文件 `MathFunctions.h`和一个源文件`mysqrt.cxx`。源文件具有一个称为的`mysqrt`功能，该功能提供与编译器`sqrt`功能相似的功能。

将以下一个行`CMakeLists.txt`文件添加到`MathFunctions` 目录：

```cmake
add_library(MathFunctions mysqrt.cxx)
```

为了利用新库，我们将添加一个 [`add_subdirectory()`](https://cmake.org/cmake/help/latest/command/add_subdirectory.html#command:add_subdirectory "add_sub目录") 调用顶级`CMakeLists.txt`文件，以便构建库。我们将新库添加到可执行文件，并添加`MathFunctions`为包含目录，以便`mqsqrt.h`可以找到头文件。现在，顶级`CMakeLists.txt`文件的最后几行应如下所示：

```cmake
# add the MathFunctions library
add_subdirectory(MathFunctions)

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC MathFunctions)

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                          "${PROJECT_BINARY_DIR}"
                          "${PROJECT_SOURCE_DIR}/MathFunctions"
                          )
```

现在让我们将 MathFunctions 库设为可选。虽然对于本教程而言确实没有任何必要，但是对于较大的项目，这是常见的情况。第一步是向顶层`CMakeLists.txt`文件添加一个选项 。

```cmake
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

此选项将显示在 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）") 和 [`ccmake`](https://cmake.org/cmake/help/latest/manual/ccmake.1.html#manual:ccmake(1) "ccmake（1）") 用户可以更改的默认值 ON。此设置将存储在缓存中，因此用户无需在每次在构建目录上运行 CMake 时都设置该值。

下一个更改是使建立和链接 MathFunctions 库成为条件。为此，我们将顶级`CMakeLists.txt` 文件的末尾更改为如下所示：

```cmake
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
  list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           ${EXTRA_INCLUDES}
                           )


```

请注意，使用变量`EXTRA_LIBS`来收集所有可选库，以便以后链接到可执行文件中。该变量 `EXTRA_INCLUDES`类似地用于可选的头文件。在处理许多可选组件时，这是一种经典方法，我们将在下一步中介绍现代方法。

对源代码的相应更改非常简单。首先，如果需要，请在`tutorial.cxx`中包含`MathFunctions.h`标头：

```c++
#ifdef USE_MYMATH
#  include "MathFunctions.h"
#endif


```

然后，在同一文件中，`USE_MYMATH`控制使用哪个平方根函数：

```c++
#ifdef USE_MYMATH
  const double outputValue = mysqrt(inputValue);
#else
  const double outputValue = sqrt(inputValue);
#endif


```

由于源代码现在需要，`USE_MYMATH`我们可以`TutorialConfig.h.in`使用以下行将其添加到 其中：

**练习**：为什么`TutorialConfig.h.in` 在`USE_MYMATH`选项之后进行配置很重要？如果我们将两者倒置会怎样？

跑过 [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）")配置项目，然后使用您选择的构建工具进行构建。然后运行构建的 Tutorial 可执行文件。

使用 [`ccmake`](https://cmake.org/cmake/help/latest/manual/ccmake.1.html#manual:ccmake(1) "ccmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）") 更新的值`USE_MYMATH`。重新生成并再次运行本教程。sqrt 或 mysqrt 哪个函数可提供更好的结果？

[添加库的使用要求（步骤 3）](#id7)[¶](#adding-usage-requirements-for-library-step-3 "固定到该标题")
---------------------------------------------------------------------------------

使用要求可以更好地控制库或可执行文件的链接并包含行，同时还可以更好地控制 CMake 内部目标的传递属性。利用使用需求的主要命令是：

- target_compile_definitions()

- target_compile_options()

- target_include_directories()

- target_link_libraries()

让我们从[添加库（第 2 步）中](#adding-a-library-step-2)重构代码，以利用使用需求的现代 CMake 方法。我们首先声明，链接到 MathFunctions 的任何人都需要包括当前源目录，而 MathFunctions 本身不需要。因此这能成为`INTERFACE`使用要求。

记住`INTERFACE`是指消费者需要的东西，而生产者则不需要。将以下行添加到`MathFunctions/CMakeLists.txt`的末尾：

```cmake
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          )


```

既然我们已经指定了 MathFunction 的使用要求，我们就可以安全地`EXTRA_INCLUDES`从顶级`CMakeLists.txt`（这里）删除对变量 的使用：

```cmake
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif()


```

和这里：

```cmake
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )


```

完成后，运行 [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）")配置项目，然后使用您选择的构建工具或通过构建目录进行构建。`cmake --build .`

[安装和测试（步骤 4）](#id8)[¶](#installing-and-testing-step-4 "固定到该标题")
---------------------------------------------------------------

现在，我们可以开始为项目添加安装规则和测试支持。

### [安装规则](#id9)  [¶](#install-rules "固定到该标题")

安装规则非常简单：对于 MathFunctions，我们要安装库和头文件，对于应用程序，我们要安装可执行文件和已配置的头文件。

因此，`MathFunctions/CMakeLists.txt`我们最后添加：

```cmake
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)


```

并在顶层末尾`CMakeLists.txt`添加：

```cmake
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )


```

这是创建本教程的基本本地安装所需的全部。

跑过 [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") 可执行文件或 [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）")配置项目，然后使用您选择的构建工具进行构建。使用以下`install` 选项运行安装步骤 [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）")命令（在 3.15 中引入，较早版本的 CMake 必须使用`make install`），或者从 IDE 构建`INSTALL`目标。这将安装适当的头文件，库和可执行文件。

CMake 变量 [`CMAKE_INSTALL_PREFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html#variable:CMAKE_INSTALL_PREFIX "CMAKE_INSTALL_PREFIX")用于确定文件的安装根目录。如果使用`cmake --install`定制安装目录，则可以通过参数`--prefix`指定。对于多配置工具，请使用参数`--config`指定配置。

验证已安装的教程正在运行。

### [测试支持](#id10)  [¶](#testing-support "固定到该标题")

接下来让我们测试我们的应用程序。在顶级`CMakeLists.txt` 文件的末尾，我们可以启用测试，然后添加一些基本测试以验证应用程序是否正常运行。

```cmake
enable_testing()

# does the application run
add_test(NAME Runs COMMAND Tutorial 25)

# does the usage message work?
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )

# define a function to simplify adding tests
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction(do_test)

# do a bunch of result based tests
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")


```

第一个测试只是验证应用程序正在运行，没有段错误或其他崩溃，并且返回值为零。这是 CTest 测试的基本形式。

下一个测试利用 [`PASS_REGULAR_EXPRESSION`](https://cmake.org/cmake/help/latest/prop_test/PASS_REGULAR_EXPRESSION.html#prop_test:PASS_REGULAR_EXPRESSION "PASS_REGULAR_EXPRESSION")测试属性，以验证测试的输出包含某些字符串。在这种情况下，请验证在提供了错误数量的参数时是否打印了用法消息。

最后，我们有一个名为`do_test`的函数，该函数运行应用程序并验证所计算的平方根对于给定输入是否正确。 每次调用`do_test`时，都会根据传递的参数将另一个测试添加到项目中，并带有名称，输入和预期结果。

重建应用程序，然后`cd`到二进制目录并运行 [`ctest`](https://cmake.org/cmake/help/latest/manual/ctest.1.html#manual:ctest(1) "ctest（1）")可执行文件：`ctest -N`和`ctest -VV`。对于多配置生成器（例如Visual Studio），必须指定配置类型。例如，要在“调试”模式下运行测试，请在 build目录使用`ctest -C Debug -VV`（而非Debug子目录！）。或者，从IDE 构建目标`RUN_TESTS`。

[添加系统自检（步骤5）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#id11)[¶](https://cmake.org/cmake/help/latest/guide/tutorial/index.html#adding-system-introspection-step-5 "固定到该标题")
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

让我们考虑将一些代码添加到我们的项目中，这依赖于目标平台可能不具备的函数。对于此示例，我们将添加一些代码，这些代码取决于目标平台是否具有`log`和`exp` 函数。当然，几乎每个平台都具有这些功能，但对于本教程而言，它们并不常见。

如果平台具有`log`和`exp`函数,则我们将使用它们来计算函数中的平方根`mysqrt`。我们首先使用 `CMakeLists.txt`顶层模块[`CheckSymbolExists`](https://cmake.org/cmake/help/latest/module/CheckSymbolExists.html#module:CheckSymbolExists "CheckSymbolExists")测试这些函数的可用性。我们将在`TutorialConfig.h.in`中使用新的定义 ，因此请确保在配置该文件之前进行设置.

```cmake
include(CheckSymbolExists)
set(CMAKE_REQUIRED_LIBRARIES "m")
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)
```

Now let’s add these defines to `TutorialConfig.h.in` so that we can use them from `mysqrt.cxx`:

```c++
// does the platform provide exp and log functions?
#cmakedefine HAVE_LOG
#cmakedefine HAVE_EXP
```


If `log` and `exp` are available on the system, then we will use them to compute the square root in the `mysqrt` function. Add the following code to the `mysqrt` function in `MathFunctions/mysqrt.cxx` (don’t forget the `#endif` before returning the result!):

```c++
#if defined(HAVE_LOG) && defined(HAVE_EXP)
  double result = exp(log(x) * 0.5);
  std::cout << "Computing sqrt of " << x << " to be " << result
            << " using log and exp" << std::endl;
#else
  double result = x;
```

We will also need to modify `mysqrt.cxx` to include `cmath`.

Run the [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") executable or the [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）") to configure the project and then build it with your chosen build tool and run the Tutorial executable.

You will notice that we’re not using `log` and `exp`, even if we think they should be available. We should realize quickly that we have forgotten to include `TutorialConfig.h` in `mysqrt.cxx`.

We will also need to update `MathFunctions/CMakeLists.txt` so `mysqrt.cxx` knows where this file is located:

```cmake
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          PRIVATE ${CMAKE_BINARY_DIR}
          )


```

After making this update, go ahead and build the project again and run the built Tutorial executable. If `log` and `exp` are still not being used, open the generated `TutorialConfig.h` file from the build directory. Maybe they aren’t available on the current system?

Which function gives better results now, sqrt or mysqrt?

### [Specify Compile Definition](#id12)[¶](#specify-compile-definition "固定到该标题")

Is there a better place for us to save the `HAVE_LOG` and `HAVE_EXP` values other than in `TutorialConfig.h`? Let’s try to use [`target_compile_definitions()`](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions "target_compile_definitions").

First, remove the defines from `TutorialConfig.h.in`. We no longer need to include `TutorialConfig.h` from `mysqrt.cxx` or the extra include in `MathFunctions/CMakeLists.txt`.

Next, we can move the check for `HAVE_LOG` and `HAVE_EXP` to `MathFunctions/CMakeLists.txt` and then specify those values as `PRIVATE` compile definitions.

```cmake
include(CheckSymbolExists)
set(CMAKE_REQUIRED_LIBRARIES "m")
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)

if(HAVE_LOG AND HAVE_EXP)
  target_compile_definitions(MathFunctions
                             PRIVATE "HAVE_LOG" "HAVE_EXP")
endif()


```

After making these updates, go ahead and build the project again. Run the built Tutorial executable and verify that the results are same as earlier in this step.

[Adding a Custom Command and Generated File (Step 6)](#id13)[¶](#adding-a-custom-command-and-generated-file-step-6 "固定到该标题")
----------------------------------------------------------------------------------------------------------------------------

Suppose, for the purpose of this tutorial, we decide that we never want to use the platform `log` and `exp` functions and instead would like to generate a table of precomputed values to use in the `mysqrt` function. In this section, we will create the table as part of the build process, and then compile that table into our application.

First, let’s remove the check for the `log` and `exp` functions in `MathFunctions/CMakeLists.txt`. Then remove the check for `HAVE_LOG` and `HAVE_EXP` from `mysqrt.cxx`. At the same time, we can remove `#include <cmath>`.

In the `MathFunctions` subdirectory, a new source file named `MakeTable.cxx` has been provided to generate the table.

After reviewing the file, we can see that the table is produced as valid C++ code and that the output filename is passed in as an argument.

The next step is to add the appropriate commands to the `MathFunctions/CMakeLists.txt` file to build the MakeTable executable and then run it as part of the build process. A few commands are needed to accomplish this.

First, at the top of `MathFunctions/CMakeLists.txt`, the executable for `MakeTable` is added as any other executable would be added.

```cmake
add_executable(MakeTable MakeTable.cxx)
```

Then we add a custom command that specifies how to produce `Table.h` by running MakeTable.

```cmake
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  )


```

Next we have to let CMake know that `mysqrt.cxx` depends on the generated file `Table.h`. This is done by adding the generated `Table.h` to the list of sources for the library MathFunctions.

```cmake
add_library(MathFunctions
            mysqrt.cxx
            ${CMAKE_CURRENT_BINARY_DIR}/Table.h
            )


```

We also have to add the current binary directory to the list of include directories so that `Table.h` can be found and included by `mysqrt.cxx`.

```cmake
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          PRIVATE ${CMAKE_CURRENT_BINARY_DIR}
          )



```

Now let’s use the generated table. First, modify `mysqrt.cxx` to include `Table.h`. Next, we can rewrite the mysqrt function to use the table:

```c++
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }

  // use the table to help find an initial value
  double result = x;
  if (x >= 1 && x < 10) {
    std::cout << "Use the table to help find an initial value " << std::endl;
    result = sqrtTable[static_cast<int>(x)];
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


```

Run the [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") executable or the [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）") to configure the project and then build it with your chosen build tool.

When this project is built it will first build the `MakeTable` executable. It will then run `MakeTable` to produce `Table.h`. Finally, it will compile `mysqrt.cxx` which includes `Table.h` to produce the MathFunctions library.

Run the Tutorial executable and verify that it is using the table.

[Building an Installer (Step 7)](#id14)[¶](#building-an-installer-step-7 "固定到该标题")
----------------------------------------------------------------------------------

Next suppose that we want to distribute our project to other people so that they can use it. We want to provide both binary and source distributions on a variety of platforms. This is a little different from the install we did previously in [Installing and Testing (Step 4)](#installing-and-testing-step-4) , where we were installing the binaries that we had built from the source code. In this example we will be building installation packages that support binary installations and package management features. To accomplish this we will use CPack to create platform specific installers. Specifically we need to add a few lines to the bottom of our top-level `CMakeLists.txt` file.

```cmake
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include(CPack)


```

That is all there is to it. We start by including [`InstallRequiredSystemLibraries`](https://cmake.org/cmake/help/latest/module/InstallRequiredSystemLibraries.html#module:InstallRequiredSystemLibraries "InstallRequiredSystemLibraries"). This module will include any runtime libraries that are needed by the project for the current platform. Next we set some CPack variables to where we have stored the license and version information for this project. The version information was set earlier in this tutorial and the `license.txt` has been included in the top-level source directory for this step.

Finally we include the [`CPack module`](https://cmake.org/cmake/help/latest/module/CPack.html#module:CPack "包") which will use these variables and some other properties of the current system to setup an installer.

The next step is to build the project in the usual manner and then run the [`cpack`](https://cmake.org/cmake/help/latest/manual/cpack.1.html#manual:cpack(1) "cpack（1）") executable. To build a binary distribution, from the binary directory run:

To specify the generator, use the `-G` option. For multi-config builds, use `-C` to specify the configuration. For example:

To create a source distribution you would type:

```shell
cpack --config CPackSourceConfig.cmake
```

Alternatively, run `make package` or right click the `Package` target and `Build Project` from an IDE.

Run the installer found in the binary directory. Then run the installed executable and verify that it works.

[Adding Support for a Dashboard (Step 8)](#id15)[¶](#adding-support-for-a-dashboard-step-8 "固定到该标题")
----------------------------------------------------------------------------------------------------

Adding support for submitting our test results to a dashboard is simple. We already defined a number of tests for our project in [Testing Support](#testing-support). Now we just have to run those tests and submit them to a dashboard. To include support for dashboards we include the [`CTest`](https://cmake.org/cmake/help/latest/module/CTest.html#module:CTest "测试") module in our top-level `CMakeLists.txt`.

Replace:

```cmake
# enable testing
enable_testing()


```

With:

```cmake
# enable dashboard scripting
include(CTest)


```

The [`CTest`](https://cmake.org/cmake/help/latest/module/CTest.html#module:CTest "测试") module will automatically call `enable_testing()`, so we can remove it from our CMake files.

We will also need to create a `CTestConfig.cmake` file in the top-level directory where we can specify the name of the project and where to submit the dashboard.

```cmake
set(CTEST_PROJECT_NAME "CMakeTutorial")
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")

set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)


```

The [`ctest`](https://cmake.org/cmake/help/latest/manual/ctest.1.html#manual:ctest(1) "ctest（1）") executable will read in this file when it runs. To create a simple dashboard you can run the [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") executable or the [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1) "cmake-gui（1）") to configure the project, but do not build it yet. Instead, change directory to the binary tree, and then run:

> ctest [-VV] -D Experimental

Remember, for multi-config generators (e.g. Visual Studio), the configuration type must be specified:

```shell
ctest [-VV] -C Debug -D Experimental


```

Or, from an IDE, build the `Experimental` target.

The [`ctest`](https://cmake.org/cmake/help/latest/manual/ctest.1.html#manual:ctest(1) "ctest（1）") executable will build and test the project and submit the results to Kitware’s public dashboard: [https://my.cdash.org/index.php?project=CMakeTutorial](https://my.cdash.org/index.php?project=CMakeTutorial).

[Adding Generator Expressions (Step 10)](#id17)[¶](#adding-generator-expressions-step-10 "固定到该标题")
--------------------------------------------------------------------------------------------------

[`Generator expressions`](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions（7）") are evaluated during build system generation to produce information specific to each build configuration.

[`Generator expressions`](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions（7）") are allowed in the context of many target properties, such as [`LINK_LIBRARIES`](https://cmake.org/cmake/help/latest/prop_tgt/LINK_LIBRARIES.html#prop_tgt:LINK_LIBRARIES "LINK_LIBRARIES"), [`INCLUDE_DIRECTORIES`](https://cmake.org/cmake/help/latest/prop_tgt/INCLUDE_DIRECTORIES.html#prop_tgt:INCLUDE_DIRECTORIES "INCLUDE_DIRECTORIES"), [`COMPILE_DEFINITIONS`](https://cmake.org/cmake/help/latest/prop_tgt/COMPILE_DEFINITIONS.html#prop_tgt:COMPILE_DEFINITIONS "COMPILE_DEFINITIONS") and others. They may also be used when using commands to populate those properties, such as [`target_link_libraries()`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries"), [`target_include_directories()`](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories "target_include_directories"), [`target_compile_definitions()`](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions "target_compile_definitions") and others.

[`Generator expressions`](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions（7）") may be used to enable conditional linking, conditional definitions used when compiling, conditional include directories and more. The conditions may be based on the build configuration, target properties, platform information or any other queryable information.

There are different types of [`generator expressions`](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions（7）") including Logical, Informational, and Output expressions.

Logical expressions are used to create conditional output. The basic expressions are the 0 and 1 expressions. A `$<0:...>` results in the empty string, and `<1:...>` results in the content of “…”. They can also be nested.

A common usage of [`generator expressions`](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions（7）") is to conditionally add compiler flags, such as those for language levels or warnings. A nice pattern is to associate this information to an `INTERFACE` target allowing this information to propagate. Lets start by constructing an `INTERFACE` target and specifying the required C++ standard level of `11` instead of using [`CMAKE_CXX_STANDARD`](https://cmake.org/cmake/help/latest/variable/CMAKE_CXX_STANDARD.html#variable:CMAKE_CXX_STANDARD "CMAKE_CXX_STANDARD").

So the following code:

```cmake
# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)


```

Would be replaced with:

```cmake
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)


```

Next we add the desired compiler warning flags that we want for our project. As warning flags vary based on the compiler we use the `COMPILE_LANG_AND_ID` generator expression to control which flags to apply given a language and a set of compiler ids as seen below:

```cmake
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)


```

Looking at this we see that the warning flags are encapsulated inside a `BUILD_INTERFACE` condition. This is done so that consumers of our installed project will not inherit our warning flags.

**Exercise**: Modify `MathFunctions/CMakeLists.txt` so that all targets have a [`target_link_libraries()`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries") call to `tutorial_compiler_flags`.

[Adding Export Configuration (Step 11)](#id18)[¶](#adding-export-configuration-step-11 "固定到该标题")
------------------------------------------------------------------------------------------------

During [Installing and Testing (Step 4)](#installing-and-testing-step-4) of the tutorial we added the ability for CMake to install the library and headers of the project. During [Building an Installer (Step 7)](#building-an-installer-step-7) we added the ability to package up this information so it could be distributed to other people.

The next step is to add the necessary information so that other CMake projects can use our project, be it from a build directory, a local install or when packaged.

The first step is to update our [`install(TARGETS)`](https://cmake.org/cmake/help/latest/command/install.html#command:install "安装") commands to not only specify a `DESTINATION` but also an `EXPORT`. The `EXPORT` keyword generates and installs a CMake file containing code to import all targets listed in the install command from the installation tree. So let’s go ahead and explicitly `EXPORT` the MathFunctions library by updating the `install` command in `MathFunctions/CMakeLists.txt` to look like:

```cmake
install(TARGETS MathFunctions tutorial_compiler_flags
        DESTINATION lib
        EXPORT MathFunctionsTargets)
install(FILES MathFunctions.h DESTINATION include)


```

Now that we have MathFunctions being exported, we also need to explicitly install the generated `MathFunctionsTargets.cmake` file. This is done by adding the following to the bottom of the top-level `CMakeLists.txt`:

```cmake
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)
```

At this point you should try and run CMake. If everything is setup properly you will see that CMake will generate an error that looks like:

```log
Target "MathFunctions" INTERFACE_INCLUDE_DIRECTORIES property contains
path:

  "/Users/robert/Documents/CMakeClass/Tutorial/Step11/MathFunctions"

which is prefixed in the source directory.
```

What CMake is trying to say is that during generating the export information it will export a path that is intrinsically tied to the current machine and will not be valid on other machines. The solution to this is to update the MathFunctions [`target_include_directories()`](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories "target_include_directories") to understand that it needs different `INTERFACE` locations when being used from within the build directory and from an install / package. This means converting the [`target_include_directories()`](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories "target_include_directories") call for MathFunctions to look like:

```cmake
target_include_directories(MathFunctions
                           INTERFACE
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                            $<INSTALL_INTERFACE:include>
                           )


```

Once this has been updated, we can re-run CMake and verify that it doesn’t warn anymore.

At this point, we have CMake properly packaging the target information that is required but we will still need to generate a `MathFunctionsConfig.cmake` so that the CMake [`find_package()`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package "find_package") command can find our project. So let’s go ahead and add a new file to the top-level of the project called `Config.cmake.in` with the following contents:

```cmake
@PACKAGE_INIT@

include ( "${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake" )


```

Then, to properly configure and install that file, add the following to the bottom of the top-level `CMakeLists.txt`:

```cmake
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)

include(CMakePackageConfigHelpers)
# generate the config file that is includes the exports
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/example"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )
# generate the version file for the config file
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
  COMPATIBILITY AnyNewerVersion
)

# install the configuration file
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
  DESTINATION lib/cmake/MathFunctions
  )


```

At this point, we have generated a relocatable CMake Configuration for our project that can be used after the project has been installed or packaged. If we want our project to also be used from a build directory we only have to add the following to the bottom of the top level `CMakeLists.txt`:

```cmake
export(EXPORT MathFunctionsTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsTargets.cmake"
)


```

With this export call we now generate a `Targets.cmake`, allowing the configured `MathFunctionsConfig.cmake` in the build directory to be used by other projects, without needing it to be installed.

[Packaging Debug and Release (Step 12)](#id19)[¶](#packaging-debug-and-release-step-12 "固定到该标题")
------------------------------------------------------------------------------------------------

**Note:** This example is valid for single-configuration generators and will not work for multi-configuration generators (e.g. Visual Studio).

By default, CMake’s model is that a build directory only contains a single configuration, be it Debug, Release, MinSizeRel, or RelWithDebInfo. It is possible, however, to setup CPack to bundle multiple build directories and construct a package that contains multiple configurations of the same project.

First, we want to ensure that the debug and release builds use different names for the executables and libraries that will be installed. Let’s use d as the postfix for the debug executable and libraries.

Set [`CMAKE_DEBUG_POSTFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_DEBUG_POSTFIX.html#variable:CMAKE_DEBUG_POSTFIX "CMAKE_DEBUG_POSTFIX") near the beginning of the top-level `CMakeLists.txt` file:

```cmake
set(CMAKE_DEBUG_POSTFIX d)

add_library(tutorial_compiler_flags INTERFACE)


```

And the [`DEBUG_POSTFIX`](https://cmake.org/cmake/help/latest/prop_tgt/DEBUG_POSTFIX.html#prop_tgt:DEBUG_POSTFIX "DEBUG_POSTFIX") property on the tutorial executable:

```cmake
add_executable(Tutorial tutorial.cxx)
set_target_properties(Tutorial PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})

target_link_libraries(Tutorial PUBLIC MathFunctions)


```

Let’s also add version numbering to the MathFunctions library. In `MathFunctions/CMakeLists.txt`, set the [`VERSION`](https://cmake.org/cmake/help/latest/prop_tgt/VERSION.html#prop_tgt:VERSION "版") and [`SOVERSION`](https://cmake.org/cmake/help/latest/prop_tgt/SOVERSION.html#prop_tgt:SOVERSION "覆盖") properties:

```cmake
set_property(TARGET MathFunctions PROPERTY VERSION "1.0.0")
set_property(TARGET MathFunctions PROPERTY SOVERSION "1")


```

From the `Step12` directory, create `debug` and `release` subbdirectories. The layout will look like:

```log
- Step12
   └── debug
   └── release


```

Now we need to setup debug and release builds. We can use [`CMAKE_BUILD_TYPE`](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html#variable:CMAKE_BUILD_TYPE "CMAKE_BUILD_TYPE") to set the configuration type:

```shell
cd debug
cmake -DCMAKE_BUILD_TYPE=Debug ..
cmake --build .
cd ../release
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .


```

Now that both the debug and release builds are complete, we can use a custom configuration file to package both builds into a single release. In the `Step12` directory, create a file called `MultiCPackConfig.cmake`. In this file, first include the default configuration file that was created by the [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1) "cmake（1）") executable.

Next, use the `CPACK_INSTALL_CMAKE_PROJECTS` variable to specify which projects to install. In this case, we want to install both debug and release.

```cmake
include("release/CPackConfig.cmake")

set(CPACK_INSTALL_CMAKE_PROJECTS
    "debug;Tutorial;ALL;/"
    "release;Tutorial;ALL;/"
    )


```

From the `Step12` directory, run [`cpack`](https://cmake.org/cmake/help/latest/manual/cpack.1.html#manual:cpack(1) "cpack（1）")使用以下`config`选项指定我们的自定义配置文件：

```shell
cpack --config MultiCPackConfig.cmake
```