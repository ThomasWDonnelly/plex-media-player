cmake_minimum_required(VERSION 3.1.0 FATAL_ERROR)

# use a toolchain file if any for Embedded
if (EXISTS "${CMAKE_SOURCE_DIR}/toolchain.cmake")
  set(CMAKE_TOOLCHAIN_FILE "${CMAKE_SOURCE_DIR}/toolchain.cmake")
endif()

project(PlexMediaPlayer CXX C)

if(EXISTS ${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
  include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
else()
  message(WARNING "Could not find conanbuildinfo.cmake - could be a problem! Make sure to run conan install first")
endif()

# we want this to be able to run clang-tidy
set(CMAKE_EXPORT_COMPILE_COMMANDS ON CACHE BOOL "" FORCE)

option(OPENELEC "Make an OpenELEC build" OFF)
option(LINUX_DBUS "Enable D-Bus power management" OFF)

if(OPENELEC)
  set(LINUX_DBUS ON)
  add_definitions(-DKONVERGO_OPENELEC=1)
  Message(STATUS "Making an OpenELEC build")
endif(OPENELEC)

if(LINUX_DBUS)
  add_definitions(-DLINUX_DBUS=1)
  Message(STATUS "Enabling D-Bus power management")
endif(LINUX_DBUS)

if(BUILD_TARGET STREQUAL "RPI")
  add_definitions(-DTARGET_RPI=1)
  set(RPI_LIBS bcm_host)
  Message(STATUS "Build for Raspberry PI target")
endif(BUILD_TARGET STREQUAL "RPI")

option(ENABLE_HELPER "Enable PMP Helper" ON)
if (ENABLE_HELPER)
  add_definitions(-DENABLE_HELPER=1)
endif(ENABLE_HELPER)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/CMakeModules/")
set(CMAKE_INSTALL_DEFAULT_COMPONENT_NAME Core)

macro(set_policy _POLICY _VALUE)
  if(POLICY ${_POLICY})
    cmake_policy(SET ${_POLICY} ${_VALUE})
  endif()
endmacro()

set_policy(CMP0020 NEW)
set_policy(CMP0017 NEW)
set_policy(CMP0058 NEW)
set_policy(CMP0026 OLD)

include(utils)
include(CompilerFlags)
include(DependencyConfiguration)
include(QtConfiguration)
include(VersionConfiguration)
include(NameConfiguration)
include(PlayerConfiguration)
include(InputConfiguration)
include(FindBreakpad)
include(BreakpadSymbols)
include(ClangTidy)
include(WebClientConan)

add_definitions(-DQS_LOG_LINE_NUMBERS -DQS_LOG_SEPARATE_THREAD)

if(APPLE)
  include(AppleConfiguration)
elseif(WIN32)
  include(Win32Configuration)
elseif(UNIX AND (NOT APPLE))
  include(LinuxConfiguration)
endif(APPLE)

if (Qt5_POSITION_INDEPENDENT_CODE)
  SET(CMAKE_POSITION_INDEPENDENT_CODE ON)
endif(Qt5_POSITION_INDEPENDENT_CODE)

add_subdirectory(external)
add_subdirectory(src)

include(CPackConfiguration)

add_custom_target(install_app_bundle COMMAND ${CMAKE_COMMAND} -P cmake_install.cmake DEPENDS PlexMediaPlayer PMPHelper)
