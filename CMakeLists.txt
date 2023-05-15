cmake_minimum_required(VERSION 3.8)
project(inotify-cpp)

if(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 17)
endif()
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_EXAMPLE "Build inotify-cpp example program" OFF)
option(BUILD_TEST "Build inotify-cpp unittest program" OFF)
option(BUILD_SHARED_LIBS "Build inotify-cpp as a shared library" OFF)
option(BUILD_STATIC_LIBS "Build inotify-cpp as a static library" ON)
option(USE_BOOST_FILESYSTEM "Build with boost::filesystem" OFF)
if(NOT DEFINED BIMAP_INCLUDE_DIR)
    set(BIMAP_INCLUDE_DIR /home/clint/code/temp/bimap)
endif(NOT DEFINED BIMAP_INCLUDE_DIR)
# include(ExternalProject)
# ExternalProject_Add(
#     bimap
#     PREFIX bimap
#     GIT_REPOSITORY https://github.com/LarsHagemann/bimap.git
#     GIT_SHALLOW TRUE
#     CONFIGURE_HANDLED_BY_BUILD ON
#     INSTALL_COMMAND ""
#     LOG_DOWNLOAD ON
# )

if(USE_BOOST_FILESYSTEM)
    list(APPEND USED_BOOST_LIBS filesystem)
    find_package(Boost 1.54.0 COMPONENTS ${USED_BOOST_LIBS} REQUIRED)
endif()
if(BUILD_TEST)
    list(APPEND USED_BOOST_LIBS unit_test_framework)
endif()

#find_package(Boost 1.54.0 COMPONENTS ${USED_BOOST_LIBS} REQUIRED)

add_library(inotify-filesystem-adapter INTERFACE)
if(CMAKE_CXX_STANDARD LESS 17 OR USE_BOOST_FILESYSTEM)
    add_definitions(-DUSE_BOOST_FILESYSTEM)
    target_link_libraries(inotify-filesystem-adapter INTERFACE Boost::filesystem)
else()
    target_link_libraries(inotify-filesystem-adapter INTERFACE stdc++fs)
endif()
target_include_directories(inotify-filesystem-adapter INTERFACE 
    ${BIMAP_INCLUDE_DIR}
    #$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/bimap/src/bimap>
    #$<INSTALL_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/bimap/src/bimap>
)

add_subdirectory(src)

if(BUILD_EXAMPLE)
    add_subdirectory(example)
endif()

if(BUILD_TEST)
    enable_testing()
    add_subdirectory(test)
endif()

message(STATUS "")
message(STATUS "")
message(STATUS "${PROJECT_NAME} configuration summary:")
message(STATUS "")
message(STATUS "  CMake build type ................ : ${CMAKE_BUILD_TYPE}")
message(STATUS "  Build shared libs  .............. : ${BUILD_SHARED_LIBS}")
message(STATUS "  Build static libs  .............. : ${BUILD_STATIC_LIBS}")
message(STATUS "  Build example  .................. : ${BUILD_EXAMPLE}")
message(STATUS "  Build test ...................... : ${BUILD_TEST}")
message(STATUS "  Build c++ standard .............. : ${CMAKE_CXX_STANDARD}")
message(STATUS "  Build with boost::filesystem .... : ${USE_BOOST_FILESYSTEM}")
message(STATUS "")
message(STATUS " Dependencies:")
message(STATUS "  Boost version.................... : ${Boost_VERSION}")
message(STATUS "  Boost root....................... : ${Boost_DIR}")
message(STATUS "")
