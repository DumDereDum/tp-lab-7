cmake_minimum_required(VERSION 2.8)

# Make PROJECT_SOURCE_DIR, PROJECT_BINARY_DIR, and PROJECT_NAME available.
set(PROJECT_NAME ocean) 
project(${PROJECT_NAME}) 

set(CMAKE_CXX_FLAGS "-g -Wall")

# If you want your own include/ directory, set this, and then you can do
# include_directories(${COMMON_INCLUDES}) in other CMakeLists.txt files.
# set(COMMON_INCLUDES ${PROJECT_SOURCE_DIR}/include)

#----------------------------------------------------------------------------
# Define project sources and includes
#----------------------------------------------------------------------------
set(COMMON_INCLUDES ${PROJECT_SOURCE_DIR}/include)
include_directories(${COMMON_INCLUDES})

add_subdirectory(ext/gtest-1.8.0)

include_directories(${COMMON_INCLUDES})
include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})

set(TEST_SRC_FILES_LOCAL ${PROJECT_SOURCE_DIR}/test/)
file(GLOB TEST_SRC_FILES ${PROJECT_SOURCE_DIR}/test/*.cpp)


set(SRC_FILES
        src/main.cpp
        src/Ocean.cpp include/Ocean.h
        src/Cell.cpp include/Cell.h
        src/Object.cpp include/Object.h
        src/Stone.cpp include/Stone.h
        src/Fish.cpp include/Fish.h
        src/Prey.cpp include/Prey.h
        src/Predator.cpp include/Predator.h
        )

set(CMAKE_CXX_STANDARD 11)

add_library(ocean_lib ${SRC_FILES})
add_executable(ocean_run ${PROJECT_SOURCE_DIR}/src/main.cpp)

target_link_libraries(ocean_run ocean_lib)

add_executable(runUnitTests test/tests.cpp)

target_link_libraries(runUnitTests gtest gtest_main)
target_link_libraries(runUnitTests ocean_lib)

################################
# Testing
################################

# Options. Turn on with 'cmake -Dmyvarname=ON'.
option(BUILD_TESTS "Build all tests." OFF) # Makes boolean 'test' available.

if (BUILD_TESTS)
  # This adds another subdirectory, which has 'project(gtest)'.
   
  enable_testing()

  # Include the gtest library. gtest_SOURCE_DIR is available due to
  # 'project(gtest)' above.
  include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})

  ##############
  # Unit Tests
  ##############
  
  # This is so you can do 'make test' to see all your tests run, instead of
  # manually running the executable runUnitTests to see those specific tests.
  add_test(UnitTests runUnitTests)

  set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/.travis/cmake)

  if (CMAKE_BUILD_TYPE STREQUAL "Coverage")
    include(CodeCoverage)
    setup_target_for_coverage(${PROJECT_NAME}_coverage runUnitTests coverage)

    SET(CMAKE_CXX_FLAGS "-g -O0 -fprofile-arcs -ftest-coverage")
    SET(CMAKE_C_FLAGS "-g -O0 -fprofile-arcs -ftest-coverage")
  endif() #CMAKE_BUILD_TYPE STREQUAL "Coverage"

endif()
