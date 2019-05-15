cmake_minimum_required(VERSION 3.5)
project( bitwyre_server )

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -Wall -pedantic -Wextra -Wno-missing-field-initializers")
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/CMakeModules)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

if(CMAKE_BUILD_TYPE MATCHES "Debug")
    set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -g -O0 -fprofile-arcs -ftest-coverage")
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake)

option(BUILD_DOCS "build docs alongside the project" OFF)

find_program(CTEST_MEMORYCHECK_COMMAND NAMES valgrind)
find_program(CTEST_COVERAGE_COMMAND NAMES gcov)
find_program(CMAKE_CXX_CPPCHECK NAMES cppcheck)

if (CMAKE_CXX_CPPCHECK)
    message("-- Cppcheck found " ${CMAKE_CXX_CPPCHECK})
    list(
            APPEND CMAKE_CXX_CPPCHECK
            "--enable=all"
            "--suppress=*:${PROJECT_SOURCE_DIR}/third_party*"
            "--suppress=*:${PROJECT_SOURCE_DIR}/tests*"
    )
else()
    message("-- Cppcheck not found")
    set(CMAKE_CXX_CPPCHECK "")
endif()

INCLUDE(Dart)


include_directories( ${PROJECT_SOURCE_DIR}/include )

file(GLOB server_SOURCES
    "${PROJECT_SOURCE_DIR}/src/main.cpp"
)

add_executable( server ${server_SOURCES} )


find_package(Boost COMPONENTS system regex thread REQUIRED)

if (Boost_FOUND)
    include_directories(${Boost_INCLUDEDIR})

    set(Boost_USE_STATIC_LIBS        ON)
    set(Boost_USE_MULTITHREADED      ON)
    set(Boost_USE_STATIC_RUNTIME    OFF)

    target_link_libraries(server Boost::regex
                                 Boost::thread
                                 Boost::system)

else(Boost_FOUND)
    message(FATAL_ERROR "Boost not found. Could not build this app.")
endif(Boost_FOUND)

target_link_libraries( server pthread )

if (BUILD_TESTS)
    find_package(GTest)
    if (GTEST_FOUND)
        include_directories(${GTEST_INCLUDE_DIRS})
    else()
        ADD_SUBDIRECTORY (third_party/googletest)
        include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})
    endif()

    enable_testing()
    add_subdirectory(tests)
endif()

if (BUILD_DOCS)

    find_package(Doxygen
            REQUIRED dot
            OPTIONAL_COMPONENTS mscgen dia)

    if (DOXYGEN_FOUND)

        set(DOXYGEN_IN ${CMAKE_SOURCE_DIR}/docs/Doxyfile.in)
        set(DOXYGEN_OUT ${CMAKE_CURRENT_BINARY_DIR}/docs/Doxyfile)

        configure_file(${DOXYGEN_IN} ${DOXYGEN_OUT} @ONLY)

        add_custom_target(docs ALL
                COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_OUT}
                WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/docs
                COMMENT "Generating doxygen documentation"
                VERBATIM)

        message("-- Adding doxygen documentation done")

    else(DOXYGEN_FOUND)
        message(FATAL_ERROR "Doxygen need to be installed to generate the doxygen documentation")
    endif(DOXYGEN_FOUND)

endif()
