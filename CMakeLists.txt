 # Copyright (c) 2023 General Motors GTO LLC
 #
 # Licensed to the Apache Software Foundation (ASF) under one
 # or more contributor license agreements.  See the NOTICE file
 # distributed with this work for additional information
 # regarding copyright ownership.  The ASF licenses this file
 # to you under the Apache License, Version 2.0 (the
 # "License"); you may not use this file except in compliance
 # with the License.  You may obtain a copy of the License at
 #
 #   http://www.apache.org/licenses/LICENSE-2.0
 #
 # Unless required by applicable law or agreed to in writing,
 # software distributed under the License is distributed on an
 # "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 # KIND, either express or implied.  See the License for the
 # specific language governing permissions and limitations
 # under the License.

cmake_minimum_required(VERSION 3.10.0)
set(CMAKE_CXX_STANDARD 17)
project(up-cpp  LANGUAGES CXX)

find_package(Protobuf REQUIRED)
find_package(spdlog REQUIRED)
find_package(fmt REQUIRED)
set(PROTOBUF_PROTOC "${Protobuf_PROTOC_EXECUTABLE}")

if (IS_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/up-core-api/uprotocol/)
  set(PROTOBUF_INPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/up-core-api/uprotocol/")
elseif (IS_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../up-core-api/uprotocol/)
  set(PROTOBUF_INPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../up-core-api/uprotocol/")
elseif(UP_CORE_API_ROOT_DIR)
  set(PROTOBUF_INPUT_DIRECTORY "${UP_CORE_API_ROOT_DIR}/uprotocol/")
else()
  message(FATAL_ERROR "Could not find up-core-api. Please, set UP_CORE_API_ROOT_DIR to the root directory of up-core-api.")
endif()

set(PROTOBUF_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/include/up-core-api/")

file(GLOB_RECURSE PROTOBUF_DEFINITION_FILES "${PROTOBUF_INPUT_DIRECTORY}/*.proto")
file(MAKE_DIRECTORY ${PROTOBUF_OUTPUT_DIRECTORY})

foreach(file ${PROTOBUF_DEFINITION_FILES})
    set(PROTOBUF_ARGUMENTS "-I=${PROTOBUF_INPUT_DIRECTORY}" "--cpp_out=${PROTOBUF_OUTPUT_DIRECTORY}" "${file}")
    execute_process(
        COMMAND ${PROTOBUF_PROTOC} ${PROTOBUF_ARGUMENTS}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        RESULT_VARIABLE PROTOBUF_RESULT
        OUTPUT_VARIABLE PROTOBUF_OUTPUT_VARIABLE)
endforeach()

include_directories(${protobuf_INCLUDE_DIR})
include_directories(${spdlog_INCLUDE_DIR})
include_directories(${fmt_INCLUDE_DIR})
include_directories(${CMAKE_CURRENT_BINARY_DIR}/include)
include_directories(${CMAKE_CURRENT_BINARY_DIR}/include/up-core-api)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

file(GLOB_RECURSE 	SRC_PROTO_CORE_API 	"${CMAKE_CURRENT_BINARY_DIR}/include/up-core-api/*.cc")
file(GLOB_RECURSE 	SRC_FILES 	      	"${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")

set(UPCPP_SRC
    ${SRC_FILES}
    ${SRC_PROTO_CORE_API})

add_definitions(-DFMT_HEADER_ONLY)
add_definitions(-DSPDLOG_FMT_EXTERNAL)

add_library(up-cpplib OBJECT ${UPCPP_SRC})

set_property(TARGET up-cpplib PROPERTY POSITION_INDEPENDENT_CODE 1)

add_library(up-cpp SHARED $<TARGET_OBJECTS:up-cpplib>)
add_library(up-cpp_static STATIC $<TARGET_OBJECTS:up-cpplib>)

set_target_properties(up-cpp_static PROPERTIES OUTPUT_NAME up-cpp)

target_link_libraries(up-cpp_static PRIVATE protobuf::protobuf spdlog::spdlog)

target_link_libraries(up-cpp PRIVATE protobuf::protobuf spdlog::spdlog)

add_subdirectory(test)

INSTALL(TARGETS up-cpp)
INSTALL(TARGETS up-cpp_static)
INSTALL(DIRECTORY include DESTINATION .)
INSTALL(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/include/up-core-api DESTINATION include)
