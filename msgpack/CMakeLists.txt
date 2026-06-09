cmake_minimum_required(VERSION 3.9)

list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(GNUInstallDirs)

option(CPPACK_ENABLE_TESTS "Enable tests (requires Catch2)" OFF)

add_library(Msgpack INTERFACE)
add_library(Msgpack::Msgpack ALIAS Msgpack)

target_compile_features(Msgpack INTERFACE cxx_std_17)

target_include_directories(Msgpack
                           INTERFACE
                           $<INSTALL_INTERFACE:include>
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                           )

install(TARGETS Msgpack
        EXPORT MsgpackTargets
        DESTINATION ${CMAKE_INSTALL_LIBDIR}
        )

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

set(MSGPACK_CMAKE_CONFIG_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/Msgpack")

install(EXPORT MsgpackTargets
        FILE MsgpackTargets.cmake
        NAMESPACE Msgpack::
        DESTINATION ${MSGPACK_CMAKE_CONFIG_DESTINATION})

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
        ${CMAKE_CURRENT_BINARY_DIR}/MsgpackConfigVersion.cmake
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY AnyNewerVersion
)
configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/cmake/MsgpackConfig.cmake.in
                              ${CMAKE_CURRENT_BINARY_DIR}/MsgpackConfig.cmake
                              INSTALL_DESTINATION ${MSGPACK_CMAKE_CONFIG_DESTINATION}
                              )

install(FILES
        ${CMAKE_CURRENT_BINARY_DIR}/MsgpackConfig.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/MsgpackConfigVersion.cmake
        DESTINATION ${MSGPACK_CMAKE_CONFIG_DESTINATION})

export(EXPORT MsgpackTargets FILE ${CMAKE_CURRENT_BINARY_DIR}/MsgpackTargets.cmake NAMESPACE Msgpack::)
export(PACKAGE Msgpack)

if (CPPACK_ENABLE_TESTS)
        add_subdirectory(tests)
endif()

# PACKAGE #######################################################################
string(TOLOWER "${CMAKE_SYSTEM_NAME}" PACKAGE_OS_NAME)
if(APPLE AND CMAKE_OSX_ARCHITECTURES)
  list(LENGTH CMAKE_OSX_ARCHITECTURES PACKAGE_OSX_ARCHITECTURES_COUNT)
  if(PACKAGE_OSX_ARCHITECTURES_COUNT GREATER 1)
    set(PACKAGE_ARCH_NAME "universal")
  else()
    set(PACKAGE_ARCH_NAME "${CMAKE_OSX_ARCHITECTURES}")
  endif()
else()
  set(PACKAGE_ARCH_NAME "${CMAKE_SYSTEM_PROCESSOR}")
endif()
if(PACKAGE_ARCH_NAME STREQUAL "")
  set(PACKAGE_ARCH_NAME "${CMAKE_HOST_SYSTEM_PROCESSOR}")
endif()
if(PACKAGE_ARCH_NAME STREQUAL "")
  set(PACKAGE_ARCH_NAME "unknown")
endif()
string(TOLOWER "${PACKAGE_ARCH_NAME}" PACKAGE_ARCH_NAME)

set(CPACK_GENERATOR "TGZ")
set(CPACK_PACKAGE_NAME "${PROJECT_NAME}")
set(CPACK_PACKAGE_VENDOR "CPPack Contributors")
set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}")
set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY ON)
set(CPACK_PACKAGE_FILE_NAME
  "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${PACKAGE_OS_NAME}-${PACKAGE_ARCH_NAME}")
include(CPack)