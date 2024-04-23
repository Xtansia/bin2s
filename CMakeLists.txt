cmake_minimum_required(VERSION 3.5)
project(bin2s)

set(CMAKE_CXX_STANDARD 17)

if (${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU" OR ${CMAKE_CXX_COMPILER_ID} MATCHES "Clang")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wpedantic")
elseif (${CMAKE_CXX_COMPILER_ID} STREQUAL "MSVC")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4 /MP")
endif()

if (${CMAKE_CXX_COMPILER_ID} MATCHES "AppleClang")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
endif()

add_executable(bin2s src/bin2s.cpp)
target_include_directories(bin2s SYSTEM PRIVATE deps/args)

install(TARGETS bin2s RUNTIME DESTINATION bin)
