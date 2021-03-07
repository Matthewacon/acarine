file(GLOB_RECURSE ACARINE_TESTS "src/**.c" "src/**.cpp")

foreach(TEST ${ACARINE_TESTS})
 #Convert the fully-qualified path in `TEST` to a path relative to
 #the current source directory
 file(RELATIVE_PATH TEST "${CMAKE_CURRENT_SOURCE_DIR}" "${TEST}")

 #Convert the path to a cmake-style list
 string(REPLACE "/" ";" TEST_FILE_PATH "${TEST}")
 
 #Get file name from the path and remove the extension
 list(GET TEST_FILE_PATH -1 TEST_FILE_NAME)
 string(REPLACE ".cpp" "" TEST_FILE_NAME "${TEST_FILE_NAME}")
 string(REPLACE ".c" "" TEST_FILE_NAME "${TEST_FILE_NAME}")
 
 #Build preface string with all leading directories unitl `src/acarine` to
 #distinguish tests with the same name. (removes the `src` and `acarine` 
 #elements as well as the target name, since it will be combined later)
 list(REMOVE_AT TEST_FILE_PATH 1 0 -1)
 #Reset the preface string before building it
 unset(TEST_FILE_PATH_PREFIX)
 foreach(PATH_PREFIX ${TEST_FILE_PATH})
  string(APPEND TEST_FILE_PATH_PREFIX "${PATH_PREFIX}.")
 endforeach()

 #Combine `TEST_FILE_PATH_PREFIX` and `TEST_FILE_NAME` to form the unique
 #test target name
 string(PREPEND TEST_FILE_NAME "${TEST_FILE_PATH_PREFIX}")

 set(TEST_TARGET_NAME "${TEST_FILE_NAME}_unit_test")
 add_executable(${TEST_TARGET_NAME} ${TEST})
 target_include_directories(
  ${TEST_TARGET_NAME} PRIVATE
  include
  ${ACARINE_TEST_COMMON_INCLUDE_DIRS} 
  ${GOOGLE_TEST_INCLUDE_DIRS}
 )
 target_compile_options(${TEST_TARGET_NAME} PRIVATE "${ACARINE_COMPILE_FLAGS}")
 target_link_libraries(
  ${TEST_TARGET_NAME}
  acarine
  ${ACARINE_TEST_COMMON_TARGET}
  ${GOOGLE_TEST_TARGET}
  ${GOOGLE_TEST_MAIN_TARGET}
 )
 target_link_options(${TEST_TARGET_NAME} PUBLIC ${ACARINE_LINK_FLAGS})
 add_test(
  NAME ${TEST_TARGET_NAME}
  COMMAND ./${TEST_TARGET_NAME}
  WORKING_DIRECTORY unit
 )
 list(APPEND ACARINE_TEST_TARGETS "${TEST_TARGET_NAME}")
endforeach()

set(ACARINE_TEST_TARGETS "${ACARINE_TEST_TARGETS}" PARENT_SCOPE)
