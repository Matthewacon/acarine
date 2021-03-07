file(GLOB_RECURSE ACARINE_TEST_COMMON_SRC "src/**.c" "src/**.cpp")

add_library(acarine_test_common SHARED ${ACARINE_TEST_COMMON_SRC})
target_link_libraries(
 acarine_test_common
 acarine
 ${GOOGLE_TEST_TARGET}
)
target_include_directories(
 acarine_test_common PUBLIC 
 include
 ${gtest_SOURCE_DIR}/include
)

target_compile_options(
 acarine_test_common PUBLIC 
 ${ACARINE_COMPILE_FLAGS}
)
target_link_options(
 acarine_test_common PUBLIC
 ${ACARINE_LINK_FLAGS}
)

set(ACARINE_TEST_COMMON_TARGET acarine_test_common PARENT_SCOPE)
set(ACARINE_TEST_COMMON_INCLUDE_DIRS "${${ACARINE_TEST_COMMON_TARGET}_SOURCE_DIR}/include" PARENT_SCOPE)
