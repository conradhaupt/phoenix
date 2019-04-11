cmake_minimum_required(VERSION 3.9)
project(phoenix_word_count LANGUAGES CXX)

# Necessary for CMAKE_INSTALL_{LIB,BIN,INCLUDE}DIR
include(GNUInstallDirs)

# Default to Release build type
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

add_executable(word_count word_count.cpp)

target_link_libraries(word_count PUBLIC phoenix++)
