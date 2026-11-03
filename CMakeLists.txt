cmake_minimum_required(VERSION 4.0.0)

project(Jwt LANGUAGES CXX)

set(JWT_VERSION "0.1.0")
set(JWT_ROOT_DIR ${CMAKE_CURRENT_LIST_DIR})
set(JWT_INCLUDE_DIR ${JWT_ROOT_DIR}/include)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
add_custom_target(
    copy-compile-commands ALL
    ${CMAKE_COMMAND} -E copy_if_different
        ${CMAKE_BINARY_DIR}/compile_commands.json
        ${JWT_ROOT_DIR}/build/compile_commands.json
    )


include(cmake/CPM.cmake)
CPMAddPackage(
    NAME googletest
    GIT_REPOSITORY "https://github.com/google/googletest"
    GIT_TAG "v1.17.0"
    OPTIONS
        "INSTALL_GTEST OFF"
        "BUILD_GMOCK OFF"
)

CPMAddPackage(
    NAME argparse
    GIT_REPOSITORY "https://github.com/p-ranav/argparse"
    GIT_TAG "v3.2"
)

find_package(OpenSSL QUIET)
if(NOT OpenSSL_FOUND)
    message(STATUS "OpenSSL not found on system. Building from source via CPM...")

    CPMAddPackage(
        NAME openssl
        GIT_REPOSITORY "https://github.com/jimmy-park/openssl-cmake"
        GIT_TAG "4.1.0"
        OPTIONS
            "OPENSSL_BUILD_TESTS OFF"
    )
endif()


file(GLOB_RECURSE LibrarySources src/lib/**.cpp)
file(GLOB_RECURSE LibraryHeaders include/**.h)
file(GLOB_RECURSE UtilSources src/util/**.cpp)
file(GLOB_RECURSE TestSources src/test/**.cpp)

add_compile_definitions(JWT_VERSION=\"${JWT_VERSION}\")

add_library(JwtLib STATIC ${LibrarySources})
target_include_directories(JwtLib PUBLIC ${JWT_INCLUDE_DIR})
target_link_libraries(JwtLib OpenSSL::Crypto)
target_precompile_headers(JwtLib PUBLIC ${LibraryHeaders})

add_executable(JwtUtil ${UtilSources})
target_include_directories(JwtUtil PUBLIC ${JWT_INCLUDE_DIR})
target_link_libraries(JwtUtil PRIVATE JwtLib)
target_link_libraries(JwtUtil PRIVATE argparse)

add_executable(JwtTest ${TestSources})
target_include_directories(JwtTest PUBLIC ${JWT_INCLUDE_DIR})
target_link_libraries(JwtTest PRIVATE JwtLib)
target_link_libraries(JwtTest PRIVATE gtest_main)


