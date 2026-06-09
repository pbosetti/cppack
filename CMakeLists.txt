cmake_minimum_required(VERSION 3.9)
execute_process(
  COMMAND git rev-parse --is-inside-work-tree
  WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
  OUTPUT_VARIABLE IS_GIT_REPO
  OUTPUT_STRIP_TRAILING_WHITESPACE
  ERROR_QUIET
)
if(IS_GIT_REPO STREQUAL "true")
  # Get the latest tag
  execute_process(
    COMMAND git describe --tags --dirty
    WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
    OUTPUT_VARIABLE GIT_TAG
    OUTPUT_STRIP_TRAILING_WHITESPACE
    ERROR_QUIET
  )
  if(GIT_TAG MATCHES "^v([0-9]+)\\.([0-9]+)\\.([0-9]+)(.*)?$")
    string(REGEX REPLACE "^v([0-9]+)\\.([0-9]+)\\.([0-9]+)(.*)?$" "\\1" CPACK_PACKAGE_VERSION_MAJOR "${GIT_TAG}")
    string(REGEX REPLACE "^v([0-9]+)\\.([0-9]+)\\.([0-9]+)(.*)?$" "\\2" CPACK_PACKAGE_VERSION_MINOR "${GIT_TAG}")
    string(REGEX REPLACE "^v([0-9]+)\\.([0-9]+)\\.([0-9]+)(.*)?$" "\\3" CPACK_PACKAGE_VERSION_PATCH "${GIT_TAG}")
    message(STATUS "Project root is a git repository, using version from latest tag: ${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")
  else()
    message(WARNING "Latest git tag does not match vMAJOR.MINOR.PATCH format, using default version")
    set(CPACK_PACKAGE_VERSION_MAJOR 0)
    set(CPACK_PACKAGE_VERSION_MINOR 0)
    set(CPACK_PACKAGE_VERSION_PATCH 1)
  endif()
else()
  message(WARNING "Project root is NOT a git repository, using default version")
  set(CPACK_PACKAGE_VERSION_MAJOR 0)
  set(CPACK_PACKAGE_VERSION_MINOR 0)
  set(CPACK_PACKAGE_VERSION_PATCH 1)
  set(GIT_TAG "vUNKNOWN_GIT_VERSION")
endif()

project(Msgpack LANGUAGES CXX VERSION ${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH})

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

enable_testing()
add_subdirectory(msgpack)