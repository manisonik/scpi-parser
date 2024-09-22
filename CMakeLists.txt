cmake_minimum_required(VERSION 3.22)

set(scpi_VERSION 2.4.0)

# Core project settings
project(scpi
    VERSION ${scpi_VERSION}
    DESCRIPTION "Open Source SCPI device library"
    HOMEPAGE_URL "https://github.com/manisonik/scpi-parser"
    LANGUAGES C
)

# Setup compiler settings
set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)

# if you haven't included it already
# definitions of CMAKE_INSTALL_LIBDIR, CMAKE_INSTALL_INCLUDEDIR and others
include(GNUInstallDirs)

# check if in the top-level project
if (NOT CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    message(STATUS "This project has a top-level one called [${CMAKE_PROJECT_NAME}]")
else()
    message(STATUS "This project is a top-level one")
endif()

# create the shared library (DLL)
add_library(${PROJECT_NAME}
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/src/error.c
	${CMAKE_CURRENT_SOURCE_DIR}/libscpi/src/expression.c
	${CMAKE_CURRENT_SOURCE_DIR}/libscpi/src/fifo.c
	${CMAKE_CURRENT_SOURCE_DIR}/libscpi/src/ieee488.c
	${CMAKE_CURRENT_SOURCE_DIR}/libscpi/src/lexer.c
	${CMAKE_CURRENT_SOURCE_DIR}/libscpi/src/minimal.c
	${CMAKE_CURRENT_SOURCE_DIR}/libscpi/src/parser.c
	${CMAKE_CURRENT_SOURCE_DIR}/libscpi/src/units.c
	${CMAKE_CURRENT_SOURCE_DIR}/libscpi/src/utils.c
)

target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/src
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc
)

# note that it is not CMAKE_INSTALL_PREFIX we are checking here
if(DEFINED CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    message(
        STATUS
        "CMAKE_INSTALL_PREFIX is not set\n"
        "Default value: ${CMAKE_INSTALL_PREFIX}\n"
        "Will set it to ${CMAKE_SOURCE_DIR}/install"
    )
    set(CMAKE_INSTALL_PREFIX
        "${CMAKE_SOURCE_DIR}/install"
        CACHE PATH "Where the library will be installed to" FORCE
    )
else()
    message(
        STATUS
        "CMAKE_INSTALL_PREFIX was already set\n"
        "Current value: ${CMAKE_INSTALL_PREFIX}"
    )
endif()

# without it public headers won't get installed
set(public_headers
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc/scpi/cc.h
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc/scpi/config.h
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc/scpi/constants.h
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc/scpi/error.h
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc/scpi/expression.h
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc/scpi/ieee488.h
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc/scpi/minimal.h
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc/scpi/parser.h
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc/scpi/scpi.h
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc/scpi/types.h
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc/scpi/units.h
    ${CMAKE_CURRENT_SOURCE_DIR}/libscpi/inc/scpi/utils.h
)

# note that ${public_headers} has to be in quotes
set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER "${public_headers}")

# set the debug post-fix
set_target_properties(${PROJECT_NAME} PROPERTIES DEBUG_POSTFIX "d")

# install the target and create export-set
install(TARGETS ${PROJECT_NAME}
    EXPORT "${PROJECT_NAME}Targets"
    # these get default values from GNUInstallDirs, no need to set them
    #RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} # bin
    #LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} # lib
    #ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} # lib
    # except for public headers, as we want them to be inside a library folder
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME} # include/SomeLibrary
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} # include
)

# generate and install export file
install(EXPORT "${PROJECT_NAME}Targets"
    FILE "${PROJECT_NAME}Targets.cmake"
    NAMESPACE manisonik::
    DESTINATION cmake
)

include(CMakePackageConfigHelpers)

# generate the version file for the config file
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    VERSION "${version}"
    COMPATIBILITY AnyNewerVersion
)
# create config file
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    INSTALL_DESTINATION cmake
)
# install config files
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    DESTINATION cmake
)

