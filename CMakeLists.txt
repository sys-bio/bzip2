cmake_minimum_required(VERSION 3.15)

# Take the version from the sources so it cannot drift from bzlib_private.h.
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/bzlib_private.h" _bz_version_line
     REGEX "^#define[ \t]+BZ_VERSION[ \t]+\"")
string(REGEX MATCH "[0-9]+\\.[0-9]+\\.[0-9]+" BZIP2_VERSION "${_bz_version_line}")
if(NOT BZIP2_VERSION)
    message(FATAL_ERROR "Could not determine the bzip2 version from bzlib_private.h")
endif()

project(bzip2 VERSION ${BZIP2_VERSION} LANGUAGES C)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# Library only. Static or shared follows BUILD_SHARED_LIBS.
add_library(bz2
    blocksort.c
    bzlib.c
    compress.c
    crctable.c
    decompress.c
    huffman.c
    randtable.c
)
add_library(BZip2::BZip2 ALIAS bz2)

if(WIN32 AND BUILD_SHARED_LIBS)
    target_sources(bz2 PRIVATE libbz2.def)
endif()

if(MSVC)
    target_compile_definitions(bz2 PRIVATE _CRT_SECURE_NO_WARNINGS)
endif()

target_include_directories(bz2 PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

set_target_properties(bz2 PROPERTIES
    EXPORT_NAME BZip2
    PUBLIC_HEADER bzlib.h
    POSITION_INDEPENDENT_CODE ON
    DEBUG_POSTFIX d
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
)

install(TARGETS bz2
    EXPORT bzip2-targets
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Package files go directly in <prefix>/lib/cmake so find_package(bzip2 CONFIG)
# is found in the same place as before.
install(EXPORT bzip2-targets
    NAMESPACE BZip2::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake
)

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/bzip2-config.cmake"
    "include(\"\${CMAKE_CURRENT_LIST_DIR}/bzip2-targets.cmake\")\n")

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/bzip2-config-version.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/bzip2-config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/bzip2-config-version.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake
)
