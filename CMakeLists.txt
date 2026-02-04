cmake_minimum_required(VERSION 3.21)
cmake_policy(VERSION 3.21)

project(quickjs VERSION "2025.09.13" DESCRIPTION "QuickJS JavaScript Engine" LANGUAGES C)

include(CMakePackageConfigHelpers)

add_library(
    quickjs
    quickjs.c
    dtoa.c
    libregexp.c
    libunicode.c
    cutils.c
)

set_property(TARGET quickjs PROPERTY POSITION_INDEPENDENT_CODE TRUE)

target_compile_definitions(quickjs PRIVATE CONFIG_VERSION="2025-09-13")

if(UNIX)
    target_compile_definitions(quickjs PRIVATE _GNU_SOURCE)
endif()

if(WIN32)
    set_target_properties(quickjs PROPERTIES
        DEBUG_POSTFIX "-debug"
        MINSIZEREL_POSTFIX "-minsizerel"
        RELWITHDEBINFO_POSTFIX "-relwithdebinfo"
    )
    target_compile_options(quickjs PRIVATE /utf-8)
else()
    # Use -fwrapv for defined signed overflow behavior (QuickJS relies on this)
    target_compile_options(quickjs PRIVATE -fwrapv -Wno-sign-compare -Wno-unused-parameter)
endif()

if(UNIX)
    target_link_libraries(quickjs PRIVATE m)
endif()

target_include_directories(quickjs
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:include>
)

install(
    FILES quickjs.h
    DESTINATION include
)

install(
    TARGETS quickjs
    EXPORT quickjs-targets
    ARCHIVE
    LIBRARY
    RUNTIME
    INCLUDES
)

configure_package_config_file("quickjs-config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/quickjs-config.cmake"
    INSTALL_DESTINATION lib/cmake/quickjs
)

install(
    EXPORT quickjs-targets
    FILE quickjs-targets.cmake
    NAMESPACE ProCern::
    DESTINATION "lib/cmake/quickjs"
)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/quickjs-config-version.cmake"
    VERSION ${PACKAGE_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(
    FILES ${CMAKE_CURRENT_BINARY_DIR}/quickjs-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/quickjs-config-version.cmake
    DESTINATION "lib/cmake/quickjs"
)
