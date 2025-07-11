cmake_minimum_required(VERSION 3.14)
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
  "Enable compression support using zlib (this option is ignored if ENABLE_LIBDEFLATE is enabled)"
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

set(uwebsockets_headers
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

get_target_property(usockets_cpp_defs uSockets::uSockets "INTERFACE_COMPILE_DEFINITIONS")
if("LIBUS_USE_QUIC" IN_LIST usockets_cpp_defs)
  list(APPEND uwebsockets_headers
    "src/Http3App.h"
    "src/Http3Context.h"
    "src/Http3ContextData.h"
    "src/Http3Request.h"
    "src/Http3Response.h"
    "src/Http3ResponseData.h"
  )
endif()

add_library(uWebSockets INTERFACE)
target_compile_features(uWebSockets INTERFACE cxx_std_17)
target_compile_definitions(uWebSockets
  INTERFACE
    "$<$<BOOL:${ENABLE_PROXY}>:UWS_WITH_PROXY>"
    ${usockets_cpp_defs}
)
target_link_libraries(uWebSockets INTERFACE uSockets::uSockets)

if(ENABLE_LIBDEFLATE)
  target_compile_definitions(uWebSockets INTERFACE "UWS_USE_LIBDEFLATE")
  target_link_libraries(uWebSockets INTERFACE libdeflate::libdeflate_static)
elseif(ENABLE_ZLIB)
  target_link_libraries(uWebSockets INTERFACE ZLIB::ZLIB)
else()
  target_compile_definitions(uWebSockets INTERFACE "UWS_NO_ZLIB")
endif()

#----------------------------------------------------------------
# Installation
#----------------------------------------------------------------

include(GNUInstallDirs)

install(FILES
  ${uwebsockets_headers}
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/uWebSockets"
)
target_include_directories(uWebSockets
  INTERFACE "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
)

install(TARGETS uWebSockets EXPORT uWebSocketsConfig)

install(EXPORT uWebSocketsConfig
  DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/uWebSockets"
  NAMESPACE "uWebSockets::"
)
