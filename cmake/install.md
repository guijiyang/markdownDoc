# 使用方式
```cmake
install(TARGETS <target>... [...])
install(IMPORTED_RUNTIME_ARTIFACTS <target>... [...])
install({FILES | PROGRAMS} <file>... [...])
install(DIRECTORY <dir>... [...])
install(SCRIPT <file> [...])
install(CODE <code> [...])
install(EXPORT <export-name> [...])
install(RUNTIME_DEPENDENCY_SET <set-name> [...])
```
# 介绍
此命令为项目生成安装规则。 通过调用源目录中的 install() 命令指定的安装规则在安装期间按顺序执行。
在 3.14 版更改：通过调用 add_subdirectory() 命令添加的子目录中的安装规则与父目录中的规则交错，以按声明的顺序运行（参见策略 CMP0082）。 
在 3.22 版更改: 环境变量 CMAKE_INSTALL_MODE 可以覆盖 install() 的默认复制行为。
此命令有多个签名。 其中一些定义了文件和目标的安装选项。 此处涵盖了多个签名共有的选项，但它们仅对指定它们的签名有效。 常见的选项是：
- DESTINATION
    指定要安装文件的磁盘目录。 参数可以是相对路径或绝对路径。
    如果给出了相对路径，则相对于 CMAKE_INSTALL_PREFIX 变量的值进行解释。 可以使用 CMAKE_INSTALL_PREFIX 变量文档中解释的 DESTDIR 机制在安装时重新定位前缀。
    如果给出了绝对路径（带有前导斜杠或驱动器号），则将逐字使用。
    由于 cpack 安装程序生成器不支持绝对路径，因此最好始终使用相对路径。 特别是，不需要通过预先添加 CMAKE_INSTALL_PREFIX 来使路径成为绝对路径； 如果 DESTINATION 是相对路径，则默认使用此前缀。
    
- PERMISSIONS
    指定已安装文件的权限。 有效权限为 OWNER_READ、OWNER_WRITE、OWNER_EXECUTE、GROUP_READ、GROUP_WRITE、GROUP_EXECUTE、WORLD_READ、WORLD_WRITE、WORLD_EXECUTE、SETUID 和 SETGID。 在某些平台上没有意义的权限在这些平台上会被忽略。

- CONFIGURATIONS
    指定安装规则适用的构建配置列表（Debug、Release等）。 请注意，为此选项指定的值仅适用于在 CONFIGURATIONS 选项之后列出的选项。 例如，要为 Debug 和 Release 配置设置单独的安装路径，请执行以下操作：
    ```cmake
    install(TARGETS target
            CONFIGURATIONS Debug
            RUNTIME DESTINATION Debug/bin)
    install(TARGETS target
            CONFIGURATIONS Release
            RUNTIME DESTINATION Release/bin)            
    ```
    注意 CONFIGURATIONS 出现在 RUNTIME DESTINATION 之前。

- COMPONENT
    指定与安装规则关联的安装组件名称，例如“runtime”或“development”。 在特定于组件的安装期间，只会执行与给定组件名称关联的安装规则。 在完全安装期间，所有组件都会安装，除非标有 EXCLUDE_FROM_ALL。 如果未提供 COMPONENT，则会创建默认组件“未指定”。 默认组件名称可以通过 CMAKE_INSTALL_DEFAULT_COMPONENT_NAME 变量来控制。

- EXCLUDE_FROM_ALL
    3.6 版中的新功能。
    指定该文件从完整安装中排除，并且仅作为特定组件安装的一部分进行安装。

- RENAME
    指定可能与原始文件不同的已安装文件的名称。 仅当该命令安装了单个文件时才允许重命名。

- OPTIONAL
    指明被安装的文件不存在时不报错。

3.1 版新功能：安装文件的命令签名可能会在安装期间打印消息。 使用 CMAKE_INSTALL_MESSAGE 变量来控制打印哪些消息。
3.11 版新功能：许多 install() 变体隐式创建包含已安装文件的目录。 如果设置了 CMAKE_INSTALL_DEFAULT_DIRECTORY_PERMISSIONS，则将使用指定的权限创建这些目录。 否则，它们将根据类 Unix 平台上的 uname 规则创建。 Windows 平台不受影响。

