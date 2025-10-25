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
set(find_dependency_calls "find_dependency(uSockets REQUIRED)\n")

if(ENABLE_LIBDEFLATE)
  find_package(libdeflate REQUIRED)
  string(APPEND find_dependency_calls "find_dependency(libdeflate REQUIRED)\n")
elseif(ENABLE_ZLIB)
  find_package(ZLIB REQUIRED)
  string(APPEND find_dependency_calls "find_dependency(ZLIB REQUIRED)\n")
endif()

include(GNUInstallDirs)
set(CMAKE_INSTALL_DOCDIR "${CMAKE_INSTALL_DATAROOTDIR}/doc/${PROJECT_NAME}")

#----------------------------------------------------------------
# Targets and properties
#----------------------------------------------------------------

add_library(uWebSockets_uWebSockets INTERFACE)
target_compile_features(uWebSockets_uWebSockets INTERFACE cxx_std_17)

set(headers_install_dir "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}")
target_include_directories(uWebSockets_uWebSockets
  INTERFACE
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>"
    "$<INSTALL_INTERFACE:${headers_install_dir}>"
)

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

foreach(header IN LISTS headers)
  get_filename_component(name "${header}" NAME)
  target_sources(uWebSockets_uWebSockets
    INTERFACE
      "$<BUILD_INTERFACE:${header}>"
      "$<INSTALL_INTERFACE:${headers_install_dir}/${name}>"
  )
endforeach()

set_target_properties(uWebSockets_uWebSockets
  PROPERTIES
    EXPORT_NAME "${PROJECT_NAME}"
    PUBLIC_HEADER "${headers}"
)

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

install(TARGETS uWebSockets_uWebSockets
  EXPORT exported_targets
  PUBLIC_HEADER DESTINATION "${headers_install_dir}"
)

set(config_dest "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

install(EXPORT exported_targets
  DESTINATION "${config_dest}"
  NAMESPACE   "${PROJECT_NAME}::"
  FILE        "${PROJECT_NAME}Targets.cmake"
)

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
  "include(CMakeFindDependencyMacro)
${find_dependency_calls}
include(\"\${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}Targets.cmake\")"
)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
  DESTINATION "${config_dest}"
)

install(FILES "README.md" "misc/READMORE.md"
  TYPE DOC
)
install(DIRECTORY "examples"
  TYPE DOC
)
