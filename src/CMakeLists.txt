cmake_minimum_required(VERSION 2.8)

message("")
message("cmake options:")
message("  -DCMAKE_BUILD_TYPE='${CMAKE_BUILD_TYPE}': Build debug or release. (Debug|Release)")
message("  -DCMAKE_VERBOSE='${CMAKE_VERBOSE}': Spew cmake project options. (On|Off)")
message("  -DBUILD_X64='${BUILD_X64}': Build 32 or 64-bit. (On|Off)")
message("")

#
# voglperf
#
project(voglperf)
find_package(Threads REQUIRED)

include("${CMAKE_SOURCE_DIR}/build_options.cmake")

set(SRC_LIST voglperf.c)

add_compiler_flag("-fno-exceptions")

add_shared_linker_flag("-Wl,--version-script=${PROJECT_SOURCE_DIR}/libvoglperf_linker_script.txt")

# We need a DT_NEEDED entry for libGL so it will get loaded in our namespace under us and we need
#  to make sure the linker doesn't remove it since we're not actually calling any libGL functions.
add_shared_linker_flag("-Wl,--no-as-needed")

add_library(${PROJECT_NAME} SHARED ${SRC_LIST})

set(CMAKE_LIBRARY_ARCHITECTURE ${LIBDIR})

find_library(OPENGL_gl_LIBRARY
    NAMES GL MesaGL libGL.so.1
    PATHS /opt/graphics/OpenGL/lib
          /usr/openwin/lib
          /usr/shlib
          /usr/X11R6/lib
    )
if (NOT OPENGL_gl_LIBRARY)
    message(FATAL_ERROR "Find_package failed to locate ${LIBDIR} libGL.so.")
endif()

target_link_libraries(${PROJECT_NAME}
    dl
    rt
    m
    ${CMAKE_THREAD_LIBS_INIT}
    ${OPENGL_gl_LIBRARY}
    )

build_options_finalize()

#
# voglperfrun
#
project(voglperfrun)

# Add libedit library.
include_directories(${CMAKE_SOURCE_DIR}/libedit-3.1/include)
set(LIBEDIT_LIBS
    "${CMAKE_SOURCE_DIR}/libedit-3.1/lib/${LIBDIR}/libedit.a"
    tinfo
    )

set(SRC_LIST
    voglperfrun.cpp
    voglutils.cpp
    webby/webby.c)

# Copy index.html to our build directory and add it to our binary.
#  http://stackoverflow.com/questions/4864866/c-c-with-gcc-statically-add-resource-files-to-executable-library
configure_file(index.html index.html COPYONLY)
add_linker_flag("-Wl,--format=binary -Wl,index.html -Wl,--format=default")

add_executable(${PROJECT_NAME} ${SRC_LIST})

target_link_libraries(${PROJECT_NAME}
    rt
    ${CMAKE_THREAD_LIBS_INIT}
    ${LIBEDIT_LIBS}
    )

build_options_finalize()

