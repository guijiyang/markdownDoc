xmake 常用命令：
|快捷键|作用|
|:-|:-|
|xmake g|配置全局选项，如 `xmake g --pkg_searchdirs=F:`，具体可查看帮助`xmake g -h`|
|xmake f|工程配置，如 `xmake f -m debug`|
|xmake f --runtimes=MDd|配置时指定需要链接vs运行时库|
|xmake f --pkg_searchdirs=F:\xmake_packages|配置时指定pkg搜索路径|
|xmake f --policies=package.cmake_generator.ninja:y|配置时指定pkg的cmake generator为ninja，默认在vs平台使用msbuild|