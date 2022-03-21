<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [引言](#引言)
- [使用包](#使用包)
- [Config-file Packages](#config-file-packages)
- [Find-module Packages](#find-module-packages)
- [Package Configuration File](#package-configuration-file)
- [Package Version File](#package-version-file)
- [创建包](#创建包)
- [创建一个包配置文件](#创建一个包配置文件)
- [为构建树创建一个包配置文件](#为构建树创建一个包配置文件)
- [创建可重定位包](#创建可重定位包)
- [包注册](#包注册)
  - [用户包注册](#用户包注册)
  - [系统包注册](#系统包注册)
  - [禁用包注册](#禁用包注册)
  - [包注册实例：](#包注册实例)
  - [包注册表所有权](#包注册表所有权)

<!-- /code_chunk_output -->
# 引言
包(Package)提供了基于CMake构建系统的依赖信息。包通过find_package()命令查找，查找的结果要么是导入目标的集合或者有关构建信息的变量集合。
# 使用包
CMake对两种形式的包提供了直接支持。`Config-file Packages` 和 `Find-module Packages`。通过FindPkgConfig模块间接提供了对pkg-config Packages的支持。
find_package()的基础调用方式如下：
```cmake
find_package(Qt4 4.7.0 REQUIRED) # CMake provides a Qt4 find-module
find_package(Qt5Core 5.1.0 REQUIRED) # Qt provides a Qt5 package config file.
find_package(LibXml2 REQUIRED) # Use pkg-config via the LibXml2 find-module
```
如果只需要上游库的包配置文件，可以使用CONFIG关键词指定：
```cmake
find_package(Qt5Core 5.1.0 CONFIG REQUIRED)
find_package(Qt5Gui 5.1.0 CONFIG)
```
类似MODULE关键词用于只需要查找包模块：
```cmake
find_package(Qt4 4.7.0 MODULE REQUIRED)
```
在没有找到指定类型的包的情况下，将显示错误信息。两种查找方式都支持指定类形的包，可以使用`REQUIRED`:
```cmake
find_package(Qt5 5.1.0 CONFIG REQUIRED Widgets Xml Sql)
```
或者使用分割`COMPONENTS`列表：
```cmake
find_package(Qt5 5.1.0 COMPONENTS Widgets Xml Sql)
```
或者使用分割`OPTIONAL_COMPONENTS `列表：
```cmake
find_package(Qt5 5.1.0 COMPONENTS Widgets
                       OPTIONAL_COMPONENTS Xml Sql
)
```
通过设置`CMAKE_DISABLE_FIND_PACKAGE_<PackageName>`变量为真，`<PackageName>`将不会被查找，而设置`CMAKE_REQUIRE_FIND_PACKAGE_<PackageName>`变量为真，将使包必需。
# Config-file Packages 
Config-file Packages是上游库提供给下游使用的文件集合。CMake从一组路径位置查找包配置文件。如果要查找不标准前缀的包，最简单的方式是设置`CMAKE_PREFIX_PATH`缓存变量。
`<PackageName>_FOUND`变量会被设置为真或假，取决于包有没有被找到。`<PackageName>_DIR`缓存路径设置了包配置文件的位置。

# Find-module Packages
Find-module Packages是一个文件伴随着一系列规则用于找到依赖所需的片段
，主要是头文件和库。通常一个Find-module用于上游库不是使用CMake构建的，或者没有提供Config-file Packages情况下。与包配置文件不同，它不是上游提供的，而是由下游使用带有特定于平台提示的文件位置来查找文件。
不同于上游提供的包配置文件，在包会被发现的过程中没有引用标识，所有`<PackageName>_FOUND`变量是不会自动设置。
# Package Configuration File
一个配置文件包由一个包配置文件和可选的包版本文件组成。
考虑一个项目`Foo`安装如下文件：
```shell
<prefix>/include/foo-1.2/foo.h
<prefix>/lib/foo-1.2/libfoo.a
```
这可能提供一个CMake包配置文件：
```shell
<prefix>/lib/cmake/foo-1.2/FooConfig.cmake
```
内容定义了IMPORTED目标，或者定义变量，如：
```cmake
# ...
# (compute PREFIX relative to file location)
# ...
set(Foo_INCLUDE_DIRS ${PREFIX}/include/foo-1.2)
set(Foo_LIBRARIES ${PREFIX}/lib/foo-1.2/libfoo.a)
```
如果其他项目希望使用`Foo`，仅仅需要定位`FooConfig.cmake`文件，并加载获取包内容定位的所有相关信息。因为包配置文件是由已经知道所有文件位置的安装包提供。
find_package()命令给定`Foo`，将会查找`FooConfig.cmake`或`foo-config.cmake`。查找的路径在find_package()命令文档有描述，可能如下：
```cmake
<prefix>/lib/cmake/Foo*/
```
# Package Version File
用于查找版本文件并加载，测试包版本是否和需要的版本匹配，如果版本文件匹配则配置文件被接受，否则将忽视。
包版本文件和包配置文件名字部分一致，并在后缀前附属version或 Version，如下：
```cmake
<prefix>/lib/cmake/foo-1.3/foo-config.cmake
<prefix>/lib/cmake/foo-1.3/foo-config-version.cmake
```
和
```cmake
<prefix>/lib/cmake/bar-4.2/BarConfig.cmake
<prefix>/lib/cmake/bar-4.2/BarConfigVersion.cmake
```
当find_package()命令加载一个版本文件，首先设置如下的变量：
- PACKAGE_FIND_NAME：包名称
- PACKAGE_FIND_VERSION：完整的版本字符串
- PACKAGE_FIND_VERSION_MAJOR：主版本
- PACKAGE_FIND_VERSION_MINOR：副版本
- PACKAGE_FIND_VERSION_PATCH：补丁版本
- PACKAGE_FIND_VERSION_TWEAK：Tweak version if requested, else 0
- PACKAGE_FIND_VERSION_COUNT：版本组成数，0-4

# 创建包
通常上游依赖于CMake自身可以使用CMake的便利来创建包文件。考虑一个上游提供一个单独的共享库：
```cmake
project(UpstreamLib)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_INCLUDE_CURRENT_DIR_IN_INTERFACE ON)

set(Upstream_VERSION 3.4.1)

include(GenerateExportHeader)

add_library(ClimbingStats SHARED climbingstats.cpp)
generate_export_header(ClimbingStats)
set_property(TARGET ClimbingStats PROPERTY VERSION ${Upstream_VERSION})
set_property(TARGET ClimbingStats PROPERTY SOVERSION 3)
set_property(TARGET ClimbingStats PROPERTY
  INTERFACE_ClimbingStats_MAJOR_VERSION 3)
set_property(TARGET ClimbingStats APPEND PROPERTY
  COMPATIBLE_INTERFACE_STRING ClimbingStats_MAJOR_VERSION
)

install(TARGETS ClimbingStats EXPORT ClimbingStatsTargets
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)
install(
  FILES
    climbingstats.h
    "${CMAKE_CURRENT_BINARY_DIR}/climbingstats_export.h"
  DESTINATION
    include
  COMPONENT
    Devel
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/ClimbingStats/ClimbingStatsConfigVersion.cmake"
  VERSION ${Upstream_VERSION}
  COMPATIBILITY AnyNewerVersion
)

export(EXPORT ClimbingStatsTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/ClimbingStats/ClimbingStatsTargets.cmake"
  NAMESPACE Upstream::
)
configure_file(cmake/ClimbingStatsConfig.cmake
  "${CMAKE_CURRENT_BINARY_DIR}/ClimbingStats/ClimbingStatsConfig.cmake"
  COPYONLY
)

set(ConfigPackageLocation lib/cmake/ClimbingStats)
install(EXPORT ClimbingStatsTargets
  FILE
    ClimbingStatsTargets.cmake
  NAMESPACE
    Upstream::
  DESTINATION
    ${ConfigPackageLocation}
)
install(
  FILES
    cmake/ClimbingStatsConfig.cmake
    "${CMAKE_CURRENT_BINARY_DIR}/ClimbingStats/ClimbingStatsConfigVersion.cmake"
  DESTINATION
    ${ConfigPackageLocation}
  COMPONENT
    Devel
)
```
The CMakePackageConfigHelpers module provides a macro for creating a simple ConfigVersion.cmake file. This file sets the version of the package. It is read by CMake when find_package() is called to determine the compatibility with the requested version, and to set some version-specific variables <PackageName>_VERSION, <PackageName>_VERSION_MAJOR, <PackageName>_VERSION_MINOR etc.The install(EXPORT) command is used to export the targets in the ClimbingStatsTargets export-set, defined previously by the install(TARGETS) command. This command generates the ClimbingStatsTargets.cmake file to contain IMPORTED targets, suitable for use by downstreams and arranges to install it to lib/cmake/ClimbingStats. The generated ClimbingStatsConfigVersion.cmake and a cmake/ClimbingStatsConfig.cmake are installed to the same location, completing the package。
生成的IMPORTED目标有合适的属性来定义他们的应用需求，比如INTERFACE_INCLUDE_DIRECTORIES, INTERFACE_COMPILE_DEFINITIONS和其他无关内建接口属性。在COMPATIBLE_INTERFACE_STRING和其他COMPATIBLE INTERFACE properties中列出的用户定义属性的INTERFACE变体也被传播到生成的导入目标。
带有双冒号的NAMESPACE是在导出安装目标时指定的。这种双冒号的约定给了CMake一个提示，当它被下游用target link libraries()命令使用时，它是一个IMPORTED目标。这样，如果还没有找到提供诊断的包，CMake可以发出诊断。
在本例中，当使用install(TARGETS)时，指定了INCLUDES DESTINATION。这将导致导入的目标在CMAKE INSTALL PREFIX中使用包含目录填充其接口包含目录。当IMPORTED目标被下游使用时，它会自动使用来自该属性的条目。

# 创建一个包配置文件
`ClimbingStatsConfig.cmake`文件可以简单如下：
```cmake
include("${CMAKE_CURRENT_LIST_DIR}/ClimbingStatsTargets.cmake")
```
这个让下游使用IMPORTED目标。如果任何宏需要ClimbingStats包提供，它们应该在一个单独的文件中，安装到与ClimbingStatsConfig.Cmake相同的位置文件，并包含在其中。
这也需要被扩展到包裹依赖：
```cmake
# ...
add_library(ClimbingStats SHARED climbingstats.cpp)
generate_export_header(ClimbingStats)

find_package(Stats 2.6.4 REQUIRED)
target_link_libraries(ClimbingStats PUBLIC Stats::Types)
```
因为Stats::Types目标是ClimbingStats的PUBLIC依赖，下游同样需要找到Stas包并链接到Stats::Types库。Stats包应当在ClimbingStatsConfig.cmake能找到。所有REQUIRED依赖包都需要从Config.cmake文件中找到：
```cmake
include(CMakeFindDependencyMacro)
find_dependency(Stats 2.6.4)

include("${CMAKE_CURRENT_LIST_DIR}/ClimbingStatsTargets.cmake")
include("${CMAKE_CURRENT_LIST_DIR}/ClimbingStatsMacros.cmake")
```
如果没有找到该依赖项，find_dependency宏还会将ClimbingStats_FOUND的设置为False，并提供一个诊断--没有Stats包，就不能使用ClimbingStats包。
如果`COMPONENTS`在下游使用find_packages()时指定，他们会列入到`<PackageName>_FIND_COMPONENTS`参数中。如果一个特别的组件是无选择的，那么`<PackageName>_FIND_REQUIRED_<comp>`将为真：
```cmake
include(CMakeFindDependencyMacro)
find_dependency(Stats 2.6.4)

include("${CMAKE_CURRENT_LIST_DIR}/ClimbingStatsTargets.cmake")
include("${CMAKE_CURRENT_LIST_DIR}/ClimbingStatsMacros.cmake")

set(_supported_components Plot Table)

foreach(_comp ${ClimbingStats_FIND_COMPONENTS})
  if (NOT ";${_supported_components};" MATCHES ";${_comp};")
    set(ClimbingStats_FOUND False)
    set(ClimbingStats_NOT_FOUND_MESSAGE "Unsupported component: ${_comp}")
  endif()
  include("${CMAKE_CURRENT_LIST_DIR}/ClimbingStats${_comp}Targets.cmake")
endforeach()
```
`ClimbingStats_NOT_FOUND_MESSAGE` 被设置为一个诊断，即无法找到包，因为指定了无效的组件。在FOUND变量设置为False的任何情况下，都可以设置此消息变量，并将其显示给用户。

# 为构建树创建一个包配置文件
export命令创建一个IMPORTED 目标定义文件，由构建树指定，且不可重分配。类似地，这可以与合适的包配置文件和包版本文件一起使用，以定义构建树的包，无需安装即可使用。构建树的用户可以简单的确保 `CMAKE_PREFIX_PATH`包含构建路径，或设置`ClimbingStats_DIR`到`<build_dir>/ClimbingStats`在缓存中。

# 创建可重定位包
可重定位包不能引用构建包的机器上文件的绝对路径，这些文件路径可能不存在安装软件包的机器上。
通过`install(EXPORT)`创建的包被设计成可重定位，使用包的位置的相对路径。当顶一个目标接口为`EXPORT`，记住包含文件夹指定的是相对路径，且相对于`CMAKE_INSTALL_PREFIX`。
```cmake
target_include_directories(tgt INTERFACE
  # Wrong, not relocatable:
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_PREFIX}/include/TgtName>
)

target_include_directories(tgt INTERFACE
  # Ok, relocatable:
  $<INSTALL_INTERFACE:include/TgtName>
)
```
使用`$<INSTALL_PREFIX>` 生成表达式用作安装前缀的占位符，不会导致不可重定位的包。 如果使用复杂的生成器表达式，这是必要的:
```cmake
target_include_directories(tgt INTERFACE
  # Ok, relocatable:
  $<INSTALL_INTERFACE:$<$<CONFIG:Debug>:$<INSTALL_PREFIX>/include/TgtName>>
)
```
这也适用于引用外部依赖项的路径。 不建议使用与依赖关系相关的路径填充任何可能包含路径的属性，例如 INTERFACE_INCLUDE_DIRECTORIES 和 INTERFACE_LINK_LIBRARIES。 例如，此代码可能不适用于可重定位的包：
```cmake
target_link_libraries(ClimbingStats INTERFACE
  ${Foo_LIBRARIES} ${Bar_LIBRARIES}
  )
target_include_directories(ClimbingStats INTERFACE
  "$<INSTALL_INTERFACE:${Foo_INCLUDE_DIRS};${Bar_INCLUDE_DIRS}>"
  )
```
引用的变量可能包含库的绝对路径，并包括在制作包的机器上找到的目录。 这将创建一个包含硬编码路径的包，不适合重定位。
理想情况下，此类依赖项应通过其自己的 IMPORTED targets 使用，这些目标具有自己的 IMPORTED_LOCATION 和适当填充的 INTERFACE_INCLUDE_DIRECTORIES 等使用要求属性。 然后可以将这些导入的目标与 ClimbingStats 的 target_link_libraries() 命令一起使用:
```cmake
target_link_libraries(ClimbingStats INTERFACE Foo:Foo Bar::Bar)
```
使用这种方法，包仅通过 IMPORTED 目标的名称引用其外部依赖项。 当消费者使用已安装的包时，消费者将运行适当的 find_package() 命令（通过上述的 find_dependency 宏）来查找依赖项并在他们自己的机器上使用适当的路径填充导入的目标。
不幸的是，许多 CMake 附带的模块还没有提供 IMPORTED 目标，因为它们的开发早于这种方法。 随着时间的推移，这可能会逐渐改善。 使用此类模块创建可重定位包的解决方法包括：
- 构建包时，将每个 Foo_LIBRARY 缓存条目指定为库名称，例如 -DFoo_LIBRARY=foo。 这告诉相应的查找模块用 foo 填充 Foo_LIBRARIES 以要求链接器搜索库而不是硬编码路径。
- 或者，在安装包内容之后但在创建用于重新分发的包安装二进制文件之前，手动将绝对路径替换为占位符，以便在安装包时由安装工具替换。

# 包注册
CMake 提供了两个中心位置来注册已在系统上任何位置构建或安装的包：
- 用户包注册
- 系统包注册
注册表对于帮助项目在非标准安装位置或直接在自己的构建树中查找包特别有用。 项目可以填充用户或系统注册表（使用其自己的方式，见下文）以引用其位置。 在任何一种情况下，包都应在注册位置存储包配置文件 (<PackageName>Config.cmake) 和可选的包版本文件 (<PackageName>ConfigVersion.cmake)。
find_package() 命令将两个包注册表作为其文档中指定的两个搜索步骤进行搜索。 如果它具有足够的权限，它还会删除陈旧的包注册表条目，这些条目引用不存在或不包含匹配包配置文件的目录。

## 用户包注册
用户包注册表存储在每个用户的位置。 export(PACKAGE) 命令可用于在用户包注册表中注册项目构建树。 CMake 目前没有提供将安装树添加到用户包注册表的接口。 如果需要，必须手动教导安装人员注册他们的软件包。
Windows用户包注册存储在Windows注册表中`HKEY_CURRENT_USER`:
<PackageName> 可能会出现在注册表项下：
```shell
HKEY_CURRENT_USER\Software\Kitware\CMake\Packages\<PackageName>
```
作为 REG_SZ 值，具有任意名称，它指定包含包配置文件的目录。
在UNIX平台用户包注册存储在用户 home 文件夹下 `~/.cmake/packages`。一个`<PackageName>`可能显示如下：
```shell
~/.cmake/packages/<PackageName>
```
作为一个文件，具有任意名称，其内容指定包含包配置文件的目录。

## 系统包注册
系统包注册表存储在系统范围的位置。 CMake 目前没有提供添加到系统包注册表的接口。 如果需要，必须手动教导安装人员注册他们的软件包。
在 Windows 上，系统包注册表存储在 Windows 注册表中的 HKEY_LOCAL_MACHINE 键下。 <PackageName> 可能会出现在注册表项下：
```shell
HKEY_LOCAL_MACHINE\Software\Kitware\CMake\Packages\<PackageName>
```
作为 REG_SZ 值，具有任意名称，它指定包含包配置文件的目录。
非windows平台是没有系统包注册的。

## 禁用包注册
在某些情况下，使用包注册表是不可取的。 CMake 允许使用以下变量禁用它们：
- 当 CMP0090 设置为 NEW 时，export(PACKAGE) 命令不会填充用户包注册表，除非 CMAKE_EXPORT_PACKAGE_REGISTRY 变量显式启用它。 当 CMP0090 未设置为 NEW 时，export(PACKAGE) 会填充用户包注册表，除非 CMAKE_EXPORT_NO_PACKAGE_REGISTRY 变量明确禁用它。
- 当设置为 FALSE 时，CMAKE_FIND_USE_PACKAGE_REGISTRY 在所有 find_package() 调用中禁用用户包注册表。
- 当设置为 TRUE 时，已弃用的 CMAKE_FIND_PACKAGE_NO_PACKAGE_REGISTRY 在所有 find_package() 调用中禁用用户包注册表。 当 CMAKE_FIND_USE_PACKAGE_REGISTRY 已设置时，此变量将被忽略。
- CMAKE_FIND_PACKAGE_NO_SYSTEM_PACKAGE_REGISTRY 在所有 find_package() 调用中禁用系统包注册表。
## 包注册实例：
命名包注册表项的一个简单约定是使用内容哈希。 它们是确定性的，不太可能发生冲突（export(PACKAGE) 使用这种方法）。 引用特定目录的条目名称只是目录路径本身的内容哈希。
如果项目安排包注册表项存在，例如：
```shell
 reg query HKCU\Software\Kitware\CMake\Packages\MyPackage
HKEY_CURRENT_USER\Software\Kitware\CMake\Packages\MyPackage
 45e7d55f13b87179bb12f907c8de6fc4 REG_SZ c:/Users/Me/Work/lib/cmake/MyPackage
 7b4a9844f681c80ce93190d4e3185db9 REG_SZ c:/Users/Me/Work/MyPackage-build
```
或
```shell
$ cat ~/.cmake/packages/MyPackage/7d1fb77e07ce59a81bed093bbee945bd
/home/me/work/lib/cmake/MyPackage
$ cat ~/.cmake/packages/MyPackage/f92c1db873a1937f3100706657c63e07
/home/me/work/MyPackage-build
```
然后`CMakeLists.txt`代码：
```cmake
find_package(MyPackage)
```
将在注册位置搜索包配置文件（MyPackageConfig.cmake）。 单个包的包注册表条目之间的搜索顺序未指定，条目名称（本示例中的哈希）没有意义。 注册的位置可能包含包版本文件（MyPackageConfigVersion.cmake），以告诉 find_package() 特定位置是否适合请求的版本。

## 包注册表所有权
包注册表项由它们引用的项目安装单独拥有。 包安装程序负责添加自己的条目，相应的卸载程序负责删除它。
export(PACKAGE) 命令使用项目构建树的位置填充用户包注册表。 构建树往往会被开发人员删除，并且没有可能触发删除其条目的“卸载”事件。 为了保持注册表清洁，如果 find_package() 命令具有足够的权限，它会自动删除它遇到的陈旧条目。 一旦调用 export(PACKAGE)，CMake 不提供删除引用现有构建树的条目的接口。 但是，如果项目从构建树中删除其包配置文件，则引用该位置的条目将被视为陈旧。