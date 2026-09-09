# 使用CMake组织你的项目结构

# 磐石框架：古法编程与现代C++高性能服务器

**主讲人：花间客 | 十年一线游戏后台开发**

***

## 我是谁

**花间客**

* 十年一线游戏后台开发经验
* 参与开发两款日活超百万的MMORPG

**本系列要做什么**

* 手把手教你构建工业级C++游戏服务器
* UE客户端开发
* 最终目标：独立游戏，上线Steam

***

## 课程介绍

本专栏讲解如何使用C++20标准实现一套多Reactor+多线程+多协程的高性能服务器内核。

**上节内容**

* Linux 服务器配置方式已就绪
* pans 空项目已创建

**今日任务**

1. 搭建工程目录结构
2. 编写 CMakeLists.txt

***

## 目录结构

**五大目录**

```
pans/
├── docs/     # 文档
├── pans/     # 库核心代码
├── tests/    # 测试程序
├── tools/    # 辅助工具
└── cmake/    # CMake 公共代码
```

***

## docs 目录

**存放文档性内容**

* 维护 README 等说明文档
* 重点：维护**中文版 Readme**

***

## 代码规范约定

```
常量：全部大写
类名：首字母大写驼峰
非静态成员函数：首字母小写驼峰
静态成员函数：首字母大写驼峰
非静态成员变量：以m_开头，首字母小写驼峰
静态成员变量：以s_开头，首字母小写驼峰
全局函数：首字母大写驼峰
函数参数：小写下划线隔开
局部变量：小写下划线隔开
```

***

## pans 目录

**库的核心代码**

* 维持库功能完整性的**必需代码**

***

## tests 目录

**测试程序**

* 验证代码的**正确性与健壮性**
* 与核心代码**分离存放**

***

## tools 目录

**非核心但重要的工具**

* 承担重要辅助功能
* 例：Excel 导表工具

***

## cmake 目录

**CMake 公共代码**

* 存放函数等公共 CMake 代码
* 独立存放，供 CMakeLists.txt 包含

***

## CMake 是什么

**跨平台开源构建工具**

* 本身**不编译**，只生成各平台构建文件
* 生成目标：`Makefile` / `sln` / `Ninja`
* 官网：cmake.com.cn

***

## CMakeLists.txt 层级结构

**树形结构**

```
pans/
├── CMakeLists.txt      # 顶层
├── pans/
│   └── CMakeLists.txt  # 子级
├── tests/
│   └── CMakeLists.txt
└── tools/
└── CMakeLists.txt
```

* 顶层定义**全局变量**，子级**继承**
* 今日重点：**顶层 CMakeLists.txt**

***

## 版本要求

```cmake
cmake_minimum_required(VERSION 3.31)
```

* 指定最低 CMake 版本
* 3.31：**稳妥、功能现代、兼容性好**

***

## 指定编译器

* `设置`C`/C++文件编译器，使用C`ACHE FORCE强制覆盖缓存`；多套环境可锁定版本（g++ 12.2 / 14.2 / 15.1），确保使用指定版本而非系统默认`

***

## 编译器标准

```cmake
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)
```

* 前两行设置使用C++20标准，并强制编译器支持，否则报错；第三行允许编译器扩展，可能降低可移植性，问题不大先开着

***

## 定义项目

```cmake
project(pans
VERSION 0.1.0
LANGUAGES CXX C)
```

**project() 的作用**

* 定义项目名：`pans`
* 指定版本：`0.1.0`
* 声明语言：`CXX`（C++）与 `C`

**配套分发支持**

```cmake
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)
```

* `GNUInstallDirs`：引入标准安装目录变量，符合GNU惯例，避免硬编码，提升可移植性
* `CMakePackageConfigHelpers`：生成包配置文件，支持外部项目find\_package(pans)引用

***

## 编译开关

```cmake
option(PANS_BUILD_EXAMPLES "Build examples" ON)
option(PANS_BUILD_TESTS    "Build tests"    ON)
option(PANS_BUILD_TOOLS    "Build tools"    ON)
```

* 控制tests、tools等模块是否参与编译；把pans库提供给别人时通常不需要编译这几个模块，只编译核心模块可加速编译

***

## 编译模式

* 默认：**Debug**
* 下拉可选：
  * `Debug`
  * `Release`
  * `RelWithDebInfo`
  * `MinSizeRel`

***

## 并行编译线程数

**线程数 = CPU核心数 × 1.5**

* 一个编译任务需要1\~2G内存，取均值1.5G；4核8G若开8线程，容易内存吃满、频繁换出甚至OOM；若内存比1:3或1:4，可把1改为2，即开2N线程

***

## 编译选项与链接选项

**创建接口库**

```cmake
add_library(pans_options INTERFACE)
```