# 安装目标
```cmake
install(TARGETS targets... [EXPORT <export-name>]
        [RUNTIME_DEPENDENCIES args...|RUNTIME_DEPENDENCY_SET <set-name>]
        [[ARCHIVE|LIBRARY|RUNTIME|OBJECTS|FRAMEWORK|BUNDLE|
          PRIVATE_HEADER|PUBLIC_HEADER|RESOURCE]
         [DESTINATION <dir>]
         [PERMISSIONS permissions...]
         [CONFIGURATIONS [Debug|Release|...]]
         [COMPONENT <component>]
         [NAMELINK_COMPONENT <component>]
         [OPTIONAL] [EXCLUDE_FROM_ALL]
         [NAMELINK_ONLY|NAMELINK_SKIP]
        ] [...]
        [INCLUDES DESTINATION [<dir> ...]]
        )
```
TARGETS 表单指定了从项目安装目标的规则。 可以安装几种目标输出工件：
- ARCHIVE
- - 静态库(除了macOS上标记为 FRAMEWORK)；
- - DLL导入库(所有的Windows平台，包括Cygwin；含有扩展.lib，与RUNTIME的.dll不同)；
- - 在 AIX 上，为启用了 ENABLE_EXPORTS 的可执行文件创建的链接器导入文件。

- LIBRARY
- - 共享库，除了：
- - - DLLs
- - - macOS标记的 FRAMEWORK。

- RUNTIME
- - 可执行文件
- - DLLs(需要伴随着DLL导入库)

- OBJECTS
    与对象库关联的对象文件。

- FRAMEWORK
    标有 FRAMEWORK 属性的静态库和共享库都被视为 macOS 上的 FRAMEWORK 目标。
- BUNDLE
    标有 MACOSX_BUNDLE 属性的可执行文件在 macOS 上被视为 BUNDLE 目标。
- PUBLIC_HEADER
    任何与库关联的 PUBLIC_HEADER 文件都安装在非 Apple 平台上由 PUBLIC_HEADER 参数指定的目标位置。 对于 Apple 平台上的 FRAMEWORK 库，此参数定义的规则将被忽略，因为相关文件已安装到框架文件夹内的适当位置。 有关详细信息，请参阅 PUBLIC_HEADER。
- PRIVATE_HEADER
    类似于 PUBLIC_HEADER，但适用于 PRIVATE_HEADER 文件。 有关详细信息，请参阅 PRIVATE_HEADER。
- RESOURCE
    类似于 PUBLIC_HEADER 和 PRIVATE_HEADER，但用于 RESOURCE 文件。 有关详细信息，请参阅资源。

对于给定的每个参数，它们后面的参数仅适用于参数中指定的目标或文件类型。 如果没有给出，则安装属性适用于所有目标类型。 如果只给出一个，那么只会安装该类型的目标（可用于仅安装 DLL 或仅安装导入库。）
对于常规可执行文件、静态库和共享库，不需要 DESTINATION 参数。 对于这些目标类型，当省略 DESTINATION 时，将从 GNUInstallDirs 的适当变量中获取默认目标，如果未定义该变量，则将其设置为内置默认值。 对于通过 PUBLIC_HEADER 和 PRIVATE_HEADER 目标属性与已安装目标关联的公共和私有标头也是如此。 必须始终为模块库、Apple 包和框架提供目的地。 接口和对象库可以省略目标，但它们的处理方式不同（请参阅本节末尾对本主题的讨论）。

下表显示了目标类型及其关联变量和在未指定目标时适用的内置默认值：

