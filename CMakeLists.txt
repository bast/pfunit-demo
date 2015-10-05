cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

project(pfunit-demo)

enable_language(Fortran)

if(CMAKE_Fortran_COMPILER_ID MATCHES GNU)
    set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -g -fprofile-arcs -ftest-coverage")
endif()

add_library(demo-sources-to-test hello.F90)

if(DEFINED PFUNIT_INSTALL)
    message(STATUS "Manual setup of variable PFUNIT_INSTALL : ${PFUNIT_INSTALL}")
    set(PFUNIT_DIR ${PFUNIT_INSTALL})
else()
    add_custom_target(
        git_update
        COMMAND git submodule init
        COMMAND git submodule update
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
        )

    include(ExternalProject)

    file(MAKE_DIRECTORY ${PROJECT_BINARY_DIR}/generated)

    set(ExternalProjectCMakeArgs
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_INSTALL_PREFIX=${PROJECT_BINARY_DIR}/external/pfunit
        -DCMAKE_Fortran_COMPILER=${CMAKE_Fortran_COMPILER}
        )
    ExternalProject_Add(pfunit
        DOWNLOAD_COMMAND git submodule update
        DOWNLOAD_DIR ${PROJECT_SOURCE_DIR}
        SOURCE_DIR ${PROJECT_SOURCE_DIR}/external/pfunit
        BINARY_DIR ${PROJECT_BINARY_DIR}/external/pfunit-build
        STAMP_DIR ${PROJECT_BINARY_DIR}/external/pfunit-stamp
        TMP_DIR ${PROJECT_BINARY_DIR}/external/pfunit-tmp
        INSTALL_DIR ${PROJECT_BINARY_DIR}/external
        CMAKE_ARGS ${ExternalProjectCMakeArgs}
        )
    include_directories(${PROJECT_BINARY_DIR}/external/pfunit/mod)
    add_dependencies(pfunit git_update)
    set(PFUNIT_DIR ${PROJECT_BINARY_DIR}/external/pfunit)
endif()

include_directories(${PFUNIT_DIR}/mod)

include_directories(
    ${PROJECT_SOURCE_DIR}
    ${PROJECT_BINARY_DIR}/generated
    )

file(WRITE ${PROJECT_BINARY_DIR}/generated/testSuites.inc "")
set(_test_sources)
foreach(_test
    test_add_numbers
    test_subtract_numbers
    )
    if (DEFINED PFUNIT_INSTALL)
        set(test_dependency ${PROJECT_SOURCE_DIR}/${_test}.F90)
    else()
        set(test_dependency pfunit ${PROJECT_SOURCE_DIR}/${_test}.F90)
    endif()
    add_custom_command(
        OUTPUT ${PROJECT_BINARY_DIR}/generated/${_test}.F90
        COMMAND python ${PFUNIT_DIR}/bin/pFUnitParser.py ${PROJECT_SOURCE_DIR}/${_test}.F90 ${PROJECT_BINARY_DIR}/generated/${_test}.F90
        DEPENDS ${test_dependency}
        )
    set(_test_sources ${_test_sources} ${PROJECT_BINARY_DIR}/generated/${_test}.F90)
    file(APPEND ${PROJECT_BINARY_DIR}/generated/testSuites.inc "ADD_TEST_SUITE(${_test}_suite)\n")
endforeach()

set_source_files_properties(${PFUNIT_DIR}/include/driver.F90 PROPERTIES GENERATED 1)

add_executable(
    pftest_alltests
    ${PFUNIT_DIR}/include/driver.F90
    ${_test_sources}
    )
target_link_libraries(
    pftest_alltests
    ${PFUNIT_DIR}/lib/libpfunit.a
    demo-sources-to-test
    )

enable_testing()
add_test(pftest_alltests ${PROJECT_BINARY_DIR}/pftest_alltests)
