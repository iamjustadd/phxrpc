cmake_minimum_required(VERSION 3.6)

project(phxrpc)

set(CMAKE_CXX_STANDARD 14)

set(PHXRPC_HOME ${CMAKE_CURRENT_LIST_DIR}/..)
set(PHXRPC_BOOST_HOME ${PHXRPC_HOME}/plugin_boost)
set(PHXRPC_DARWIN_HOME ${PHXRPC_HOME}/plugin_darwin)
set(PROTOBUF_HOME ${CMAKE_CURRENT_LIST_DIR}/../third_party/protobuf/)

option(BUILD_SHARED_LIB "Whether building shared library" OFF)
option(BUILD_WITH_BOOST "Whether building lib with boost" OFF)
option(BUILD_WITH_TEST "Building test binary" OFF)

set(LIBRARY_BUILD_TYPE STATIC)
if (BUILD_SHARED_LIB)
    set(LIBRARY_BUILD_TYPE SHARED)
endif()

add_definitions(-D_XOPEN_SOURCE)

include_directories(
    ${CMAKE_CURRENT_LIST_DIR}/..
    ${PROTOBUF_HOME}/include/
    ${PROTOBUF_HOME}/src/
)


set(PB_SRC_LIST
    rpc/phxrpc.pb.cc
)

set(COMMON_SRC_LIST
    comm/assert.cpp
)

set(FILE_SRC_LIST
    file/config.cpp
    file/file_utils.cpp
    file/log_utils.cpp
    file/opt_map.cpp
)

set(HTTP_SRC_LIST
    http/http_client.cpp
    http/http_msg_handler_factory.cpp
    http/http_msg_handler.cpp
    http/http_msg.cpp
    http/http_protocol.cpp
)

set(MSG_SRC_LIST
    msg/base_msg_handler_factory.cpp
    msg/base_msg_handler.cpp
    msg/base_msg.cpp
)

set(NETWORK_SRC_LIST
    network/socket_stream_base.cpp
    network/uthread_context_system.cpp
    network/socket_stream_block.cpp
    network/uthread_context_util.cpp
    network/socket_stream_uthread.cpp
    network/timer.cpp
    network/uthread_epoll.cpp
    network/uthread_context_base.cpp
    network/uthread_runtime.cpp
)

set(NETWORK_TEST_SRC_LIST
    network/test_echo_client.cpp
    network/test_timer.cpp
    network/test_epoll_server.cpp
    network/test_epoll_client.cpp
    network/test_uthread_context.cpp
    network/test_echo_server.cpp
    network/test_uthread.cpp
)

if (CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    list(APPEND NETWORK_SRC_LIST
        ${PHXRPC_DARWIN_HOME}/network/epoll-darwin.cpp
    )
    include_directories(
        ${PHXRPC_DARWIN_HOME}/network/
    )
    message("Using Darwin architecture!")
endif()

set(RPC_SRC_LIST
    rpc/caller.cpp
    rpc/hsha_server.cpp
    rpc/server_config.cpp
    rpc/uthread_caller.cpp
    rpc/client_config.cpp
    rpc/monitor_factory.cpp
    rpc/server_monitor.cpp
    rpc/client_monitor.cpp
    rpc/server_base.cpp
    rpc/socket_stream_phxrpc.cpp
)

set(RPC_TEST_SRC_LIST
    rpc/test_hsha_server.cpp
    rpc/test_thread_queue.cpp
    rpc/test_client.cpp
)

set(ALL_SRC_LIST
    ${COMMON_SRC_LIST}
    ${FILE_SRC_LIST}
    ${HTTP_SRC_LIST}
    ${MSG_SRC_LIST}
    ${NETWORK_SRC_LIST}
    ${NETWORK_SRC_LIST}
    ${RPC_SRC_LIST}
    ${PB_SRC_LIST}
)

if (BUILD_WITH_BOOST)
    list(APPEND ALL_SRC_LIST
        ${PHXRPC_BOOST_HOME}/network/uthread_context_boost.cpp
    )
    include_directories(
        ${PHXRPC_BOOST_HOME}/network
    )
endif()

add_library(${PROJECT_NAME} ${LIBRARY_BUILD_TYPE}
    ${ALL_SRC_LIST}
)

link_directories(
    ${PROTOBUF_HOME}/lib/
)

target_link_directories(${PROJECT_NAME} PRIVATE ${PROTOBUF_HOME}/lib)

target_link_libraries(
    ${PROJECT_NAME}
    PRIVATE
    protobuf
)