|目标类型|GNUInstallDirs变量|内置默认值|
|:-|:-|:-|
|RUNTIME|${CMAKE_INSTALL_BINDIR}|bin|
|LIBRARY|${CMAKE_INSTALL_LIBDIR}|lib|
|ARCHIVE|${CMAKE_INSTALL_LIBDIR}|lib|
|PRIVATE_HEADER|${CMAKE_INSTALL_INCLUDEDIR}|include|
|PUBLIC_HEADER|${CMAKE_INSTALL_INCLUDEDIR}|include|
希望遵循将标头安装到项目特定子目录中的常见做法的项目将需要提供目标而不是依赖于上述内容。
为了使包符合分发文件系统布局策略，如果项目必须指定 DESTINATION，建议它们使用以适当的 GNUInstallDirs 变量开头的路径。 这允许包维护者通过设置适当的缓存变量来控制安装目标。 以下示例显示了一个静态库被安装到 GNUInstallDirs 提供的默认目标，但其头文件安装到遵循上述建议的项目特定子目录：
```cmake
add_library(mylib STATIC ...)
set_target_properties(mylib PROPERTIES PUBLIC_HEADER mylib.h)
include(GNUInstallDirs)
install(TARGETS mylib
        PUBLIC_HEADER
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/myproj)
```
除了上面列出的常用选项之外，每个目标都可以接受以下附加参数：
- NAMELINK_COMPONENT
    版本 3.12 中的新功能。
    在某些平台上，版本化共享库具有符号链接，例如：
    `lib<name>.so -> lib<name>.so1`
    其中 lib\<name>.so.1 是库的 soname，而 lib\<name>.so 是“名称链接”，允许链接器在给定 -l<name> 时找到库。 NAMELINK_COMPONENT 选项类似于 COMPONENT 选项，但如果生成了共享库名称链接，它会更改安装组件。 如果未指定，则默认为 COMPONENT 的值。 在 LIBRARY 块之外使用此参数是错误的。
    考虑下面的例子：
    ```cmake
    install(TARGETS mylib
            LIBRARY 
                COMPONENT Libraries
                NAMELINKE_COMPONENT Development
            PUBLIC_HEADER
                COMPONENT Development)
    ```
    在这种情况下，如果您选择仅安装 Development 组件，则头文件和名称链接都将在没有库的情况下安装。 （如果您不安装 Libraries 组件，namelink 将是一个悬空符号链接，并且链接到该库的项目将出现构建错误。）如果您只安装 Libraries 组件，则只会安装该库，而没有 标题和名称链接。
    此选项通常用于具有单独运行时和开发包的包管理器。例如，在 Debian 系统上，库应该在 runtime 包中，而 headers 和 namelink 应该在 development 包中。
- NAMELINK_ONLY
    此选项导致在安装库目标时仅安装名称链接。 在版本化共享库没有名称链接的平台上或当库没有版本化时，NAMELINK_ONLY 选项不会安装任何内容。 在 LIBRARY 块之外使用此参数是错误的。
- NAMELINK_SKIP
    类似于 NAMELINK_ONLY，但它具有相反的效果：它会在安装库目标时导致安装名称链接以外的库文件。 当 NAMELINK_ONLY 或 NAMELINK_SKIP 都没有给出时，这两个部分都被安装。 在版本化共享库没有符号链接的平台上或库没有版本化时，NAMELINK_SKIP 会安装该库。 在 LIBRARY 块之外使用此参数是错误的。
    如果指定了 NAMELINK_SKIP，则 NAMELINK_COMPONENT 无效。 不建议将 NAMELINK_SKIP 与 NAMELINK_COMPONENT 结合使用。

install(TARGETS) 命令也可以在顶层接受下面的选项：
- EXPORT
    此选项将已安装的目标文件与名为 \<export-name> 的导出相关联。 它必须出现在任何目标选项之前。 要实际安装导出文件本身，请调用 install(EXPORT)，如下所述。 请参阅 EXPORT_NAME 目标属性的文档以更改导出目标的名称。
- INCLUDES DESTINATION
    此选项指定目录列表，当由 install(EXPORT) 命令导出时，这些目录将添加到 \<targets> 的 INTERFACE_INCLUDE_DIRECTORIES 目标属性中。 如果指定了相对路径，则将其视为相对于 $<INSTALL_PREFIX>。