* 不产生任何编译产物，仅用于传递编译选项、链接选项、宏定义和依赖关系，就像一个配置包

***

## 通用编译选项

```cmake
target_compile_options(pans_options INTERFACE
-Wall -Wextra -Wpedantic
-fno-strict-aliasing)
```

* 关闭严格别名规则后，不同类型的指针可以相互赋值，如把float指针指向的内容转成int处理，序列化网络传输数据时有用，不加可能编译不通过

***

## 位置无关代码

```cmake
target_compile_options(pans_options INTERFACE
-fPIC)
```

* 编译模块时用相对地址替换绝对地址，链接时正确解析符号；会增加库的体积及运行时开销，详见《程序员的自我修养》

***

## 链接选项

```cmake
target_link_options(pans_options INTERFACE -rdynamic)
```

* 导出所有符号到动态符号表，栈回溯和dlopen查找符号时有用，pans库要用到

***

## 调试信息选项

* `-g`：生成默认调试信息（等价-g2），让gdb看到源码行号、变量名、函数名、调用栈，不等于Debug构建
* `-g3`：额外包含宏定义等信息，调试宏、日志宏、复杂条件编译时更有用，目标文件更大
* `-ggdb`：生成适合GDB使用的调试信息格式，可能带GDB扩展
* `-fno-omit-frame-pointer`：保留栈帧指针，-O1及以上默认启用-fomit-frame-pointer，省略frame pointer可少生成保存/恢复指令并多释放一个寄存器，Fedora评估编译Linux kernel慢约2.4%、Blender渲染慢约2%，Meta默认启用未看到显著性能影响

***

## 覆盖率选项

* 提供一个开关，表示是否打开gcc的代码覆盖率选项

***

## 添加子目录

* pans`是一定要添加的子目录；tests/tools是根据变量控制是否添加；每一个子模块里面都要有CMakeLists.txt，哪怕文件是空的，否则添加子目录就会出错；把pans库提供给别人时，别人通常不需要编译这几个模块，只编译核心模块可加速编译`

***

## pans 子模块

**① 收集源文件**

```cmake
file(GLOB_RECURSE PANS_SOURCES CONFIGURE_DEPENDS *.cpp *.cc *.c)
file(GLOB_RECURSE PANS_HEADERS CONFIGURE_DEPENDS *.h *.hpp)
```

* 递归收集`目录下所有`源`文件和头`文件

**② 创建静态库**

```cmake
add_library(pans STATIC ${PANS_SOURCES} ${PANS_HEADERS})
```

* 通过STATIC指定生成静态库

**③ 头文件路径分层**

```cmake
target_include_directories(pans PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include> $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}> PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src)
```

* PUBLIC`公开的头文件，链接pans库的地方可自动获取搜索目录；不想公开的放PRIVATE；公开太多会暴露内部实现、增加使用者心智负担`

**④ 继承编译选项**

```cmake
target_link_libraries(pans PUBLIC pans_options)
```

* 继承全局编译选项和链接选项

**⑤ 主库零警告**

```cmake
target_compile_options(pans PRIVATE -Werror)
```

* `严格要求主库一个警告都不要有，否则编译失败`

**⑥ 安装与分发**

```cmake
install(TARGETS pans EXPORT pansTargets ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES ${PANS_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/pans)
```

* 将编译好的库复制到系统标准目录，其它项目可通过find\_package找到使用

***

## cmake 测试

* 会报错，原因是搜集不到目标文件；正常情况库里一定有源文件不会出现；创建空源文件pans.not.empty.cc消除问题，再执行cmake即可通过

***

## 总结

**本节完成**

* ✅ 项目结构搭建完成
* ✅ CMakeLists.txt 逐段拆解

**学习建议**

1. **敲一遍**：亲手输入，加深记忆
2. **问 AI 理解**：逐行追问每个命令的作用
3. **边用边学**：在实战中掌握，而非死记硬背

***

## 下期预告

* 封装断言assert，让assert提供更多信息，写一个测试用例，补全tests目录CMakeLists.txt逻辑

***

## 加入我们，一起成长

### QQ交流群

* **群号**：940661137
* **暗号**：**坚如磐石**
* **入群验证**：发送暗号，管理员审核

### 群内资源

* **源码分享**：磐石框架完整代码
* **问题解答**：技术难题指导
* **学习资料**：精选书籍、文章、工具
* **项目实践**：组队完成实战项目

### 互动方式

* **技术讨论**：架构设计、性能优化
* **代码Review**：互相学习，共同进步
* **经验分享**：踩坑记录，避坑指南

***

> 加群交流：QQ群 940661137（暗号：坚如磐石）| 脚本文件可在群资料下载 | 点赞、评论、关注支持，谢谢！
