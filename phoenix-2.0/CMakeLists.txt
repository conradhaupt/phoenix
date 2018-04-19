cmake_minimum_required(VERSION 3.9)
project(phoenix2 VERSION 2 LANGUAGES C)

# Necessary for CMAKE_INSTALL_{LIB,BIN,INCLUDE}DIR
include(GNUInstallDirs)

# Default to Release build type
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif ()

add_library(phoenix2
    src/synch.c
    src/scheduler.c
    src/map_reduce.c
    src/memory.c
    src/processor.c
    src/pt_mutex.c
    src/locality.c
    src/mcs.c
    src/taskQ.c
    src/tpool.c
    src/iterator.c
)

# Add include directories and make them public
target_include_directories(phoenix2 PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                           $<INSTALL_INTERFACE:include>
                           PRIVATE src include)

# Threading is required for pthreads
find_package(Threads REQUIRED)
target_link_libraries(phoenix2 ${CMAKE_THREAD_LIBS_INIT})

# If we are running under Linux, declare _LINUX_ preprocessor definition
if (UNIX AND NOT APPLE)
    target_compile_definitions(phoenix2 PUBLIC _LINUX_)
endif ()

target_compile_options(phoenix2 PRIVATE
                       -Wall
                       $<$<CONFIG:RELEASE>:-O3>)


EXECUTE_PROCESS(COMMAND uname -m COMMAND tr -d '\n' OUTPUT_VARIABLE ARCHITECTURE)
message(STATUS "Architecture: ${ARCHITECTURE}")
if (${ARCHITECTURE} STREQUAL "x86_64")
    target_compile_options(phoenix2 PUBLIC -D__x86_64__)
endif ()

install(TARGETS phoenix2
        EXPORT Phoenix2Config
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        )

install(DIRECTORY ./include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(EXPORT Phoenix2Config DESTINATION share/Phoenix2/cmake)
export(TARGETS phoenix2 FILE Phoenix2Config.cmake)