- RUNTIME_DEPENDENCY_SET
    3.21 版中的新功能。
    此选项会导致已安装的可执行文件、共享库和模块目标的所有运行时依赖项添加到指定的运行时依赖项集中。 然后可以使用 install(RUNTIME_DEPENDENCY_SET) 命令安装此集。
    此关键字和 RUNTIME_DEPENDENCIES 关键字是互斥的。
- RUNTIME_DEPENDENCIES
    3.21 版中的新功能。
    此选项会导致已安装的可执行文件、共享库和模块目标的所有运行时依赖项与目标本身一起安装。 RUNTIME、LIBRARY、FRAMEWORK 和通用参数用于确定这些依赖项安装的属性（DESTINATION、COMPONENT 等）。

    RUNTIME_DEPENDENCIES 在语义上等价于以下一对调用：
    ```cmake
    install(TARGETS ... RUNTIME_DEPENDENCY_SET <set-name>)
    install(RUNTIME_DEPENDENCY_SET <set-name> args...)
    ```
    其中 \<set-name> 将是随机生成的集合名称。 args... 可能包含 install(RUNTIME_DEPENDENCY_SET) 命令支持的以下任何关键字：
    - DIRECTORIES
    - PRE_INCLUDE_REGEXES
    - PRE_EXCLUDE_REGEXES
    - POST_INCLUDE_REGEXES
    - POST_EXCLUDE_REGEXES
    - POST_INCLUDE_FILES
    - POST_EXCLUDE_FILES
    RUNTIME_DEPENDENCIES 和 RUNTIME_DEPENDENCY_SET 时彼此相互排斥的。

可以在对该命令的 TARGETS 形式的一次调用中指定一组或多组属性。 一个目标可以多次安装到不同的位置。 考虑假设目标 myExe、mySharedLib 和 myStaticLib。 代码：
```cmake
install(TARGETS myExe mySharedLib myStaticLib
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib/static)
install(TARGETS mySharedLib DESTINATION /some/full/path)
```
将 myExe 安装到 \<prefix>/bin 并将 myStaticLib 安装到 \<prefix>/lib/static。 在非 DLL 平台上，mySharedLib 将安装到 \<prefix>/lib 和 /some/full/path。 在 DLL 平台上，mySharedLib DLL 将安装到 \<prefix>/bin 和 /some/full/path，其导入库将安装到 \<prefix>/lib/static 和 /some/full/path。
接口库可能会列在要安装的目标中。 他们不安装工件，但将包含在相关的 EXPORT 中。 如果列出了对象库但没有为其对象文件指定目标，则它们将被导出为接口库。 这足以满足在其实现中链接到对象库的其他目标的传递使用要求。
安装一个将 EXCLUDE_FROM_ALL 目标属性设置为 TRUE 的目标具有未定义的行为。 
3.3 版中的新功能：作为 DESTINATION 参数给出的安装目标可以使用语法为 $<...> 的“生成器表达式”。 有关可用表达式，请参阅 cmake-generator-expressions(7) 手册。
3.13 版新功能： install(TARGETS) 可以安装在其他目录中创建的目标。 使用此类跨目录安装规则时，从子目录运行 make install （或类似的）将不能保证来自其他目录的目标是最新的。 您可以使用 target_link_libraries() 或 add_dependencies() 来确保在运行特定于子目录的安装规则之前构建此类目录外目标。

# 安装导入的运行时工件
3.21版本的新功能：
```cmake
install(IMPORTED_RUNTIME_ARTIFACTS targets...
        [RUNTIME_DEPENDENCY_SET <set-name>]
        [[LIBRARY|RUNTIME|FRAMEWORK|BUNDLE]
         [DESTINATION <dir>]
         [PERMISSIONS permissions...]
         [CONFIGURATIONS [Debug|Release|...]]
         [COMPONENT <component>]
         [OPTIONAL] [EXCLUDE_FROM_ALL]
        ] [...]
        )
```
IMPORTED_RUNTIME_ARTIFACTS 表单指定了安装导入目标的运行时工件的规则。 如果项目想要在其安装中捆绑外部可执行文件或模块，则可以这样做。 LIBRARY、RUNTIME、FRAMEWORK 和 BUNDLE 参数的语义与它们在 TARGETS 模式下的语义相同。 仅安装导入目标的运行时工件（FRAMEWORK 库、MACOSX_BUNDLE 可执行文件和 BUNDLE CFBundle 除外）。例如，与 DLL 关联的头文件和导入库未安装。 对于 FRAMEWORK 库、MACOSX_BUNDLE 可执行文件和 BUNDLE CFBundle，将安装整个目录。

