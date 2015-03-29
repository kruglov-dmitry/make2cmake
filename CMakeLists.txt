cmake_minimum_required(VERSION 2.6 FATAL_ERROR)

# NOTE that The best method is to set the environment variables CC and CXX 
# before calling CMake for the very first time in a build tree.
#SET(CMAKE_C_COMPILER /usr/bin/gcc)   # to compile xerces library
#SET(CMAKE_CXX_COMPILER /usr/bin/g++)

project ( put_project_name_here C CXX )
find_package(PythonInterp 2 REQUIRED)

set (CMAKE_BINARY_DIR ${CMAKE_SOURCE_DIR}/build)
set (EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR})
set (LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR})

set (PROJECT_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include)
set (CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/CMake")

#
#	add precompiled headers - you can change them if you want
#	PCH_SRC is used in every sub-project if generation of precompiled headers are enabled 
set (PCH_SRC_H ${CMAKE_SOURCE_DIR}/stdwx.h)
#set (PCH_SRC_CPP ${CMAKE_SOURCE_DIR}/stdwx.cpp)
set (PCH_SRC_CPP ${CMAKE_SOURCE_DIR}/stdwx.c)
set (PCH_SRC_CMPL_FLAG )
#SET_SOURCE_FILES_PROPERTIES(${PCH_SRC_CPP} PROPERTIES COMPILE_FLAGS "${PCH_SRC_CMPL_FLAG} /Yc\"${PCH_SRC_H}\"")
set (PCH_SRC ${PCH_SRC_H} ${PCH_SRC_CPP})
#

# if we explicitly define targets - all their subfolders will be included in single project
# for example, project structure:
# lib 
#   src_1
#     	src_11
#     	src_12
#   src_2
#     	src_21
#     	src_22
# 	etc
# set (TARGET_LIST lib)
# will produce single lib which depend on all of the targets
# otherwise it will generate separate subprojects for every sub-folder in hierarchy

#set (TARGET_LIST src/xercesc tests)
set (TARGET_LIST fftw)

# TODO read config.in?
add_definitions( -DHAVE_CONFIG_H -UTIME_WITH_SYS_TIME)


#------------------------------------------------------------------------------
# Generate CMakeLists.txt files for every source sub directories if not exists
if (NOT EXISTS generated.flag)
  message(STATUS "-------------------------------------------------------------------")
  message(STATUS "Generating CMakeLists.txt files for every subdirectory containing src")
  message(STATUS "files. For details - check source_prefix in generate_cmakelists.py")
  message(STATUS "-------------------------------------------------------------------")
execute_process(
  COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_MODULE_PATH}/generate_cmakelists.py ${TARGET_LIST}
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  RESULT_VARIABLE CMAKE_GENERATION_RESULT
#  OUTPUT_VARIABLE SUBDIR_STRING
  ERROR_VARIABLE CMAKE_GENERATION_OUTPUT
  )
  if (CMAKE_GENERATION_RESULT)
    message(FATAL_ERROR "Generation of CMakeLists.txt files failed: \n${CMAKE_GENERATION_OUTPUT}")
  endif()
  file ( WRITE generated.flag )

#	message(STATUS "result of cmd:")
#	message(STATUS "${TARGET_LIST} ")
#	string ( REPLACE "\n" ";" SUBDIR_LIST ${SUBDIR_STRING} )

  # list of user defined targets goes here targets goes here
	foreach(subdir ${TARGET_LIST})
		add_subdirectory( ${subdir} )
	endforeach()

  # list of auto-generated targets goes here
  # module_list_start
  # module_list_end

endif()

