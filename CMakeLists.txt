## zlib project
#
## LICENSE
# This source file is licensed under the zlib license. For the license 
# text, see the provided LICENSE.txt file, section 2.
# 
cmake_minimum_required(VERSION 3.16)

## CMake system setup ##########################################################
## Languages ##
enable_language(C)

## Policies ##
cmake_policy(SET CMP0048 NEW) # project sets VERSION vars
cmake_policy(SET CMP0083 NEW) # PIE
cmake_policy(SET CMP0091 NEW) # MSVC does not set the runtime library in CMAKE_<LANG>_FLAGS
cmake_policy(SET CMP0092 NEW) # MSVC does not set the default /W3 in CMAKE_<LANG>_FLAGS
cmake_policy(SET CMP0009 NEW) # Do not follow symlinks in GLOB_RECURSE

## Modules #####################################################################
include(CMakePackageConfigHelpers)
include(CMakeDependentOption)
include(GNUInstallDirs)
include(CheckPIESupported)
include(CheckTypeSize)
include(CheckFunctionExists)
include(CheckCSourceCompiles)

## System cabaility checks #####################################################
# Position independent code #
check_pie_supported(LANGUAGES C)

# System headers #
check_include_file("sys/types.h" HAVE_SYS_TYPES_H)
check_include_file("stdint.h"    HAVE_STDINT_H)
check_include_file("stddef.h"    HAVE_STDDEF_H)
check_include_file("unistd.h"    HAVE_UNISTD_H)

# System types
check_type_size("off64_t" OFF64_T)

# System functions
check_function_exists("fseeko" HAVE_FSEEKO)

## Project #####################################################################
# Query project version from zlib.h 
# -> original idea from official zlib distribution
file(STRINGS "src/zlib.h" _Z_ZLIB_H_VERSION
     LENGTH_MINIMUM 5 # x.y.z, where x, y, and z are a single digit
     LIMIT_COUNT    1 # we only need one
     REGEX          [[ZLIB_VERSION]] # version macro's name
     )
string(REGEX REPLACE 
       [[.*([0-9]+\.[0-9]+\.[0-9]+).*]] [[\1]] 
       ZLIB_VERSION 
       "${_Z_ZLIB_H_VERSION}"
       )
message(STATUS "Building zlib version ${ZLIB_VERSION}")

project(zlib 
        VERSION ${ZLIB_VERSION}
        DESCRIPTION "The zlib compression library"
        LANGUAGES C
        )

option(Z_ENABLE_ASSEMBLY_X86
       "Enable building the i686 assemlby implementation [Off]" Off
       )
option(Z_ENABLE_ASSEMBLY_X86_64
       "Enable building the x86_64 assemlby implementation [Off]" Off
       )
option(Z_ENABLE_INSTALL 
       "Enable installing zlib runtime for the system [On]" On
       )
cmake_dependent_option(Z_ENABLE_INSTALL_DEVELOPMENT
                       "Enable installing zlib development file for the system [Off]" Off
                       Z_ENABLE_INSTALL Off
                       )
option(Z_ENABLE_EXAMPLES
       "Enable building the zlib examples [Off]" Off
       )
option(Z_ENABLE_SHARED 
       "Enable building the shared zlib library [On]" On
       )
option(Z_ENABLE_STATIC
      "Enable building the static zlib library [On]" On
       )

# Acting on options #
if (MSVC AND (Z_ENABLE_ASSEMBLY_X86 OR Z_ENABLE_ASSEMBLY_X86_64))
    enable_language(ASM_MASM)
endif ()

configure_file("src/zconf.h.cmakein"
               "zconf.h"
               @ONLY
               ) 

# Option integrty checks #
if (NOT (Z_ENABLE_SHARED OR Z_ENABLE_STATIC))
    message(FATAL_ERROR 
            [[
            zlib: No libraries are specified for builing. 
            Specify at least one from Z_ENABLE_SHARED and Z_ENABLE_STATIC
            as true.
            ]]
            )    
endif ()

