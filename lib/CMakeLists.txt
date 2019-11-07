set(HELLOWORLD_VERSION_MAJOR 1)
set(HELLOWORLD_VERSION_MINOR 0)
set(HELLOWORLD_VERSION_PATCH 0)
set(HELLOWORLD_VERSION_STAGE beta)

set(HELLOWORLD_ABI_MAJOR ${HELLOWORLD_VERSION_MAJOR})
set(HELLOWORLD_ABI_MINOR ${HELLOWORLD_VERSION_MINOR})
set(HELLOWORLD_ABI_PATCH ${HELLOWORLD_VERSION_PATCH})

set(HELLOWORLD_ABI_LIBVERSION
        "${HELLOWORLD_ABI_MAJOR}.${HELLOWORLD_ABI_MINOR}.${HELLOWORLD_ABI_PATCH}")

set(HELLOWORLD_PACKAGE_VERSION
        "${HELLOWORLD_VERSION_MAJOR}.${HELLOWORLD_VERSION_MINOR}.${HELLOWORLD_VERSION_PATCH}")

option(HELLOWORLD_BUILD_SHARED_LIBRARIES
        "Define if libhelloworld should be built with shared libraries instead of archives" ON)

option(HELLOWORLD_ENABLE_VERBOSE_DEBUG
        "Debug(-DUSE_DEBUG=1)" OFF)

# only a subset of names can be used, defaults to "beta"
set(HELLOWORLD_STAGE_NAME ${HELLOWORLD_VERSION_STAGE}
        CACHE STRING "set the stage name (rc|beta|alpha|release)")

# a list that defines what can set for HELLOWORLD_STAGE_VERSION
set(HELLOWORLD_ALLOWED_STAGE_NAMES
        rc
        beta
        alpha
        release)

# attempt to find the HELLOWORLD_STAGE_VERSION in the allowed list
# of accepted stage names, the return value is stord in
# HELLOWORLD_STAGE_RET

list(FIND HELLOWORLD_ALLOWED_STAGE_NAMES
        ${HELLOWORLD_STAGE_NAME}
        HELLOWORLD_STAGE_RET)

if (HELLOWORLD_STAGE_RET EQUAL "-1")
    set(HELLOWORLD_STAGE_NAME beta)
endif()

set(HELLOWORLD_VERSION
        "${HELLOWORLD_VERSION_MAJOR}.${HELLOWORLD_VERSION_MINOR}.${HELLOWORLD_VERSION_PATCH}-${HELLOWORLD_STAGE_NAME}")

if (HELLOWORLD_ENABLE_VERBOSE_DEBUG)
    add_definitions(-DUSE_DEBUG=1)
endif()

# GCC specific options.
if (CMAKE_COMPILER_IS_GNUCC)
    option(HELLOWORLD_DISABLE_GCC_WARNINGS "Disable verbose warnings with GCC" OFF)
    option(HELLOWORLD_ENABLE_GCC_HARDENING "Enable compiler security checks" OFF)
    option(HELLOWORLD_ENABLE_GCC_FUNCTION_SECTIONS "Enable gcc function sections" OFF)
    option(HELLOWORLD_ENABLE_GCC_WARNINGS "Make all GCC warnings into errors" OFF)

    list(APPEND __FLAGS -Wall -Wswitch)

    if (HELLOWORLD_DISABLE_GCC_WARNINGS)
        list(APPEND __FLAGS -w)
    endif()

    if (HELLOWORLD_ENABLE_GCC_HARDENING)
        list(APPEND __FLAGS
                -fstack-protector-all
                -fwrapv
                -fPIE
                -Wstack-protector
                "--param ssp-buffer-size=1")

        add_definitions(-D_FORTIFY_SOURCE=2)
    endif()

    if (HELLOWORLD_ENABLE_GCC_FUNCTION_SECTIONS)
        list(APPEND __FLAGS -ffunction-sections)
        # TODO: Add --gc-sections support. We need some checks for NetBSD to ensure this works.
    endif()

    if (HELLOWORLD_ENABLE_GCC_WARNINGS)
        list(APPEND __FLAGS -Werror)
    endif()

    # We need to test for at least gcc 2.95 here, because older versions don't
    # have -fno-strict-aliasing
    list(APPEND __FLAGS -fno-strict-aliasing)

    add_compile_options(${__FLAGS})
endif()

# Group the source files.
set(HDR_PUBLIC
        include/hello.h
        include/nebula.h
        ${CMAKE_CURRENT_BINARY_DIR}/include/nebula-config.h
        )

set(HDR_PRIVATE
        include/utils.h
        )

set(SRC_CORE
        helloworld/hello.c
        )

add_definitions(-DHAVE_CONFIG_H)

# We use BEFORE here so we don't accidentally look in system directories
# first for some previous versions of the headers that are installed.
include_directories(BEFORE include)

source_group("Headers Public"   FILES ${HDR_PUBLIC})
source_group("Headers Private"  FILES ${HDR_PRIVATE})
source_group("Source Core"      FILES ${SRC_CORE})

# Generate the configure headers.
# (Place them in the build dir so we don't polute the source tree with generated files).
include_directories(BEFORE ${CMAKE_CURRENT_BINARY_DIR}/include)

