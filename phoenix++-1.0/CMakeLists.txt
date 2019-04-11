cmake_minimum_required(VERSION 3.9)
project(phoenix++ LANGUAGES CXX)

# Necessary for CMAKE_INSTALL_{LIB,BIN,INCLUDE}DIR
include(GNUInstallDirs)

# Default to Release build type
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

add_library(phoenix++ src/task_queue.cpp src/thread_pool.cpp)

# Phoenix++ is programmed badly, requires -fpermissive. Make sure it is public
# so that it affects the running of other code or any program that depends on it
# given that some incorrect code is in the headers.
target_compile_options(phoenix++ PUBLIC "-fpermissive")

# Add include directories and make them public
target_include_directories(
  phoenix++
  PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
         $<INSTALL_INTERFACE:include>
  PRIVATE src)

# Threading is required for pthreads
find_package(Threads REQUIRED)
target_link_libraries(phoenix++ ${CMAKE_THREAD_LIBS_INIT})

# If we are running under Linux, add rt library and declare _LINUX_ preprocessor
if(UNIX AND NOT APPLE)
  target_link_libraries(phoenix++ rt)
  target_compile_definitions(phoenix++ PUBLIC MMAP_POPULATE _LINUX_)
endif()

target_compile_options(phoenix++
                       PRIVATE -fstrict-aliasing
                               -Wall
                               -Wstrict-aliasing
                               $<$<CONFIG:RELEASE>:-O3>)

execute_process(COMMAND uname -m
                COMMAND tr -d '\n'
                OUTPUT_VARIABLE ARCHITECTURE)
message(STATUS "Architecture: ${ARCHITECTURE}")
if(${ARCHITECTURE} STREQUAL "x86_64")
  target_compile_options(phoenix++ PUBLIC -D__x86_64__)
endif()

install(TARGETS phoenix++
        EXPORT PhoenixppConfig
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(DIRECTORY ./include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(EXPORT PhoenixppConfig DESTINATION share/Phoenixpp/cmake)
export(TARGETS phoenix++ FILE PhoenixppConfig.cmake)

# Add test subdirectory
add_subdirectory(tests)
