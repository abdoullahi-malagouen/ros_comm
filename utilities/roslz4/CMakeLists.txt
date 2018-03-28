cmake_minimum_required(VERSION 2.8.3)

project(roslz4)

if(NOT WIN32)
  set_directory_properties(PROPERTIES COMPILE_OPTIONS "-Wall;-Wextra")
endif()

if(WITH_CATKIN)
	find_package(catkin REQUIRED)
endif()

find_path(lz4_INCLUDE_DIRS NAMES lz4.h)
if (NOT lz4_INCLUDE_DIRS)
  message(FATAL_ERROR "lz4 includes not found")
endif()

find_library(lz4_LIBRARIES NAMES lz4)
if (NOT lz4_LIBRARIES)
  message(FATAL_ERROR "lz4 library not found")
endif()
set(lz4_FOUND TRUE)

if(WITH_CATKIN)
	catkin_python_setup()
endif()

if(WITH_CATKIN)
catkin_package(
  INCLUDE_DIRS include
  LIBRARIES roslz4
  DEPENDS lz4)
endif()

if(WITH_CATKIN)
	include_directories(include ${lz4_INCLUDE_DIRS} ${catkin_INCLUDE_DIRS})
else()
	include_directories(include ${lz4_INCLUDE_DIRS})
endif()	

add_library(roslz4 src/lz4s.c src/xxhash.c)

if(NOT WIN32)
	set_source_files_properties(src/lz4s.c  PROPERTIES COMPILE_FLAGS "-Wno-sign-compare")
#elseif(WIN32)
	#set_source_files_properties(src/lz4s.c)	
endif()
set_source_files_properties(src/lz4s.c src/xxhash.c PROPERTIES COMPILE_DEFINITIONS "XXH_NAMESPACE=ROSLZ4_")

if(WITH_CATKIN)
	target_link_libraries(roslz4 ${lz4_LIBRARIES} ${catkin_LIBRARIES})
else()
	target_link_libraries(roslz4 ${lz4_LIBRARIES})
endif()



if(NOT ANDROID)
if(WITH_PYTHON)
# Python bindings
set(Python_ADDITIONAL_VERSIONS "${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR}")
find_package(PythonLibs "${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR}" REQUIRED)
include_directories(${PYTHON_INCLUDE_PATH})

add_library(roslz4_py src/_roslz4module.c)
set_source_files_properties(
  src/_roslz4module.c
PROPERTIES COMPILE_FLAGS "-Wno-missing-field-initializers -Wno-unused-variable -Wno-strict-aliasing")
endif()

if(WITH_CATKIN)
set_target_properties(roslz4_py PROPERTIES OUTPUT_NAME roslz4 PREFIX "_" SUFFIX ".so" LIBRARY_OUTPUT_DIRECTORY ${CATKIN_DEVEL_PREFIX}/${PYTHON_INSTALL_DIR}/roslz4)
target_link_libraries(roslz4_py roslz4 ${catkin_LIBRARIES} ${PYTHON_LIBRARIES})
elseif(NOT WIN32)
set_target_properties(roslz4_py PROPERTIES OUTPUT_NAME roslz4 PREFIX "_" SUFFIX ".so" LIBRARY_OUTPUT_DIRECTORY ${CATKIN_DEVEL_PREFIX}/${PYTHON_INSTALL_DIR}/roslz4)
endif()
endif()

if(WITH_CATKIN)
install(TARGETS roslz4
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

if(NOT ANDROID)
install(TARGETS roslz4_py
  LIBRARY DESTINATION ${CATKIN_PACKAGE_PYTHON_DESTINATION})
endif()

install(DIRECTORY include/${PROJECT_NAME}
  DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
)

# Testing
if (CATKIN_ENABLE_TESTING)
  catkin_add_gtest(test_roslz4 test/roslz4_test.cpp)
  if (TARGET test_roslz4)
    target_link_libraries(test_roslz4 roslz4 ${catkin_LIBRARIES})
  endif()
endif()

endif()

