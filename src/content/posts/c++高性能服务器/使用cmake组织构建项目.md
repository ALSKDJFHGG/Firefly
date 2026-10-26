---
title: 使用cmake构建项目
published: 2026-10-26
description: 本篇讲解如何在c++工程中使用cmake构建项目
category: C++
tags:
  - cmake
  - hs
series: 高性能服务器
draft: false
slug: cmake-make-project
---


# 文件目录结构

```bash
.
├── .gitignore				# git 忽略文件
├── CMakeLists.txt          # 顶层构建入口
├── LICENSE                 # 许可证
├── bin/                    # 可执行文件输出目录（构建产物）
├── build/                  # CMake 构建目录（缓存、中间产物）
├── cmake/
│   └── hsOptions.cmake     # 自定义 CMake 模块（编译选项）
├── docs/                   # 文档
├── hs/                     # 核心源码 / 库
│   └── CMakeLists.txt
├── tests/                  # 测试
│   └── CMakeLists.txt
└── tools/                  # 辅助工具
    └── CMakeLists.txt
```

## CMakeLists.txt

> 整个项目的构建入口，只负责全局性的事，不做具体编译
> 
> - 设置全局编译标准、输出路径。
> - 引入 `cmake/` 下的自定义模块。
> - 统一管理依赖
> - 用 `add_subdirectory()` 把各子模块挂起来。

```cmake
cmake_minimum_required(VERSION 3.25)

# 强制指定编译器路径，放在多版本编译器影响
set(CMAKE_C_COMPILER "/usr/bin/gcc" CACHE FILEPATH "C compiler" FORCE)
set(CMAKE_CXX_COMPILER "/usr/bin/g++" CACHE FILEPATH "C++ compiler" FORCE)

# 全局标准设置
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

# clangd 支持
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

project(hs VERSION 0.1.0 LANGUAGES CXX C)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# 构建选项
option(HS_BUILD_EXAMPLES "Build example" ON)
option(HS_BUILD_TESTS "Build tests" ON)
option(HS_BUILD_TOOLS "Build tools" ON)

# 构建类型处理
if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build." FORCE)
endif()
set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS Debug Release RelWithDebInfo MinSizeRel)
message(STATUS "Build Type: ${CMAKE_BUILD_TYPE}")

# 并行编译设置
cmake_host_system_information(RESULT CPU_NUMS QUERY NUMBER_OF_LOGICAL_CORES)
message(STATUS "CPU NUMS on current server is: ${CPU_NUMS}")
math(EXPR JOBS "${CPU_NUMS} + ${CPU_NUMS} / 2")
set(CMAKE_BUILD_PARALLEL_LEVEL ${JOBS} CACHE STRING "并行编译线程数")
message(STATUS "Compiler default work jobs: ${CMAKE_BUILD_PARALLEL_LEVEL}")

# 引入自定义模块
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/hsOptions.cmake)

# 主源码子目录
add_subdirectory(hs)

if (HS_BUILD_TESTS)
	# 开启测试功能
    enable_testing()
    # 测试子目录
    add_subdirectory(tests)
endif()
if (HS_BUILD_TOOLS)
	# 工具集子目录
    add_subdirectory(tools)
endif()
```

