---
title: Vcpkg 在 Manifest 模式下防止 CMake 重复加载
date: 2025-10-15 19:38:17
categories:
- 技术
tags:
- CMake
---

# Vcpkg 在 Manifest 模式下防止 CMake 重复加载

在使用vcpkg的manifest模式下，每次加载cmake都会出发vcpkg的install命令，这导致笔者在编写代码的时候会很烦躁（不想使用集成模式是因为个人认为很多库基本只用一次两次了，没必要集成在系统里，我想的是要用的时候在装就是了）

好了，言归正传，每次都会出发的原因是CMake中有个cache变量为 `VCPKG_MANIFEST_INSTALL` 在使用Manifest模式后这个变量的值就为 ON ，每次加载CMake文件时就会检查这个值，所以我需要做的就是让这个值在我不需要刷新的时候为 OFF

下面就是解决方案：
- Vcpkg 的 Manifest 模式是通过`vcpkg.json`和`vcpkg-configuration.json`来控制安装的库
- 所以只要让CMake知道这两个发生文件没有改变就可以设置`VCPKG_MANIFEST_INSTALL`为OFF
- 确保文件是否发生改变可以通过获取文件的哈希值来判断，将上一次的哈希值保存在一个临时文件中就可以了

下面是代码：
```cmake
function(check_file_modified INPUT_FILE OUTPUT_VALUE)
    set(IS_MODIFIED OFF)

    if(EXISTS "${INPUT_FILE}")
        file(SHA256 "${INPUT_FILE}" CURRENT_HASHES)

        if(EXISTS "${INPUT_FILE}.hashes")
            file(READ "${INPUT_FILE}.hashes" PREVIOUS_HASHES)

            if(NOT "${CURRENT_HASHES}" STREQUAL "${PREVIOUS_HASHES}")
                file(WRITE "${INPUT_FILE}.hashes" "${CURRENT_HASHES}")
                set(IS_MODIFIED ON)
            endif()
        else()
            file(WRITE "${INPUT_FILE}.hashes" "${CURRENT_HASHES}")
            set(IS_MODIFIED ON)
        endif()
    endif()

    set(${OUTPUT_VALUE} ${IS_MODIFIED} PARENT_SCOPE)
endfunction()

check_file_modified(${CMAKE_SOURCE_DIR}/vcpkg.json VCPKG_IS_MODIFIED)
check_file_modified(${CMAKE_SOURCE_DIR}/vcpkg-configuration.json VCPKG_CONFIG_IS_MODIFIED)

if(VCPKG_IS_MODIFIED OR VCPKG_CONFIG_IS_MODIFIED)
    message(STATUS "vcpkg configuration changed - enabling VCPKG_MANIFEST_INSTALL")
    set(VCPKG_MANIFEST_INSTALL ON)
else()
    message(STATUS "vcpkg configuration unchanged - disabling VCPKG_MANIFEST_INSTALL")
    set(VCPKG_MANIFEST_INSTALL OFF)
endif()
```