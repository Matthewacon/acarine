cmake_minimum_required(VERSION 3.19)

project(acarine VERSION 0.1.0)

include(scripts/cmake-utils/cmake-fetch.cmake)

#Required defaults 
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)

#Explicitly disallow in-source builds
if("${PROJECT_SOURCE_DIR}" STREQUAL "${PROJECT_BINARY_DIR}")
 message(
  FATAL_ERROR
  "In-source builds are not permitted."
 )
endif()

#Display error if compiler is not clang
if(NOT ${CMAKE_CXX_COMPILER_ID} STREQUAL "Clang" OR NOT ${CMAKE_C_COMPILER_ID} STREQUAL "Clang")
 message(
  FATAL_ERROR
  "You are using an unsupported compiler, please build with Clang."
 )
endif()

#Determine whether or not this is the top-level project
if("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_CURRENT_LIST_DIR}")
 set(ACARINE_TOP_LEVEL TRUE)
else()
 set(ACARINE_TOP_LEVEL FALSE)
endif()

#acarine build options
set(ACARINE_BUILD_TYPE "shared" CACHE STRING "acarine library build type (defaults to 'shared'): [shared static]")
set(ACARINE_DEBUG OFF CACHE BOOL "Build acarine in debug mode")
set(ACARINE_QA OFF CACHE BOOL "Build acarine with sanitizers enabled")
set(ACARINE_COVERAGE OFF CACHE BOOL "Build acarine with llvm-coverage profiling enabled")
set(ACARINE_UNIT_TESTS OFF CACHE BOOL "Build and run the acarine unit tests")
set(ACARINE_INTEGRATION_TESTS OFF CACHE BOOL "Build and run the acarine integration tests")
set(ACARINE_BENCHMARKS OFF CACHE BOOL "Build and run the acarine benchmark suite")
set(ACARINE_EXAMPLES OFF CACHE BOOL "Build the acarine examples suite")

#Sanitize build options
#ACARINE_BUILD_TYPE
if(NOT (${ACARINE_BUILD_TYPE} STREQUAL "shared" OR ${ACARINE_BUILD_TYPE} STREQUAL "static"))
 message(
  FATAL_ERROR
  "Unknown build type specified for 'ACARINE_BUILD_TYPE': '${ACARINE_BUILD_TYPE}'"
 )
endif()

#Default compiler flags
list(
 APPEND ACARINE_COMPILE_FLAGS
 -Wall
 -Wextra
 -Wno-unused-parameter
 -Wno-unused-function
 -pedantic
 -ftemplate-backtrace-limit=0
 -fno-common
)

#Default linker flags
set(ACARINE_LINK_FLAGS "")

#Configure compiler and linker flags for debug and production builds
if(${ACARINE_DEBUG})
 #Debug compiler flags
 list(
  APPEND ACARINE_COMPILE_FLAGS
  -g
  -O0
  -fno-lto
  -fno-inline
  -fno-inline-functions
  -fno-omit-frame-pointer
  -fstandalone-debug
 )

 #Debug linker flags
 list(
  APPEND ACARINE_LINK_FLAGS
  -Wl,-fno-lto
 )
else()
 #Production compiler flags
 list(
  -O3
  -flto
 )

 #Production linker flags
 list(
  APPEND ACARINE_LINK_FLAGS
  -Wl,-flto 
 )
endif()

#Configure compiler and linker flags for QA builds
if(${ACARINE_QA})
 #QA Compiler flags
 list(
  APPEND ACARINE_COMPILE_FLAGS
  -fno-omit-frame-pointer
  -fno-sanitize-recover=null
  -fsanitize=address
  -fsanitize=undefined
  -fsanitize=address-use-after-scope
  -fsanitize=bounds
  -fsanitize=null
 )

 #QA Linker flags
 list(
  APPEND ACARINE_LINK_FLAGS
  -fsanitize=address
  -fsanitize=undefined
 )
endif()

#Configure compiler and linker flags for coverage builds
if(${ACARINE_COVERAGE})
 #Coverage compiler flags
 list(
  APPEND ACARINE_COMPILE_FLAGS
  -fprofile-instr-generate
  -fcoverage-mapping
 )

 #Coverage linker flags
 list(
  APPEND ACARINE_LINK_FLAGS
  -fprofile-instr-generate
  -fcoverage-mapping
 )
endif()

#TODO Declare all dependencies

#Configure library build
file(GLOB_RECURSE ACARINE_SRC "src/**.c" "src/**.cpp")

if(${ACARINE_BUILD_TYPE} STREQUAL "shared")
 add_library(acarine SHARED ${ACARINE_SRC})
else()
 add_library(acarine STATIC ${ACARINE_SRC})
endif()

#Fetch all latent dependencies and get list of link targets
fetch_latent_dependencies(TARGETS_VAR ACARINE_LINK_TARGETS)

#Disable public scope options if acarine is not the top-level project
if(${ACARINE_TOP_LEVEL})
 set(ACARINE_OPTION_SCOPE PUBLIC)
else()
 set(ACARINE_OPTION_SCOPE PRIVATE)
endif()

target_include_directories(acarine PUBLIC include)
target_compilation_options(acarine ${ACARINE_OPTION_SCOPE} ${ACARINE_COMPILE_FLAGS})
target_link_options(acarine ${ACARINE_OPTION_SCOPE} ${ACARINE_LINK_FLAGS})

#Include tests in build if testing or benchmarking
if(${ACARINE_UNIT_TESTS} OR ${ACARINE_INTEGRATION_TESTS} OR ${ACARINE_BENCHMARKS})
 add_subdirectory(tests)

 #TODO add llvm-coverage custom targets
else()
 #Warn if 'ACARINE_COVERAGE' is enabled but tests are not
 if(${ACARINE_COVERAGE})
  message(
   WARNING
   "'ACARINE_COVERAGE' is enabled but none of \
   [ACARINE_UNIT_TESTS ACARINE_INTEGRATION_TESTS ACARINE_BENCHMARKS] are \
   enabled, so no coverage runs will be performed.\
   "
  )
 endif()
endif()

#Add examples
if(${ACARINE_EXAMPLES})
 add_subdirectory(examples)
endif()
