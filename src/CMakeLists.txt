# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# set the project name
project(substrait-cpp)

# Set up Proto
set(proto_directory ${CMAKE_SOURCE_DIR}/third_party/substrait/proto)
set(substrait_proto_directory ${proto_directory}/substrait)
set(PROTO_OUTPUT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/proto/")
file(MAKE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/proto/substrait)
file(GLOB PROTO_FILES ${substrait_proto_directory}/*.proto
        ${substrait_proto_directory}/extensions/*.proto)
foreach(PROTO ${PROTO_FILES})
    file(RELATIVE_PATH REL_PROTO ${substrait_proto_directory} ${PROTO})
    string(REGEX REPLACE "\\.proto" "" PROTO_NAME ${REL_PROTO})
    list(APPEND PROTO_SRCS "${PROTO_OUTPUT_DIR}/substrait/${PROTO_NAME}.pb.cc")
    list(APPEND PROTO_HDRS "${PROTO_OUTPUT_DIR}/substrait/${PROTO_NAME}.pb.h")
endforeach()
set(PROTO_OUTPUT_FILES ${PROTO_HDRS} ${PROTO_SRCS})
set_source_files_properties(${PROTO_OUTPUT_FILES} PROPERTIES GENERATED TRUE)

get_filename_component(PROTO_DIR ${substrait_proto_directory}/, DIRECTORY)

# Generate Substrait hearders
add_custom_command(
    OUTPUT ${PROTO_OUTPUT_FILES}
    COMMAND protoc --proto_path ${proto_directory}/ --cpp_out ${PROTO_OUTPUT_DIR}
    ${PROTO_FILES}
    DEPENDS ${PROTO_DIR}
    COMMENT "Running PROTO compiler"
    VERBATIM)
add_custom_target(substrait_proto ALL DEPENDS ${PROTO_OUTPUT_FILES})
add_dependencies(substrait_proto protobuf::libprotobuf)


set(SRCS
    ${PROTO_SRCS}
    Type.cpp
    Function.cpp
    Extension.cpp
    FunctionMapping.h
    FunctionLookup.cpp
    FunctionSignature.h)

add_library(substrait-cpp ${SRCS})

target_include_directories(substrait-cpp
        PUBLIC ${PROTO_OUTPUT_DIR})
target_link_libraries(substrait-cpp yaml-cpp)

#if (${BUILD_TESTING})
    add_subdirectory(tests)
#endif ()
