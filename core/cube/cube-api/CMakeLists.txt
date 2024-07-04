# Copyright (c) 2016 PaddlePaddle Authors. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License

protobuf_generate_cpp(PROTO_SRC PROTO_HEADER idl/cube.proto idl/control.proto)
# include PROTO_HEADER
include_directories(${CMAKE_CURRENT_BINARY_DIR}/core)

find_package(OpenSSL REQUIRED) 

message(STATUS "ssl:" ${OPENSSL_SSL_LIBRARY})
message(STATUS "crypto:" ${OPENSSL_CRYPTO_LIBRARY})
if (OPENSSL_FOUND)
    include_directories (${OPENSSL_INCLUDE_DIR})
endif()

if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    include(CheckFunctionExists)
    CHECK_FUNCTION_EXISTS(clock_gettime HAVE_CLOCK_GETTIME)
    if(NOT HAVE_CLOCK_GETTIME)
        set(DEFINE_CLOCK_GETTIME "-DNO_CLOCK_GETTIME_IN_MAC")
    endif()
    
endif()
set(CMAKE_OSX_ARCHITECTURES arm64)

set(CMAKE_CXX_FLAGS "${CMAKE_CPP_FLAGS} -DNDEBUG -O2 -D__const__= -pipe -W -Wall -Wno-unused-parameter -fPIC -fno-omit-frame-pointer")

if(CMAKE_VERSION VERSION_LESS "3.1.3")
    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    endif()
    if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    endif()
else()
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

add_executable(cube-cli src/cube_cli.cpp src/cube_api.cpp src/meta.cpp
    ${PROTO_SRC} ${PROTO_HEADER})

add_library(cube-api STATIC src/cube_api.cpp src/meta.cpp
    ${PROTO_SRC} ${PROTO_HEADER})

set(DYNAMIC_LIB
    gflags
    protobuf
    leveldb
    )

if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    set(DYNAMIC_LIB ${DYNAMIC_LIB}
        "-framework CoreFoundation"
        "-framework CoreGraphics"
        "-framework CoreData"
        "-framework CoreText"
        "-framework Security"
        "-framework Foundation"
        "-Wl,-U,_MallocExtension_ReleaseFreeMemory"
        "-Wl,-U,_ProfilerStart"
        "-Wl,-U,_ProfilerStop")
endif()

target_link_libraries(cube-cli brpc ${DYNAMIC_LIB} brpc -lpthread -ldl -lz ${OPENSSL_SSL_LIBRARY} ${OPENSSL_CRYPTO_LIBRARY})
target_link_libraries(cube-api ${DYNAMIC_LIB} brpc -lpthread -ldl -lz)
ADD_DEPENDENCIES(cube-cli ssl crypto)

# install
install(TARGETS cube-api
        ARCHIVE DESTINATION ${PADDLE_SERVING_INSTALL_DIR}/lib
        )
install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/include
        DESTINATION ${PADDLE_SERVING_INSTALL_DIR}/include/cube-api)

FILE(GLOB inc ${CMAKE_CURRENT_BINARY_DIR}/*.pb.h)
install(FILES ${inc}
        DESTINATION ${PADDLE_SERVING_INSTALL_DIR}/include/cube-api)