## Targets #####################################################################
# Default, universal object sources #
add_library(Z_OBJECTS OBJECT
                            src/adler32.c
                            src/compress.c
            src/crc32.h     src/crc32.c
            src/deflate.h   src/deflate.c
            src/gzguts.h
                            src/gzclose.c
                            src/gzlib.c
                            src/gzread.c
                            src/gzwrite.c
            src/inflate.h   src/inflate.c
                            src/infback.c
            src/inftrees.h  src/inftrees.c
            src/inffast.h   src/inffast.c
            src/inffixed.h
            src/trees.h     src/trees.c
            src/zutil.h     src/zutil.c
                            src/uncompr.c
            src/zlib.h
            # special sources
            "${CMAKE_CURRENT_BINARY_DIR}/zconf.h"
            )
target_include_directories(Z_OBJECTS PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
                           $<INSTALL_INTERFACE:include>
                           )
target_compile_definitions(Z_OBJECTS PRIVATE
                           _LARGEFILE64_SOURCE=0$<$<BOOL:${HAVE_OFF64_T}>:1>
                           $<$<NOT:$<BOOL:${HAVE_FSEEKO}>>:NO_FSEEKO>
                           $<$<C_COMPILER_ID:MSVC>:_CRT_SECURE_NO_DEPRECATE>
                           $<$<C_COMPILER_ID:MSVC>:_CRT_NONSTDC_NO_DEPRECATE>
                           )

# x86 assambly sources
add_library(Z_ASM_X86 INTERFACE)
if (Z_ENABLE_ASSEMBLY_X86)
    target_sources(Z_ASM_X86 INTERFACE
                   $<$<C_COMPILER_ID:GNU>:${CMAKE_CURRENT_SOURCE_DIR}/src/asm/match.S>
                   $<$<C_COMPILER_ID:MSVC>:${CMAKE_CURRENT_SOURCE_DIR}/src/asm/match686.asm>
                   $<$<C_COMPILER_ID:MSVC>:${CMAKE_CURRENT_SOURCE_DIR}/src/asm/inffas32.asm>
                   )
    target_compile_definitions(Z_ASM_X86 INTERFACE 
                               ASMV
                               $<$<C_COMPILER_ID:MSVC>:ASMINF>
                               )
    if (CMAKE_C_COMPILER_ID STREQUAL GNU)
        set_source_files_properties("src/asm/match.S" PROPERTIES
                                    LANGUAGE C
                                    COMPILE_FLAGS -DNO_UNDERLINE
                                    )
    endif ()
endif ()

# x86_64 assambly sources
add_library(Z_ASM_X86_64 INTERFACE)
if (Z_ENABLE_ASSEMBLY_X86_64)
    target_sources(Z_ASM_X86_64 INTERFACE
                   $<$<C_COMPILER_ID:GNU>:${CMAKE_CURRENT_SOURCE_DIR}/src/asm/amd64-match.S>
                   $<$<C_COMPILER_ID:MSVC>:${CMAKE_CURRENT_SOURCE_DIR}/src/asm/gvmat64.asm>
                   $<$<C_COMPILER_ID:MSVC>:${CMAKE_CURRENT_SOURCE_DIR}/src/asm/inffasx64.asm>
                   $<$<C_COMPILER_ID:MSVC>:${CMAKE_CURRENT_SOURCE_DIR}/src/asm/inffas8664.c>
                   )
    target_compile_definitions(Z_ASM_X86_64 INTERFACE 
                               ASMV
                               $<$<C_COMPILER_ID:MSVC>:ASMINF>
                               )
    if (CMAKE_C_COMPILER_ID STREQUAL GNU)
        set_source_files_properties("src/asm/amd64-match.S" PROPERTIES
                                    LANGUAGE C
                                    COMPILE_FLAGS -DNO_UNDERLINE
                                    )
    endif ()
endif ()

target_link_libraries(Z_OBJECTS PRIVATE 
                      Z_ASM_X86 
                      Z_ASM_X86_64
                      )