RUNTIME_DEPENDENCY_SET 选项导致导入的可执行文件、共享库和模块库目标的运行时工件被添加到 \<set-name> 运行时依赖集。 然后可以使用 install(RUNTIME_DEPENDENCY_SET) 命令安装此集。

# 安装文件
```cmake
install(<FILES|PROGRAMS> files...
        TYPE <type> | DESTINATION <dir>
        [PERMISSIONS permissions...]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>]
        [RENAME <name>] [OPTIONAL] [EXCLUDE_FROM_ALL])
```
FILES 表单指定了为项目安装文件的规则。 以相对路径给出的文件名相对于当前源目录进行解释。 如果没有给出 PERMISSIONS 参数，则默认情况下通过这种形式安装的文件被授予 OWNER_WRITE、OWNER_READ、GROUP_READ 和 WORLD_READ 权限。
PROGRAMS 表单与 FILES 表单相同，只是安装文件的默认权限还包括 OWNER_EXECUTE、GROUP_EXECUTE 和 WORLD_EXECUTE。 此表单旨在安装非目标程序，例如 shell 脚本。 使用 TARGETS 表单来安装项目中构建的目标。
files... 给到 FILES 或 PROGRAMS 的可能使用语法为 $<...> 的“生成器表达式”。 有关可用表达式，请参阅 cmake-generator-expressions(7) 手册。 但是，如果任何项目以生成器表达式开头，则它必须评估为完整路径。
必须提供 TYPE 或 DESTINATION，但不能同时提供。 TYPE 参数指定正在安装的文件的通用文件类型。 然后将通过从 GNUInstallDirs 获取相应的变量来自动设置目标，或者如果未定义该变量，则使用内置的默认值。 有关支持的文件类型及其对应的变量和内置默认值，请参见下表。 如果项目希望明确定义安装目标，则可以提供 DESTINATION 参数而不是文件类型。
|TYPE|GNUInstallDirs | 内建默认值|
|:-|:-|:-|
|BIN|${CMAKE_INSTALL_BINDIR}|bin|
|SBIN|${CMAKE_INSTALL_SBINDIR}|sbin|
|LIB|${CMAKE_INSTALL_LIBDIR}|lib|
|INCLUDE|${CMAKE_INSTALL_INCLUDEDIR}|include|
|SYSCONF|${CMAKE_INSTALL_SYSCONFDIR}|etc|
|SHAREDSTATE|${CMAKE_INSTALL_SHARESTATEDIR}|com|
|LOCALSTATE|${CMAKE_INSTALL_LOCALSTATEDIR}|var|
|RUNSTATE|${CMAKE_INSTALL_RUNSTATEDIR}|<LOCALSTATE dir>/run|
|DATA|${CMAKE_INSTALL_DATADIR}|<DATAROOT dir>|
|INFO|${CMAKE_INSTALL_INFODIR}|<DATAROOT dir>/info|
|LOCALE|${CMAKE_INSTALL_LOCALEDIR}|<DATAROOT dir>/locale|
|MAN|${CMAKE_INSTALL_MANDIR}|<DATAROOT dir>/man|
|DOC|${CMAKE_INSTALL_DOCDIR}|<DATAROOT dir>/doc|
希望遵循将标头安装到项目特定子目录中的常见做法的项目将需要提供目标而不是依赖于上述内容。
请注意，某些类型的内置默认值使用 DATAROOT 目录作为前缀。 DATAROOT 前缀的计算方式与类型类似，其中 CMAKE_INSTALL_DATAROOTDIR 作为变量，share 作为内置默认值。 不能将 DATAROOT 用作 TYPE 参数； 请改用 DATA。
为了使包符合分发文件系统布局策略，如果项目必须指定 DESTINATION，建议它们使用以适当的 GNUInstallDirs 变量开头的路径。 这允许包维护者通过设置适当的缓存变量来控制安装目标。 以下示例显示了在将标头安装到特定于项目的子目录时如何遵循此建议：
```cmake
include(GNUInstallDirs)
install(FILES mylib.h
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/myproj
)
```
3.4 版中的新功能：作为 DESTINATION 参数给出的安装目标可以使用语法为 $<...> 的“生成器表达式”。 有关可用表达式，请参阅 cmake-generator-expressions(7) 手册。

