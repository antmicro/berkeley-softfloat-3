cmake_minimum_required(VERSION 3.6)

project(softfloat-3)

set(SPECIALIZED_ARCHITECTURE ${SOFTFLOAT_SPECIALIZED_ARCHITECTURE} CACHE STRING "The architecture that Softfloat mimics.")

if(NOT SPECIALIZED_ARCHITECTURE)
    message(FATAL_ERROR "Specialized architecture is not set.")
endif()

option(TLIB_PROFILING_BUILD "Build optimized for profiling" OFF)

add_definitions(
    -fPIC
    -Werror-implicit-function-declaration
)
add_compile_definitions(
    SOFTFLOAT_ROUND_ODD
    SOFTFLOAT_FAST_INT64
    SOFTFLOAT_FAST_DIV32TO16
    SOFTFLOAT_FAST_DIV64TO32
)

file(GLOB SOURCES source/*.c)
file(GLOB SPECIALIZED_SOURCES source/${SPECIALIZED_ARCHITECTURE}/*.c)
list(APPEND SOURCES ${SPECIALIZED_SOURCES})

# Exclude non fast int64
list(FILTER SOURCES EXCLUDE REGEX ".*\/s_.*128M.9*\.c")
list(FILTER SOURCES EXCLUDE REGEX ".*\/s_.*128SigM.*\.c")
list(FILTER SOURCES EXCLUDE REGEX ".*\/s_.*80M.*\.c")
list(FILTER SOURCES EXCLUDE REGEX ".*\/s_.*M\.c")

# Exclude unused formats
list(FILTER SOURCES EXCLUDE REGEX ".*\/.*bf16.*\.c")
list(FILTER SOURCES EXCLUDE REGEX ".*\/.*F80.*\.c")
list(FILTER SOURCES EXCLUDE REGEX ".*\/.*F128.*\.c")
list(FILTER SOURCES EXCLUDE REGEX ".*\/.*f128.*\.c")
list(FILTER SOURCES EXCLUDE REGEX ".*\/.*BF16.*\.c")

if(TLIB_PROFILING_BUILD)
    add_definitions(
        -fno-omit-frame-pointer
        -g3
    )
elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
    add_definitions(
        -fomit-frame-pointer
    )
    add_compile_definitions(
        INLINE_LEVEL=5
    )
endif()

add_library(
    softfloat-3 STATIC
    ${SOURCES}
)
target_include_directories(
    softfloat-3 PRIVATE
    build/Linux-x86_64-GCC
    source/${SPECIALIZED_ARCHITECTURE}
)
target_include_directories(
    softfloat-3 PUBLIC
    source/include
)