if (Z_ENABLE_INSTALL_DEVELOPMENT)
    install(TARGETS Z_OBJECTS Z_ASM_X86 Z_ASM_X86_64
            EXPORT zlibTargets
            )
endif ()

## z64 utility interface ##
add_library(z64 INTERFACE)
add_library(zlib::z64 ALIAS z64)

target_compile_definitions(z64 INTERFACE _FILE_OFFSET_BITS=64)

if (Z_ENABLE_INSTALL_DEVELOPMENT)
    install(TARGETS z64
            EXPORT zlibTargets)
endif ()

## static library ##
if (Z_ENABLE_STATIC)
    add_library(zlibstatic STATIC)
    add_library(zlib::zlibstatic ALIAS zlibstatic)
    target_link_libraries(zlibstatic Z_OBJECTS)
    if (Z_ENABLE_INSTALL_DEVELOPMENT)
        install(TARGETS zlibstatic 
                EXPORT zlibTargets
                LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
                )
    endif ()
endif ()

## shared library ##
if (Z_ENABLE_SHARED)
    add_library(zlib SHARED
                $<$<PLATFORM_ID:Windows>:src/zlib.def>
                $<$<PLATFORM_ID:Windows>:src/res/zlib1.rc>
                )
    add_library(zlib::zlib ALIAS zlib)
    target_link_libraries(zlib Z_OBJECTS)
    if (Z_ENABLE_INSTALL)
        install(TARGETS zlib
                RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
                )
    endif ()
    if (Z_ENABLE_INSTALL_DEVELOPMENT)
        install(TARGETS zlib
                EXPORT zlibTargets
                LIBRARY DESTINATION "${CMAKE_INSTALL_BINDIR}"
                )
    endif ()
    
    set_source_files_properties("src/res/zlib1.rc"
                                COMPILE_FLAGS $<$<C_COMPILER_ID:GNU>:-DGCC_WINDRES>
                                )

    set(_Z_UNICES_LINK_FLAGS "-Wl,--version-script,\"${CMAKE_CURRENT_SOURCE_DIR}/src/zlib.map\"")
    target_link_options(zlib PUBLIC
                        $<$<AND:$<BOOL:${UNIX}>,$<NOT:$<BOOL:${APPLE}>>>:LINK_FLAGS ${_Z_UNICES_LINK_FLAGS}>
                        )
    set_target_properties(zlib PROPERTIES
                          DEFINE_SYMBOL ZLIB_DLL
                          SOVERSION     1
                          OUTPUT_NAME   "z$<$<BOOL:${WIN32}>:lib1>"
                          # As per the original distribution's CMakeLists.txt file. I have no idea how Cygwin works
                          $<$<NOT:$<BOOL:${CYGWIN}>>:VERSION> $<$<NOT:$<BOOL:${CYGWIN}>>:${ZLIB_VERSION}>
                          )
endif ()

## Eeampels ####################################################################
if (Z_ENABLE_EXAMPLES)
    include(CTest)

    add_subdirectory(examples)
endif ()

## Installation ################################################################
if (Z_ENABLE_INSTALL_DEVELOPMENT)
    install(EXPORT zlibTargets
            FILE zlibTargets.cmake
            NAMESPACE zlib::
            DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
            )

    configure_package_config_file("cmake/zlibConfig.in.cmake"
                                  "zlibConfig.cmake"
                                  INSTALL_DESTINATION
                                  "${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}/cmake"
                                  PATH_VARS
                                  CMAKE_INSTALL_INCLUDEDIR
                                  CMAKE_INSTALL_LIBDIR
                                  )

    write_basic_package_version_file(zlibConfigVersion.cmake
                                     VERSION ${ZLIB_VERSION}
                                     COMPATIBILITY SameMajorVersion
                                     )

    install(FILES 
            "${CMAKE_CURRENT_BINARY_DIR}/zlibConfig.cmake"
            "${CMAKE_CURRENT_BINARY_DIR}/zlibConfigVersion.cmake"
            DESTINATION
            "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
            )
endif ()

