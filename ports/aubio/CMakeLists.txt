cmake_minimum_required(VERSION 3.8)
project(aubio C)

add_definitions(
    -DHAVE_STDLIB_H=1
    -DHAVE_STDIO_H=1
    -DHAVE_MATH_H=1
    -DHAVE_STRING_H=1
    -DHAVE_LIMITS_H=1
    -DHAVE_STDARG_H=1
    -DHAVE_C99_VARARGS_MACROS=1

    -DHAVE_SNDFILE=1
    -DHAVE_WAVWRITE=1
    -DHAVE_WAVREAD=1
    -DHAVE_LIBAV=1
    -DHAVE_SWRESAMPLE=1
)

set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
set(CMAKE_DEBUG_POSTFIX d)

option(BUILD_TOOLS "Build and install tools" ON)
set(TOOLS_INSTALLDIR "bin" CACHE STRING "Target directory for installed tools")

find_path(LIBSNDFILE_H sndfile.h)
find_library(LIBSNDFILE_LIB NAMES libsndfile-1 libsndfile)
find_library(AVCODEC_LIB avcodec)
find_library(AVUTIL_LIB avutil)
find_library(AVDEVICE_LIB avdevice)
find_library(AVFILTER_LIB avfilter)
find_library(AVFORMAT_LIB avformat)
find_library(SWRESAMPLE_LIB swresample)
find_library(OGG_LIB ogg)
find_library(FLAC_LIB flac)
find_library(VORBIS_LIB vorbis)
find_library(VORBISENC_LIB vorbisenc)

include_directories(src ${LIBSNDFILE_H})

file(GLOB_RECURSE SOURCES src/*.c)

set_source_files_properties(src/io/sink_wavwrite.c PROPERTIES COMPILE_FLAGS /FIWinsock2.h)
add_library(aubio ${SOURCES})
target_link_libraries(aubio PUBLIC
    ${LIBSNDFILE_LIB}
    ${OGG_LIB}
    ${FLAC_LIB}
    ${VORBIS_LIB}
    ${VORBISENC_LIB}
    ${AVCODEC_LIB}
    ${AVUTIL_LIB}
    ${AVDEVICE_LIB}
    ${AVFILTER_LIB}
    ${AVFORMAT_LIB}
    ${SWRESAMPLE_LIB}
    ws2_32.lib
)

if(BUILD_TOOLS)
    set(EXAMPLE_EXECS aubiomfcc aubionotes aubioonset aubiopitch aubioquiet aubiotrack)
    foreach(EXAMPLE_EXEC ${EXAMPLE_EXECS})
        add_executable(${EXAMPLE_EXEC} examples/${EXAMPLE_EXEC}.c examples/utils.c examples/jackio.c)
        target_link_libraries(${EXAMPLE_EXEC} PRIVATE aubio)
        target_compile_definitions(${EXAMPLE_EXEC} PRIVATE -DHAVE_WIN_HACKS=1)
    endforeach()

    install(
        TARGETS ${EXAMPLE_EXECS}
        RUNTIME DESTINATION ${TOOLS_INSTALLDIR}
    )
endif()

install(
    TARGETS aubio
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

if(NOT DISABLE_INSTALL_HEADERS)
    install(
        DIRECTORY src/
        DESTINATION include/aubio
        FILES_MATCHING
        PATTERN "*.h"
        PATTERN "*_priv.h" EXCLUDE
        PATTERN "config.h" EXCLUDE
    )
endif()
