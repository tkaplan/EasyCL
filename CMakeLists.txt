cmake_minimum_required(VERSION 2.6)

if( "${CMAKE_SYSTEM_NAME}" STREQUAL "Linux" )
  set( ON_LINUX 1 )
endif()

if( ON_LINUX )
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
endif()

if( ON_LINUX )
    set(TEST_CLBLAS "No" CACHE BOOL "Run the clBLAS GEMM test?  clBlas should be at ../clBLAS, and built into ../clBLAS/src/build")
endif()

if (NOT CMAKE_BUILD_TYPE)
    message("Setting build type to 'Release'")
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif()

include_directories(.)

add_library( CLEW thirdparty/clew/src/clew.c )
IF( CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64" )
  SET_TARGET_PROPERTIES( CLEW PROPERTIES COMPILE_FLAGS "-fPIC")
ENDIF( CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64" )

add_library( EasyCL SHARED EasyCL.cpp CLKernel.cpp platforminfo_helper.cpp deviceinfo_helper.cpp )
target_link_libraries( EasyCL CLEW )
if( ON_LINUX )
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
target_link_libraries( EasyCL dl )
endif()

add_executable( gpuinfo gpuinfo.cpp platforminfo_helper.cpp deviceinfo_helper.cpp )
target_link_libraries( gpuinfo CLEW )
if( ON_LINUX )
target_link_libraries( gpuinfo dl )
endif()

if( ON_LINUX )
add_library( gtest SHARED thirdparty/gtest/gtest-all.cc )
target_link_libraries( gtest pthread)
else()
add_library( gtest thirdparty/gtest/gtest-all.cc )
endif()

if( TEST_CLBLAS )
    set( CLBLAS_TEST_SOURCES test/testclblas.cpp )
endif()

add_executable( unittests test/testscalars.cpp test/testintarray.cpp test/testfloatwrapper.cpp
    test/testclarray.cpp test/testfloatwrapperconst.cpp test/testintwrapper.cpp test/test_scenario_te42kyfo.cpp
    test/testfloatarray.cpp test/testeasycl.cpp test/testinout.cpp test/testintwrapper_huge.cpp
    test/testlocal.cpp ../test/testdefines.cpp test/testbuildlog.cpp test/testnewinstantiations.cpp
    test/testucharwrapper.cpp test/testkernelstore.cpp
    test/asserts.cpp thirdparty/gtest/gtest_main.cc ${CLBLAS_TEST_SOURCES} )
target_link_libraries( unittests gtest EasyCL CLEW )
target_include_directories( unittests PRIVATE thirdparty/gtest )

if( TEST_CLBLAS )
    target_include_directories( unittests PRIVATE ../clBLAS/src )
    target_link_libraries( unittests ${CMAKE_CURRENT_SOURCE_DIR}/../clBLAS/src/build/library/libclBLAS.so)
endif()

execute_process( COMMAND gitx --version RESULT_VARIABLE git_test ERROR_QUIET )
if( ${git_test} EQUAL 0 )
    set( GIT_FOUND 1 )
endif()
if( GIT_FOUND )
message("git found")
endif()

if( GIT_FOUND )
add_custom_target(
    git-submodule-update
    git submodule update --init --recursive
)
add_dependencies( EasyCL git-submodule-update )
endif()

#if( ON_LINUX )
#add_custom_target( 
#    clfiles
#    cp ${CMAKE_CURRENT_SOURCE_DIR}/test/*.cl ${CMAKE_BINARY_DIR}
#)
#add_dependencies( unittests clfiles )
#else()
# No idea how to do this on visual studio/windows
# If you have any ideas :-)
#add_custom_target( 
#    clfiles
#    copy ${CMAKE_CURRENT_SOURCE_DIR}\\tests\\*.cl ${CMAKE_BINARY_DIR}
#)
#endif()


