cmake_minimum_required(VERSION 3.10)
project(sg14 CXX)

#
# Option to build tests (default: OFF)
#
option(SG14_BUILD_TESTS "Build SG14 unit tests" OFF)

#
# Prefer C++17, downgrade if it isn't available
#
set(CMAKE_CXX_STANDARD_REQUIRED OFF)
set(CMAKE_CXX_STANDARD 17)

#
# Header-only library
#
add_library(${PROJECT_NAME} INTERFACE)

#
# Add the library interface include directories
#
set(SG14_INCLUDE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/SG14")
include(GNUInstallDirs)

target_include_directories(${PROJECT_NAME} INTERFACE
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
    $<BUILD_INTERFACE:${SG14_INCLUDE_DIRECTORY}>
)

#
# Conditionally build tests
#
if(SG14_BUILD_TESTS)
    enable_testing()

    set(SG14_TEST_SOURCE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/SG14_test")
    set(TEST_SOURCE_FILES
        "${SG14_TEST_SOURCE_DIRECTORY}/main.cpp"
        "${SG14_TEST_SOURCE_DIRECTORY}/flat_map_test.cpp"
        "${SG14_TEST_SOURCE_DIRECTORY}/flat_set_test.cpp"
        "${SG14_TEST_SOURCE_DIRECTORY}/inplace_function_test.cpp"
        "${SG14_TEST_SOURCE_DIRECTORY}/plf_colony_test.cpp"
        "${SG14_TEST_SOURCE_DIRECTORY}/ring_test.cpp"
        "${SG14_TEST_SOURCE_DIRECTORY}/slot_map_test.cpp"
        "${SG14_TEST_SOURCE_DIRECTORY}/uninitialized_test.cpp"
        "${SG14_TEST_SOURCE_DIRECTORY}/unstable_remove_test.cpp"
    )

    add_executable(${PROJECT_NAME}_tests ${TEST_SOURCE_FILES})

    # If tests require threads, link them:
    find_package(Threads REQUIRED)
    target_link_libraries(${PROJECT_NAME}_tests PRIVATE ${PROJECT_NAME} Threads::Threads)

    # Optionally add test to CTest:
    add_test(NAME ${PROJECT_NAME}_tests COMMAND ${PROJECT_NAME}_tests)

    #
    # Example compiler warnings (optional)
    #
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        target_compile_options(${PROJECT_NAME}_tests PRIVATE -Wall -Wextra -Werror)
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
        target_compile_options(${PROJECT_NAME}_tests PRIVATE /W4 /WX)
        add_definitions(-DNOMINMAX -D_SCL_SECURE_NO_WARNINGS)
    endif()
endif()

#
# Installation (headers-only library)
#
install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}_targets)

install(EXPORT ${PROJECT_NAME}_targets
    NAMESPACE ${PROJECT_NAME}::
    FILE ${PROJECT_NAME}-config.cmake
    DESTINATION "${CMAKE_INSTALL_DATADIR}/cmake/${PROJECT_NAME}"
)

install(DIRECTORY "${SG14_INCLUDE_DIRECTORY}" DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")