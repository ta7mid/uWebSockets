cmake_minimum_required(VERSION 3.15)
# Determined by: https://cmake.org/cmake/help/latest/command/add_library.html#index-0-prop_tgt:PUBLIC_HEADER

project(uWebSockets VERSION 20.74.0 LANGUAGES C CXX)

#----------------------------------------------------------------
# Options and dependencies
#----------------------------------------------------------------

option(
  ENABLE_LIBDEFLATE
  "Enable compression support using libdeflate (if enabled, ENABLE_ZLIB will be ignored)"
  OFF
)
option(
  ENABLE_ZLIB
  "Enable compression support using zlib (ignored if ENABLE_LIBDEFLATE is enabled)"
  OFF
)
option(ENABLE_PROXY "Enable PROXY Protocol v2 support" OFF)

find_package(uSockets REQUIRED)

if(ENABLE_LIBDEFLATE)
  find_package(libdeflate REQUIRED)
elseif(ENABLE_ZLIB)
  find_package(ZLIB REQUIRED)
endif()

#----------------------------------------------------------------
# Targets and properties
#----------------------------------------------------------------

include(GNUInstallDirs)

add_library(uWebSockets_uWebSockets INTERFACE)
target_include_directories(uWebSockets_uWebSockets INTERFACE "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>")
target_compile_features(uWebSockets_uWebSockets INTERFACE cxx_std_17)

set(headers
  "src/App.h"
  "src/AsyncSocket.h"
  "src/AsyncSocketData.h"
  "src/BloomFilter.h"
  "src/CachingApp.h"
  "src/ChunkedEncoding.h"
  "src/ClientApp.h"
  "src/HttpContext.h"
  "src/HttpContextData.h"
  "src/HttpErrors.h"
  "src/HttpParser.h"
  "src/HttpResponse.h"
  "src/HttpResponseData.h"
  "src/HttpRouter.h"
  "src/LocalCluster.h"
  "src/Loop.h"
  "src/LoopData.h"
  "src/MessageParser.h"
  "src/MoveOnlyFunction.h"
  "src/Multipart.h"
  "src/PerMessageDeflate.h"
  "src/ProxyParser.h"
  "src/QueryParser.h"
  "src/TopicTree.h"
  "src/Utilities.h"
  "src/WebSocket.h"
  "src/WebSocketContext.h"
  "src/WebSocketContextData.h"
  "src/WebSocketData.h"
  "src/WebSocketExtensions.h"
  "src/WebSocketHandshake.h"
  "src/WebSocketProtocol.h"
)

get_target_property(usockets_cpp_defs uSockets::uSockets INTERFACE_COMPILE_DEFINITIONS)
target_compile_definitions(uWebSockets_uWebSockets INTERFACE ${usockets_cpp_defs})
target_link_libraries(uWebSockets_uWebSockets INTERFACE uSockets::uSockets)

if("LIBUS_USE_QUIC" IN_LIST usockets_cpp_defs)
  list(APPEND headers
    "src/Http3App.h"
    "src/Http3Context.h"
    "src/Http3ContextData.h"
    "src/Http3Request.h"
    "src/Http3Response.h"
    "src/Http3ResponseData.h"
  )
endif()

set_target_properties(uWebSockets_uWebSockets PROPERTIES PUBLIC_HEADER "${headers}")

if(ENABLE_PROXY)
  target_compile_definitions(uWebSockets_uWebSockets INTERFACE "UWS_WITH_PROXY")
endif()

if(ENABLE_LIBDEFLATE)
  target_compile_definitions(uWebSockets_uWebSockets INTERFACE "UWS_USE_LIBDEFLATE")
  target_link_libraries(uWebSockets_uWebSockets INTERFACE libdeflate::libdeflate_static)
elseif(ENABLE_ZLIB)
  target_link_libraries(uWebSockets_uWebSockets INTERFACE ZLIB::ZLIB)
else()
  target_compile_definitions(uWebSockets_uWebSockets INTERFACE "UWS_NO_ZLIB")
endif()

#----------------------------------------------------------------
# Installation
#----------------------------------------------------------------

set(exported_name "${PROJECT_NAME}")
set(config_dest "${CMAKE_INSTALL_LIBDIR}/cmake/${exported_name}")

set_target_properties(uWebSockets_uWebSockets PROPERTIES EXPORT_NAME "${exported_name}")
install(TARGETS uWebSockets_uWebSockets
  EXPORT exported_targets
  PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${exported_name}"
)

install(EXPORT exported_targets
  DESTINATION "${config_dest}"
  NAMESPACE "${exported_name}::"
  FILE "uWebSocketsTargets.cmake"
)

# language=CMake
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/${exported_name}Config.cmake" [[
include(CMakeFindDependencyMacro)
find_dependency(uSockets REQUIRED)
include("${CMAKE_CURRENT_LIST_DIR}/uWebSocketsTargets.cmake")
]])
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${exported_name}Config.cmake"
  DESTINATION "${config_dest}"
)
