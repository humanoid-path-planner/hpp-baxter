#
# Copyright (c) 2016, 2020 CNRS Authors: Joseph Mirabel, Guilhem Saurel
#

cmake_minimum_required(VERSION 3.22)

set(PROJECT_NAME hpp-baxter)
set(PROJECT_DESCRIPTION "Wrappers for Baxter robot in HPP.")

set(PROJECT_USE_CMAKE_EXPORT TRUE)

# Check if the submodule cmake have been initialized
set(JRL_CMAKE_MODULES "${CMAKE_CURRENT_LIST_DIR}/cmake")
if(EXISTS "${JRL_CMAKE_MODULES}/base.cmake")
  message(STATUS "JRL cmakemodules found in 'cmake/' git submodule")
else()
  find_package(jrl-cmakemodules QUIET CONFIG)
  if(jrl-cmakemodules_FOUND)
    get_property(
      JRL_CMAKE_MODULES
      TARGET jrl-cmakemodules::jrl-cmakemodules
      PROPERTY INTERFACE_INCLUDE_DIRECTORIES)
    message(STATUS "JRL cmakemodules found on system at ${JRL_CMAKE_MODULES}")
  else()
    message(STATUS "JRL cmakemodules not found. Let's fetch it.")
    include(FetchContent)
    FetchContent_Declare(
      "jrl-cmakemodules"
      GIT_REPOSITORY "https://github.com/jrl-umi3218/jrl-cmakemodules.git")
    FetchContent_MakeAvailable("jrl-cmakemodules")
    FetchContent_GetProperties("jrl-cmakemodules" SOURCE_DIR JRL_CMAKE_MODULES)
  endif()
endif()

include("${JRL_CMAKE_MODULES}/hpp.cmake")
include("${JRL_CMAKE_MODULES}/python.cmake")

compute_project_args(PROJECT_ARGS LANGUAGES CXX)
project(${PROJECT_NAME} ${PROJECT_ARGS})

add_project_dependency(example-robot-data 3.9.1 REQUIRED)

set(CATKIN_PACKAGE_SHARE_DESTINATION
    ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME})

if(NOT BUILD_STANDALONE_PYTHON_INTERFACE)
  add_library(${PROJECT_NAME} INTERFACE)
  install(
    TARGETS ${PROJECT_NAME}
    EXPORT ${TARGETS_EXPORT_NAME}
    DESTINATION lib)
  install(FILES urdf/box.urdf urdf/baxter.urdf urdf/baxter_colman.urdf
          DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/urdf)
  install(FILES srdf/box.srdf srdf/baxter.srdf srdf/baxter_manipulation.srdf
          DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/srdf)
endif()
if(BUILD_PYTHON_INTERFACE)
  findpython(REQUIRED)
  install(FILES src/hpp/corbaserver/baxter/robot.py
                src/hpp/corbaserver/baxter/__init__.py
          DESTINATION ${PYTHON_SITELIB}/hpp/corbaserver/baxter)
  install(FILES src/hpp/corbaserver/manipulation/baxter/robot.py
                src/hpp/corbaserver/manipulation/baxter/__init__.py
          DESTINATION ${PYTHON_SITELIB}/hpp/corbaserver/manipulation/baxter)
endif()
