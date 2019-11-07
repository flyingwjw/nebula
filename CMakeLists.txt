if (WIN32)
    cmake_minimum_required(VERSION 3.1 FATAL_ERROR)
else()
    cmake_minimum_required(VERSION 2.8.0 FATAL_ERROR)
endif()

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release
        CACHE STRING "Set build type to Debug or Release (default Release)" FORCE)
endif()

project(nebula C)

set(CMAKE_MODULE_PATH "cmake/")

option(NEBULA_DISABLE_SAMPLES
        "Disable sample files" ON)

# Put the libaries and binaries that get built into directories at the
# top of the build tree rather than in hard-to-find leaf directories.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)

add_subdirectory(lib)

#################################################
#
# Samples
#
#################################################
#if (NOT NEBULA_DISABLE_SAMPLES)
#    add_subdirectory(sample)
#endif()
add_subdirectory(sample)

message(STATUS "")
message(STATUS "        ---( helloworld " ${HELLOWORLD_VERSION} " )---")
message(STATUS "")
message(STATUS "Available nebula backends: ${BACKENDS}")
message(STATUS "CMAKE_BINARY_DIR:         " ${CMAKE_BINARY_DIR})
message(STATUS "CMAKE_CURRENT_BINARY_DIR: " ${CMAKE_CURRENT_BINARY_DIR})
message(STATUS "CMAKE_SOURCE_DIR:         " ${CMAKE_SOURCE_DIR})
message(STATUS "CMAKE_CURRENT_SOURCE_DIR: " ${CMAKE_CURRENT_SOURCE_DIR})
message(STATUS "PROJECT_BINARY_DIR:       " ${PROJECT_BINARY_DIR})
message(STATUS "PROJECT_SOURCE_DIR:       " ${PROJECT_SOURCE_DIR})
message(STATUS "CMAKE_MODULE_PATH:        " ${CMAKE_MODULE_PATH})
message(STATUS "CMAKE_COMMAND:            " ${CMAKE_COMMAND})
message(STATUS "CMAKE_ROOT:               " ${CMAKE_ROOT} )
message(STATUS "CMAKE_SYSTEM:             " ${CMAKE_SYSTEM} )
message(STATUS "CMAKE_SYSTEM_NAME:        " ${CMAKE_SYSTEM_NAME} )
message(STATUS "CMAKE_SYSTEM_VERSION:     " ${CMAKE_SYSTEM_VERSION} )
message(STATUS "CMAKE_SYSTEM_PROCESSOR:   " ${CMAKE_SYSTEM_PROCESSOR} )
message(STATUS "CMAKE_SKIP_RPATH:         " ${CMAKE_SKIP_RPATH} )
message(STATUS "CMAKE_VERBOSE_MAKEFILE:   " ${CMAKE_VERBOSE_MAKEFILE} )
message(STATUS "CMAKE_C_FLAGS:            " ${CMAKE_C_FLAGS} )
message(STATUS "CMAKE_CXX_FLAGS:          " ${CMAKE_CXX_FLAGS} )
message(STATUS "CMAKE_BUILD_TYPE:         " ${CMAKE_BUILD_TYPE} )
message(STATUS "CMAKE_C_COMPILER:         " ${CMAKE_C_COMPILER} )
message(STATUS "CMAKE_AR:                 " ${CMAKE_AR} )
message(STATUS "CMAKE_RANLIB:             " ${CMAKE_RANLIB} )
message(STATUS "CMAKE_PREFIX_PATH:        " ${CMAKE_PREFIX_PATH} )
message(STATUS "CMAKE_INSTALL_PREFIX:     " ${CMAKE_INSTALL_PREFIX} )
message(STATUS "")
