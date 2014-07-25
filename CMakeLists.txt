cmake_minimum_required(VERSION 2.8)
project(s2-geometry)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
include(CheckCXXCompilerFlag)
include(FindPackageHandleStandardArgs)
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

find_package(OpenSSL REQUIRED)

message("OpenSSL include dir: ${OPENSSL_INCLUDE_DIR}")
message("OpenSSL libraries: ${OPENSSL_LIBRARIES}")


ADD_DEFINITIONS (-Wno-deprecated-declarations -DNDEBUG -std=c++11)

include_directories(geometry)
include_directories(geometry/util/endian)

file(GLOB S2Glob geometry/s2.cc geometry/s1interval.cc geometry/base/strtoint.cc geometry/util/math/exactfloat/exactfloat.cc geometry/strings/strutil.cc geometry/strings/split.cc geometry/base/stringprintf.cc geometry/s2cap.cc geometry/s1angle.cc geometry/s2cell.cc geometry/s2cellunion.cc geometry/s2cellid.cc geometry/s2edgeindex.cc geometry/s2edgeutil.cc geometry/s2latlngrect.cc geometry/s2loop.cc geometry/s2pointregion.cc geometry/s2latlng.cc geometry/s2polygon.cc geometry/s2polygonbuilder.cc geometry/s2polyline.cc geometry/s2r2rect.cc geometry/s2regionunion.cc geometry/s2region.cc geometry/s2regioncoverer.cc geometry/s2regionintersection.cc geometry/base/logging.cc geometry/util/coding/*.cc)

add_library(s2geo STATIC ${S2Glob})
# Add RPATH info to executables so that when they are run after being installed
# (i.e., from /usr/local/bin/) the linker can find library dependencies. For
# more info see http://www.cmake.org/Wiki/CMake_RPATH_handling
set_property(TARGET s2geo PROPERTY INSTALL_RPATH_USE_LINK_PATH TRUE)

add_library(s2testing STATIC geometry/s2testing.cc)

file(GLOB InstallGlob geometry/*.h)
file(GLOB BaseGlob geometry/base/*.h)
file(GLOB UtilMathGlob geometry/util/math/*.h)
file(GLOB TestGlob geometry/*_test.cc)

install(FILES ${InstallGlob} DESTINATION include/s2geo)
install(FILES ${BaseGlob} DESTINATION include/s2geo/base)
install(FILES ${UtilMathGlob} DESTINATION include/util/math)
install(TARGETS s2geo DESTINATION lib)

if (GTEST_ROOT)
    enable_testing()
    find_package(GTest REQUIRED)
    include_directories(${GTEST_INCLUDE_DIR})


    cmake_policy(SET CMP0037 OLD) # allow underscores in targets
    foreach( testsourcefile ${TestGlob} )
        get_filename_component(testname ${testsourcefile} NAME_WE)
        add_executable( ${testname} ${testsourcefile} )
        target_link_libraries( ${testname} s2geo s2testing ${OPENSSL_LIBRARIES} ${GTEST_BOTH_LIBRARIES} )
    endforeach()
endif()