3.20 版中的新功能：作为 RENAME 参数给出的安装重命名可以使用语法为 $<...> 的“生成器表达式”。 有关可用表达式，请参阅 cmake-generator-expressions(7) 手册。

# 安装目录
```cmake
install(DIRECTORY dirs...
        TYPE <type> | DESTINATION <dir>
        [FILE_PERMISSIONS permissions...]
        [DIRECTORY_PERMISSIONS permissions...]
        [USE_SOURCE_PERMISSIONS] [OPTIONAL] [MESSAGE_NEVER]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>] [EXCLUDE_FROM_ALL]
        [FILES_MATCHING]
        [[PATTERN <pattern> | REGEX <regex>]
         [EXCLUDE] [PERMISSIONS permissions...]] [...])
```
DIRECTORY 表单将一个或多个目录的内容安装到给定的目的地。目录结构被逐字复制到目的地。每个目录名称的最后一个组件都附加到目标目录，但可以使用尾部斜杠来避免这种情况，因为它使最后一个组件为空。作为相对路径给出的目录名称是相对于当前源目录解释的。如果没有给出输入目录名称，则将创建目标目录，但不会将任何内容安装到其中。 FILE_PERMISSIONS 和 DIRECTORY_PERMISSIONS 选项指定授予目标文件和目录的权限。如果指定了 USE_SOURCE_PERMISSIONS 而未指定 FILE_PERMISSIONS，则将从源目录结构复制文件权限。如果没有指定权限，文件将被赋予命令的 FILES 形式中指定的默认权限，目录将被赋予命令的 PROGRAMS 形式中指定的默认权限。
3.1 版中的新增功能：MESSAGE_NEVER 选项禁用文件安装状态输出。
可以使用 PATTERN 或 REGEX 选项以精细的粒度控制目录的安装。 这些“匹配”选项指定一个通配模式或正则表达式来匹配输入目录中遇到的目录或文件。 它们可用于将某些选项（见下文）应用于遇到的文件和目录的子集。 每个输入文件或目录的完整路径（带有正斜杠）与表达式匹配。 PATTERN 将只匹配完整的文件名：与模式匹配的完整路径部分必须出现在文件名的末尾并以斜杠开头。 REGEX 将匹配完整路径的任何部分，但它可以使用 / 和 $ 来模拟 PATTERN 行为。 默认情况下，无论是否匹配，都会安装所有文件和目录。 FILES_MATCHING 选项可以在第一个匹配选项之前给出，以禁用任何表达式不匹配的文件（但不是目录）的安装。 例如，代码:
```cmake
install(DIRECTORY src/ DESTINATION include/myproj
        FILES_MATCHING PATTERN "*.h")
```
将从源码树中提取并安装头文件。
某些选项可能遵循字符串 (REGEX) 中所述的 PATTERN 或 REGEX 表达式，并且仅适用于匹配它们的文件或目录。 EXCLUDE 选项将跳过匹配的文件或目录。 PERMISSIONS 选项覆盖匹配文件或目录的权限设置。 例如代码
```cmake
install(DIRECTORY icons scripts/ DESTINATION share/myporj
        PATTERN "CVS" EXCLUDE
        PATTERN "scripts/*"
        PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ
            GROUP_EXECUTE GROUP_READ)
```
将 icons 文件夹安装到 share/myproj/icons，scripts 文件夹安装到 share/myproj。icon将获得默认文件权限，脚本将获得特定权限，并且任何 CVS 目录都将被排除。

