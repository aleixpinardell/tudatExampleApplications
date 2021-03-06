 #    Copyright (c) 2010-2017, Delft University of Technology
 #    All rigths reserved
 #
 #    This file is part of the Tudat. Redistribution and use in source and
 #    binary forms, with or without modification, are permitted exclusively
 #    under the terms of the Modified BSD license. You should have received
 #    a copy of the license with this file. If not, please or visit:
 #    http://tudat.tudelft.nl/LICENSE.

# Specify minimum CMake version required.
cmake_minimum_required(VERSION 2.6)

# Specify project name.
project(JSON_Example)

# Load UserSettings.txt
if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_SOURCE_DIR})
  set(BUILD_STYLE "standalone")
  include("${CMAKE_CURRENT_SOURCE_DIR}/UserSettings.txt" OPTIONAL)
else()
  set(BUILD_STYLE "part of ${CMAKE_PROJECT_NAME}")
  include("${CMAKE_CURRENT_SOURCE_DIR}/UserSettings.txt" OPTIONAL)
  include("${CMAKE_SOURCE_DIR}/UserSettings.txt" OPTIONAL)
  STRING(REGEX REPLACE ${CMAKE_SOURCE_DIR} "" RELATIVE_PROJECT_PATH ${CMAKE_CURRENT_SOURCE_DIR})
  set(RELATIVE_PROJECT_PATH "${RELATIVE_PROJECT_PATH}" CACHE STRING "Relative path wrt to project for function")
  # message(STATUS "Relative path (wrt to project): ${RELATIVE_PROJECT_PATH}")
endif()

# Set CMake build-type. If it not supplied by the user (either directly as an argument of through
# the "UserSettings.txt" file, the default built type is "Release".
if((NOT CMAKE_BUILD_TYPE) OR (CMAKE_BUILD_TYPE STREQUAL "Release"))
  set(CMAKE_BUILD_TYPE Release)
elseif(CMAKE_BUILD_TYPE STREQUAL "Debug")
  set(CMAKE_BUILD_TYPE Debug)
endif()

message(STATUS "<< ${PROJECT_NAME} (${CMAKE_BUILD_TYPE} - ${BUILD_STYLE}) >>")

# Add local module path
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/CMakeModules")
message(STATUS "CMake Module path(s): ${CMAKE_MODULE_PATH}")

# Set compiler based on preferences (e.g. USE_CLANG) and system.
include(compiler)

# Define the directory with the source code.
set(SRCROOT "${CMAKE_CURRENT_SOURCE_DIR}")

# Define the code root directory.
set(CODEROOT "${CMAKE_CURRENT_SOURCE_DIR}/..")

# Set testing options based on platform.
enable_testing()

# Set lib and bin directories where static libraries and unit tests are built.
if(NOT LIBROOT)
  set(LIBROOT "${CODEROOT}/lib")
endif()
if(NOT BINROOT)
  set(BINROOT "${CODEROOT}/bin")
endif()

# Set the global macros for setting up targets.
macro(setup_executable_target target_name CUSTOM_OUTPUT_PATH)
  set_property(TARGET ${target_name} PROPERTY RUNTIME_OUTPUT_DIRECTORY "${BINROOT}/applications")
  install(TARGETS ${target_name} RUNTIME DESTINATION "${BINROOT}/applications")
endmacro(setup_executable_target)

# Include the top-level directories.
include_directories(AFTER
  "${CODEROOT}"
)

# Find Eigen3 library on local system.
find_package(Eigen3 REQUIRED)

# Include Eigen3 directories.
# Set CMake flag to suppress Eigen warnings (platform-dependent solution).
if(NOT APPLE OR APPLE_INCLUDE_FORCE)
  include_directories(SYSTEM AFTER "${EIGEN3_INCLUDE_DIR}")
else()
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -isystem \"${EIGEN3_INCLUDE_DIR}\"")
endif()

# Find Tudat library on local system.
find_package(Tudat 2.0 REQUIRED)

# Include Tudat directories.
# Set CMake flag to suppress Tudat warnings (platform-dependent solution).
if(NOT APPLE OR APPLE_INCLUDE_FORCE)
  include_directories(SYSTEM AFTER "${TUDAT_INCLUDE_DIR}")
else()
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -isystem \"${TUDAT_INCLUDE_DIR}\"")
endif()

# Find JSONCPP library on local system.
find_package(JSONCPP)

# Include JSONCPP directories.
if(NOT APPLE OR APPLE_INCLUDE_FORCE)
  include_directories(SYSTEM AFTER "${JSONCPP_INCLUDE_DIR}")
else( )
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -isystem \"${JSONCPP_INCLUDE_DIR}\"")
endif( )

# Add application.
add_executable(example_library_JSON "${SRCROOT}/Example_JSON.cpp")
setup_executable_target(example_library_JSON "${SRCROOT}")
target_link_libraries(example_library_JSON jsoncpp)