if (HELLOWORLD_BUILD_SHARED_LIBRARIES)
    set(HELLOWORLD_LIBRARY_TYPE SHARED)

    if ((CMAKE_COMPILER_IS_GNUCC) OR (${CMAKE_C_COMPILER_ID} STREQUAL "Clang"))
        add_compile_options(-fvisibility=hidden)
    elseif ("${CMAKE_C_COMPILER_ID}" STREQUAL "SunPro")
        add_compile_options(-xldscope=hidden)
    endif()

    set(NEBULA_NEED_DLLEXPORT 1)
else (HELLOWORLD_BUILD_SHARED_LIBRARIES)
    set(HELLOWORLD_LIBRARY_TYPE STATIC)
endif (HELLOWORLD_BUILD_SHARED_LIBRARIES)

# Generate the configure headers.
configure_file(
        nebula-config.h.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/include/nebula-config.h
        NEWLINE_STYLE UNIX)

#################################################
#
# Create the libraries.
#
#################################################
add_library(helloworld ${HELLOWORLD_LIBRARY_TYPE}
        ${HDR_PUBLIC}
        ${SRC_CORE})
set_target_properties(helloworld PROPERTIES DEFINE_SYMBOL NEBULA_LIB_DLL)

if (HELLOWORLD_BUILD_SHARED_LIBRARIES)
    set_target_properties(helloworld
            PROPERTIES SOVERSION ${HELLOWORLD_ABI_LIBVERSION})
endif (HELLOWORLD_BUILD_SHARED_LIBRARIES)

# Define the public headers.
set_target_properties(helloworld
        PROPERTIES PUBLIC_HEADER "${HDR_PUBLIC}")

# Export targets (This is used for other CMake projects to easily find the libraries and include files).
export(TARGETS helloworld
        FILE "${PROJECT_BINARY_DIR}/LibhelloworldTargets.cmake")
export(PACKAGE helloworld)

#################################################
#
# Installation preparation.
#
#################################################

# Allow the user to override installation directories.
set(HELLOWORLD_INSTALL_LIB_DIR       lib CACHE PATH "Installation directory for libraries")
set(HELLOWORLD_INSTALL_BIN_DIR       bin CACHE PATH "Installation directory for executables")
set(HELLOWORLD_INSTALL_INCLUDE_DIR include CACHE PATH "Installation directory for header files")

if(WIN32 AND NOT CYGWIN)
    set(DEF_INSTALL_CMAKE_DIR lib/cmake)
else()
    set(DEF_INSTALL_CMAKE_DIR lib/cmake/helloworld)
endif()

set(HELLOWORLD_INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH "Installation directory for CMake files")

# Make sure the paths are absolute.
foreach(p include cmake)
    set(var HELLOWORLD_INSTALL_${p}_DIR)
    if(NOT IS_ABSOLUTE "${${var}}")
        set(${var}_FULL "${CMAKE_INSTALL_PREFIX}/${${var}}")
    endif()
endforeach()

# Generate the config file for the build-tree.
set(HELLOWORLD_INCLUDE_DIRS
        "include"
        "${PROJECT_BINARY_DIR}/lib/include")

#set(LIBHELLOWORLD_INCLUDE_DIRS
#        ${HELLOWORLD_INCLUDE_DIRS}
#        CACHE PATH "Libhelloworld include directories")

configure_file(cmake/LibhelloworldConfigBuildTree.cmake.in
        ${PROJECT_BINARY_DIR}/LibhelloworldConfig.cmake
        @ONLY)

configure_file(cmake/LibhelloworldConfig.cmake.in
        ${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/LibhelloworldConfig.cmake
        @ONLY)

# Generate version info for both build-tree and install-tree.
configure_file(cmake/LibhelloworldConfigVersion.cmake.in
        ${PROJECT_BINARY_DIR}/LibhelloworldConfigVersion.cmake
        @ONLY)

#################################################
#
# Installation.
#
#################################################

#
# Install targets.
#
install(TARGETS helloworld
        EXPORT LibhelloworldTargets
        RUNTIME DESTINATION "${HELLOWORLD_INSTALL_BIN_DIR}" COMPONENT bin
        LIBRARY DESTINATION "${HELLOWORLD_INSTALL_LIB_DIR}" COMPONENT lib
        ARCHIVE DESTINATION "${HELLOWORLD_INSTALL_LIB_DIR}" COMPONENT lib
        PUBLIC_HEADER DESTINATION "${HELLOWORLD_INSTALL_INCLUDE_DIR}" COMPONENT dev)

# Install compat headers
install(FILES ${HDR_COMPAT}
        DESTINATION
        "${HELLOWORLD_INSTALL_INCLUDE_DIR}"
        COMPONENT dev)

# Install the configs.
install(FILES
        ${PROJECT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/LibhelloworldConfig.cmake
        ${PROJECT_BINARY_DIR}/LibhelloworldConfigVersion.cmake
        DESTINATION
        "${HELLOWORLD_INSTALL_CMAKE_DIR}"
        COMPONENT dev)

# Install exports for the install-tree.
install(EXPORT LibhelloworldTargets
        DESTINATION
        "${HELLOWORLD_INSTALL_CMAKE_DIR}"
        COMPONENT dev)

set(HELLOWORLD_LIBRARIES
        helloworld
        CACHE STRING "helloworld libraries")