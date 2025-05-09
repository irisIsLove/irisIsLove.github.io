---
title: CMake 中使用 grpc ( 使用 vcpkg 安装 grpc )
date: 2025-05-09 16:38:47
categories:
- 技术
tags:
- grpc
---

# CMake 中使用 grpc ( 使用 vcpkg 安装 grpc )

用 vcpkg 安装 grpc

```shell
vcpkg install grpc
```

cmake 文件中源码

```cmake
find_package(gRPC CONFIG REQUIRED)  # 查找库

# 查找工具链
get_target_property(PROTOC protobuf::protoc LOCATION)
get_target_property(GRPC_PLUGIN gRPC::grpc_cpp_plugin LOCATION)

set(PROTO_SRCS "")
set(PROTO_HDRS "")
set(GRPC_SRCS "")
set(GRPC_HDRS "")

# 创建生成文件夹
file(MAKE_DIRECTORY messages)
# 查找 proto 源文件
file(GLOB PROTO_FILES "${CMAKE_SOURCE_DIR}/protos/*.proto")

# 遍历源文件
foreach(PROTO_FILE ${PROTO_FILES})
    get_filename_component(PROTO_NAME ${PROTO_FILE} NAME) # 获取文件名 (带扩展)
    get_filename_component(PROTO_NAME_WE ${PROTO_FILE} NAME_WE) # 获取文件名 (不带扩展)

    # 生成 proto 对应的 .cc 文件和 .h 文件
    add_custom_command(
        OUTPUT "${CMAKE_SOURCE_DIR}/messages/${PROTO_NAME_WE}.pb.h" "${CMAKE_SOURCE_DIR}/messages/${PROTO_NAME_WE}.pb.cc"
        COMMAND ${PROTOC}
        ARGS
        -I${CMAKE_SOURCE_DIR}/protos
        --cpp_out=${CMAKE_SOURCE_DIR}/messages
        ${PROTO_NAME}
        DEPENDS ${PROTO_FILE}
        COMMENT "Generating proroto sources for ${PROTO_NAME}"
    )
    # 装入对应的 proto 列表中
    list(APPEND PROTO_SRCS "${CMAKE_SOURCE_DIR}/messages/${PROTO_NAME_WE}.pb.cc")
    list(APPEND PROTO_HDRS "${CMAKE_SOURCE_DIR}/messages/${PROTO_NAME_WE}.pb.h")

    # 生成 gRPC 对应的 .cc 文件和 .h 文件
    add_custom_command(
        OUTPUT "${CMAKE_SOURCE_DIR}/messages/${PROTO_NAME_WE}.grpc.pb.h" "${CMAKE_SOURCE_DIR}/messages/${PROTO_NAME_WE}.grpc.pb.cc"
        COMMAND ${PROTOC}
        ARGS
        -I${CMAKE_CURRENT_SOURCE_DIR}/protos
        --cpp_out=${CMAKE_SOURCE_DIR}/messages
        --plugin=protoc-gen-grpc=${GRPC_PLUGIN}
        --grpc_out=${CMAKE_SOURCE_DIR}/messages
        ${PROTO_NAME}
        DEPENDS ${PROTO_FILE}
        COMMENT "Generating gRPC sources for ${PROTO_NAME}"
    )

    # 装入对应的 grpc 列表中
    list(APPEND GRPC_SRCS "${CMAKE_SOURCE_DIR}/messages/${PROTO_NAME_WE}.grpc.pb.cc")
    list(APPEND GRPC_HDRS "${CMAKE_SOURCE_DIR}/messages/${PROTO_NAME_WE}.grpc.pb.h")
endforeach()
```