# 自定义安装逻辑
```cmake
install([[SCRIPT <file>] [CODE <code>]]
        [ALL_COMPONENTS | COMPONENT <component>]
        [EXCLUDE_FROM_ALL] [...])
```
SCRIPT 表单将在安装期间调用给定的 CMake 脚本文件。 如果脚本文件名是相对路径，它将根据当前源目录进行解释。 CODE 表单将在安装期间调用给定的 CMake 代码。 代码被指定为双引号字符串内的单个参数。 例如，代码
```cmake
install(CODE "MESSAGE(\"Sample install message.\")")
```
将在安装期间打印一条消息。
3.21 版新增功能：当给出 ALL_COMPONENTS 选项时，将为特定组件安装的每个组件执行自定义安装脚本代码。 此选项与 COMPONENT 选项互斥。
3.14 版中的新功能：<file> 或 <code> 可以使用语法为 $<...> 的“生成器表达式”（对于 <file>，这是指它们在文件名中的使用，而不是文件的内容 ）。 有关可用表达式，请参阅 cmake-generator-expressions(7) 手册。

# 安装输出
```cmake
install(EXPORT <export-name> DESTINATION <dir>
        [NAMESPACE <namespace>] [[FILE <name>.cmake]|
        [PERMISSIONS permissions...]
        [CONFIGURATIONS [Debug|Release|...]]
        [EXPORT_LINK_INTERFACE_LIBRARIES]
        [COMPONENT <component>]
        [EXCLUDE_FROM_ALL])
install(EXPORT_ANDROID_MK <export-name> DESTINATION <dir> [...])
```
EXPORT 表单生成并安装一个 CMake 文件，其中包含将目标从安装树导入另一个项目的代码。使用上面记录的 install(TARGETS) 签名的 EXPORT 选项，目标安装与导出 \<export-name> 相关联。 NAMESPACE 选项将在目标名称写入导入文件时将其添加到目标名称之前。默认情况下，生成的文件将被称为 \<export-name>.cmake，但 FILE 选项可用于指定不同的名称。给 FILE 选项的值必须是带有 .cmake 扩展名的文件名。如果给出了 CONFIGURATIONS 选项，则仅当安装了指定配置之一时才会安装该文件。此外，生成的导入文件将仅引用匹配的目标配置。 EXPORT_LINK_INTERFACE_LIBRARIES 关键字（如果存在）会导致属性的内容匹配 (IMPORTED_)?LINK_INTERFACE_LIBRARIES(_\<CONFIG>)?要导出，当策略 CMP0022 为 NEW 时。

注意：
已安装的 <export-name>.cmake 文件可能附带额外的每个配置 <export-name>-*.cmake 文件，以通过 globbing 加载。 不要将与包名称相同的导出名称与安装 <package-name>-config.cmake 文件结合使用，否则后者可能会被 glob 错误匹配并加载。

当给出 COMPONENT 选项时，列出的 \<component> 隐式依赖于导出集中提到的所有组件。导出的 \<name>.cmake 文件将要求每个导出的组件都存在，以便正确构建相关项目。例如，一个项目可以定义组件运行时和开发，共享库进入运行时组件，静态库和头文件进入开发组件。导出集通常也是 Development 组件的一部分，但它会从 Runtime 和 Development 组件中导出目标。因此，如果安装了 Development 组件，则需要安装 Runtime 组件，反之则不然。如果在没有运行时组件的情况下安装了开发组件，则尝试链接它的依赖项目将产生构建错误。包管理器（例如 APT 和 RPM）通常通过将运行时组件列为包元数据中开发组件的依赖项来处理此问题，确保在存在标头和 CMake 导出文件时始终安装库。
3.7 版新功能：除了 cmake 语言文件，EXPORT_ANDROID_MK 模式可能用于指定导出到 android ndk 构建系统。 此模式接受与正常导出模式相同的选项。 Android NDK 支持使用预建库，包括静态库和共享库。 这允许 cmake 构建项目的库，并使它们可用于具有传递依赖项的 ndk 构建系统，包括使用这些库所需的标志和定义。
EXPORT 表单有助于帮助外部项目使用当前项目构建和安装的目标。 例如，代码
```cmake
install(TARGETS myexe EXPORT myproj DESTINATION bin)
install(EXPORT myproj NAMESPACE mp_ DESTINATION lib/myproj)
install(EXPORT_ANDROID_MK myproj DFESTINATION  share/ndk-modules)
```
会将可执行文件 myexe 安装到 <prefix>/bin ，并将代码导入文件 <prefix>/lib/myproj/myproj.cmake 和 <prefix>/share/ndk-modules/Android.mk。 外部项目可以使用 include 命令加载此文件，并使用导入的目标名称 mp_myexe 从安装树引用 myexe 可执行文件，就好像目标是在自己的树中构建的一样。
注意：
此命令取代 install_targets() 命令以及 PRE_INSTALL_SCRIPT 和 POST_INSTALL_SCRIPT 目标属性。 它还替换了 install_files() 和 install_programs() 命令的 FILES 形式。 这些安装规则相对于 install_targets()、install_files() 和 install_programs() 命令生成的规则的处理顺序未定义。

