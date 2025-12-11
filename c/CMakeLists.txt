cmake_minimum_required(VERSION 3.10)
project(notepad C)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Compiler flags
if(MSVC)
    add_compile_options(/W4)
else()
    add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Main executable
add_executable(notepad src/main.c)
target_include_directories(notepad PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)

# Tests
enable_testing()
file(GLOB TEST_SOURCES tests/test_*.c)
foreach(TEST_SOURCE ${TEST_SOURCES})
    get_filename_component(TEST_NAME ${TEST_SOURCE} NAME_WE)
    add_executable(${TEST_NAME} ${TEST_SOURCE} src/module.c)
    target_include_directories(${TEST_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)
    add_test(NAME ${TEST_NAME} COMMAND ${TEST_NAME})
endforeach()

# Coverage target (GCC/Clang only)
if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
    add_custom_target(coverage
        COMMAND ${CMAKE_COMMAND} --build . --target test
        COMMAND gcov -p -o . ${CMAKE_CURRENT_SOURCE_DIR}/src/*.c
        COMMAND lcov --capture --directory . --output-file coverage.info
        COMMAND genhtml coverage.info --output-directory coverage
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    )
endif()
