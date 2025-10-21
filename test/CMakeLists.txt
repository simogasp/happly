cmake_minimum_required(VERSION 3.20)

project(happly-test)

### Policy settings
cmake_policy(SET CMP0054 NEW)   # don't implicitly dereference inside if()

### Configure output locations
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

### Compile options

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(HPLY_CXX_FEATURE c_std_${CMAKE_CXX_STANDARD})

if(CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU")
    set(HPLY_COMPILE_OPTIONS "-Wall;-Wextra;-pedantic;-Wno-comment;-Wshadow;-Wnon-virtual-dtor;-Wpedantic;-Wconversion;-Wmisleading-indentation;-Wsign-conversion;-Wfloat-equal")
    if(ENABLE_WARNINGS_AS_ERRORS)
        list(APPEND HPLY_COMPILE_OPTIONS "-Werror")
    endif()
elseif(MSVC)
    set(HPLY_COMPILE_OPTIONS "/W4;/permissive-;/Zc:__cplusplus")
    if(ENABLE_WARNINGS_AS_ERRORS)
        list(APPEND HPLY_COMPILE_OPTIONS "/WX")
    endif()
endif()
if(APPLE)
    list(APPEND HPLY_COMPILE_OPTIONS "-Wno-deprecated-declarations")
endif()

# compile definitions for all targets depending on the compiler and the system
set(HPLY_COMPILE_DEFINITIONS "")
if(MSVC)
    set(HPLY_COMPILE_DEFINITIONS "-DNOMINMAX;-D_USE_MATH_DEFINES")
endif()
#
#if ("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
#  # using Clang (linux or apple) or GCC
#  message("Using clang/gcc compiler flags")
#  SET(BASE_CXX_FLAGS "-Wall -Wextra -g3")
##  SET(DISABLED_WARNINGS " -Wno-unused-parameter -Wno-unused-variable -Wno-unused-private-field -Wno-unused-function -Wno-deprecated-declarations")
#  SET(TRACE_INCLUDES " -H -Wno-error=unused-command-line-argument")
#
#  # clang-specific things
#  if ("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
#    SET(BASE_CXX_FLAGS "${BASE_CXX_FLAGS} -ferror-limit=5 -fcolor-diagnostics")
#  endif()
#
#  SET(CMAKE_CXX_FLAGS "${BASE_CXX_FLAGS} ${DISABLED_WARNINGS}")
#  #SET(CMAKE_CXX_FLAGS_DEBUG          "-g3 -fsanitize=address") // tests are crashing in gtest with this on OSX. Is something broken or is the address checker being too aggressive with gtest?
#  SET(CMAKE_CXX_FLAGS_DEBUG          "-g3")
#  SET(CMAKE_CXX_FLAGS_MINSIZEREL     "-Os -DNDEBUG")
#  SET(CMAKE_CXX_FLAGS_RELEASE        "-march=native -O3 -DNDEBUG")
#  SET(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O2 -g")
#elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
#  # using Visual Studio C++
#  message("Using Visual Studio compiler flags")
#  set(BASE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
#  set(BASE_CXX_FLAGS "${BASE_CXX_FLAGS} /Zc:__cplusplus")
#  set(BASE_CXX_FLAGS "${BASE_CXX_FLAGS} /MP") # parallel build
#  SET(DISABLED_WARNINGS "${DISABLED_WARNINGS} /wd\"4267\"")  # ignore conversion to smaller type (fires more aggressively than the gcc version, which is annoying)
#  SET(DISABLED_WARNINGS "${DISABLED_WARNINGS} /wd\"4244\"")  # ignore conversion to smaller type (fires more aggressively than the gcc version, which is annoying)
#  SET(DISABLED_WARNINGS "${DISABLED_WARNINGS} /wd\"4305\"")  # ignore truncation on initialization
#  SET(CMAKE_CXX_FLAGS "${BASE_CXX_FLAGS} ${DISABLED_WARNINGS}")
#  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MD")
#  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MDd")
#
#  add_definitions(/D "_CRT_SECURE_NO_WARNINGS")
#  add_definitions (-DNOMINMAX)
#else()
#  # unrecognized
#  message( FATAL_ERROR "Unrecognized compiler [${CMAKE_CXX_COMPILER_ID}]" )
#endif()


# Set up googletest
include(FetchContent)
FetchContent_Declare(
    googletest
    GIT_TAG v1.17.0
    GIT_REPOSITORY https://github.com/google/googletest.git
    FIND_PACKAGE_ARGS 1.16.0 NAMES GTest COMPONENTS gmain_main
    EXCLUDE_FROM_ALL
    SYSTEM
)

# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

# Test executable
add_executable(ply-test main_test.cpp)

target_link_libraries(ply-test GTest::gtest_main)

target_include_directories(ply-test PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/..")
target_compile_options(ply-test PRIVATE ${HPLY_COMPILE_OPTIONS})
target_compile_definitions(ply-test PRIVATE ${HPLY_COMPILE_DEFINITIONS})

# Add cmake test target  ("make test")
list(APPEND CMAKE_CTEST_ARGUMENTS "--output-on-failure")
enable_testing()
add_test(NAME MainTest COMMAND ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/ply-test)