# 安装运行时依赖
3.21版本新功能。
```cmake
install(RUNTIME_DEPENDENCY_SET <set-name>
        [[LIBRARY|RUNTIME|FRAMEWORK]
         [DESTINATION <dir>]
         [PERMISSIONS permissions...]
         [CONFIGURATIONS [Debug|Release|...]]
         [COMPONENT <component>]
         [NAMELINK_COMPONENT <component>]
         [OPTIONAL] [EXCLUDE_FROM_ALL]
        ] [...]
        [PRE_INCLUDE_REGEXES regexes...]
        [PRE_EXCLUDE_REGEXES regexes...]
        [POST_INCLUDE_REGEXES regexes...]
        [POST_EXCLUDE_REGEXES regexes...]
        [POST_INCLUDE_FILES files...]
        [POST_EXCLUDE_FILES files...]
        [DIRECTORIES directories...]
        )
```
安装以前由一个或多个 install(TARGETS) 或 install(IMPORTED_RUNTIME_ARTIFACTS) 命令创建的运行时依赖项集。 属于运行时依赖集的目标的依赖关系安装在 DLL 平台上的 RUNTIME 目标和组件中，以及非 DLL 平台上的 LIBRARY 目标和组件中。 macOS 框架安装在 FRAMEWORK 目标和组件中。 在构建树中构建的目标永远不会作为运行时依赖项安装，它们自己的依赖项也不会安装，除非目标本身是使用 install(TARGETS) 安装的。

生成的安装脚本在构建树文件上调用 file(GET_RUNTIME_DEPENDENCIES) 来计算运行时依赖项。 构建树可执行文件作为 EXECUTABLES 参数传递，构建树共享库作为 LIBRARIES 参数传递，构建树模块作为 MODULES 参数传递。 在 macOS 上，如果其中一个可执行文件是 MACOSX_BUNDLE，则该可执行文件将作为 BUNDLE_EXECUTABLE 参数传递。 最多一个这样的捆绑可执行文件可能位于 macOS 上的运行时依赖集中。 MACOSX_BUNDLE 属性对其他平台没有影响。 请注意，file(GET_RUNTIME_DEPENDENCIES) 仅支持收集 Windows、Linux 和 macOS 平台的运行时依赖项，因此 install(RUNTIME_DEPENDENCY_SET) 具有相同的限制。

以下子参数作为相应参数转发给 file(GET_RUNTIME_DEPENDENCIES)（对于那些提供非空目录、正则表达式或文件列表的参数）。 它们都支持生成器表达式。
- DIRECTORIES \<directories>
- 
- PRE_INCLUDE_REGEXES \<regexes>
- 
- PRE_EXCLUDE_REGEXES \<regexes>
- 
- POST_INCLUDE_REGEXES \<regexes>
- 
- POST_EXCLUDE_REGEXES \<regexes>
- 
- POST_INCLUDE_FILES \<files>
- 
- POST_EXCLUDE_FILES \<files>