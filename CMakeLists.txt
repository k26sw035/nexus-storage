cmake_minimum_required(VERSION 3.20)

project(NexusStorage
  VERSION 1.0.0
  DESCRIPTION "S3-compatible object storage client"
  LANGUAGES CXX)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()

option(NEXUS_BUILD_CLI "Build the nexus-cli executable" ON)
option(NEXUS_WARNINGS_AS_ERRORS "Treat compiler warnings as errors" OFF)
option(NEXUS_ENABLE_SANITIZERS "Enable ASan/UBSan (GCC/Clang)" OFF)

set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(CURL REQUIRED)
find_package(OpenSSL REQUIRED)

function(nexus_apply_warnings target)
  if(MSVC)
    target_compile_options(${target} PRIVATE /W4 /permissive-
      $<$<BOOL:${NEXUS_WARNINGS_AS_ERRORS}>:/WX>)
  else()
    target_compile_options(${target} PRIVATE -Wall -Wextra -Wpedantic -Wshadow
      $<$<BOOL:${NEXUS_WARNINGS_AS_ERRORS}>:-Werror>)
    if(NEXUS_ENABLE_SANITIZERS)
      target_compile_options(${target} PRIVATE -fsanitize=address,undefined -fno-omit-frame-pointer)
      target_link_options(${target} PRIVATE -fsanitize=address,undefined)
    endif()
  endif()
endfunction()

add_library(nexus
  src/config.cpp
  src/errors.cpp
  src/http_client.cpp
  src/sigv4.cpp
  src/storage_client.cpp)
add_library(nexus::nexus ALIAS nexus)

target_compile_features(nexus PUBLIC cxx_std_17)
target_include_directories(nexus PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
target_link_libraries(nexus PRIVATE CURL::libcurl OpenSSL::Crypto)
set_target_properties(nexus PROPERTIES
  VERSION ${PROJECT_VERSION}
  SOVERSION ${PROJECT_VERSION_MAJOR}
  WINDOWS_EXPORT_ALL_SYMBOLS ON)
nexus_apply_warnings(nexus)

if(NEXUS_BUILD_CLI)
  add_executable(nexus-cli src/cli/main.cpp)
  target_link_libraries(nexus-cli PRIVATE nexus::nexus)
  nexus_apply_warnings(nexus-cli)
  install(TARGETS nexus-cli RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
endif()

install(TARGETS nexus
  EXPORT nexusTargets
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

set(NEXUS_CMAKE_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/nexus)
install(EXPORT nexusTargets NAMESPACE nexus:: DESTINATION ${NEXUS_CMAKE_DIR})

configure_package_config_file(
  cmake/nexusConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/nexusConfig.cmake
  INSTALL_DESTINATION ${NEXUS_CMAKE_DIR})
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/nexusConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion)
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/nexusConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/nexusConfigVersion.cmake
  DESTINATION ${NEXUS_CMAKE_DIR})