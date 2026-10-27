# to overwrite outdate geriatrix cmake

cmake_minimum_required (VERSION 3.5)
project (geriatrix C CXX)

include (CheckFunctionExists)

set (CMAKE_CXX_STANDARD 14)
set (CXX_STANDARD_REQUIRED True)

set (CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

if (NOT CMAKE_BUILD_TYPE)
    set (CMAKE_BUILD_TYPE Release
         CACHE STRING "Choose the type of build." FORCE)
    set_property (CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS 
                  "Debug" "Release" "RelWithDebInfo" "MinSizeRel")
endif ()
set (DEBUG_SANITIZER Off CACHE STRING "Sanitizer for debug builds")
set_property (CACHE DEBUG_SANITIZER PROPERTY STRINGS
              "Off" "Address" "Thread")
set (CMAKE_PREFIX_PATH "" CACHE STRING "External dependencies path")


set (as_flags "-fsanitize=address -O1 -fno-omit-frame-pointer")
set (ts_flags "-fsanitize=thread  -O1 -fno-omit-frame-pointer")
if (${CMAKE_BUILD_TYPE} STREQUAL "Debug")
    if (${DEBUG_SANITIZER} STREQUAL "Address")
        set (CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} ${as_flags}")
        set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} ${as_flags}")
    elseif (${DEBUG_SANITIZER} STREQUAL "Thread")
        set (CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} ${ts_flags}")
        set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} ${ts_flags}")
    endif ()
endif ()

set (CMAKE_THREAD_PREFER_PTHREAD TRUE)
set (THREADS_PREFER_PTHREAD_FLAG TRUE)
find_package (Threads REQUIRED)
list (APPEND geriatrix-depends "${CMAKE_THREAD_LIBS_INIT}")

set (Boost_NO_BOOST_CMAKE ON)
find_package(Boost CONFIG)
if (NOT Boost_FOUND AND NOT BOOST_FOUND)
    message ("geriatrix requires boost...  install boost and set")
    message ("CMAKE_PREFIX_PATH to point to boost")
    message (FATAL_ERROR "Aborting...")
endif ()
include_directories (${Boost_INCLUDE_DIRS})

check_function_exists (posix_fallocate HAS_POSIX_FALLOCATE)
if (NOT HAS_POSIX_FALLOCATE)
    add_definitions (-DNEED_POSIX_FALLOCATE)
endif ()

if (DELTAFS)
    find_package (deltafs CONFIG REQUIRED)
    add_definitions (-DDELTAFS)
    list (APPEND geriatrix-depends deltafs)
    list (APPEND geriatrix-drivers src/deltafs_driver.c)
endif ()

add_executable (geriatrix ${geriatrix-drivers} src/geriatrix.cpp)
target_link_libraries (geriatrix ${geriatrix-depends})

install (TARGETS geriatrix RUNTIME DESTINATION bin)