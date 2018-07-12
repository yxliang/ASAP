cmake_minimum_required(VERSION 2.8.9)
CMAKE_POLICY(SET CMP0020 NEW)
CMAKE_POLICY(SET CMP0043 NEW)
CMAKE_POLICY(SET CMP0053 NEW)
CMAKE_POLICY(SET CMP0071 NEW)
PROJECT(DIAGPathology)

SET(CURRENT_MAJOR_VERSION 1)
SET(CURRENT_MINOR_VERSION 8)
SET(CURRENT_PATCH_VERSION 0)
set(ASAP_VERSION ${CURRENT_MAJOR_VERSION}.${CURRENT_MINOR_VERSION}.${CURRENT_PATCH_VERSION})

include (GenerateExportHeader)

CONFIGURE_FILE (
  "${PROJECT_SOURCE_DIR}/config/ASAPMacros.h.in"
  "${PROJECT_BINARY_DIR}/config/ASAPMacros.h"
  )

SET(CMAKE_MODULE_PATH ${DIAGPathology_SOURCE_DIR}/cmakemodules)
SET_PROPERTY(GLOBAL PROPERTY USE_FOLDERS ON)

IF(NOT WIN32)
SET(CMAKE_INSTALL_RPATH "\$ORIGIN/../lib:$ORIGIN/")
SET(CMAKE_MACOSX_RPATH TRUE)
ENDIF(NOT WIN32)

set(CMAKE_CXX_STANDARD 11)

# Set compile definitions to ignore some warnings
if (WIN32)
    set(DEFAULT_COMPILE_DEFINITIONS ${DEFAULT_COMPILE_DEFINITIONS}
        /D"_SCL_SECURE_NO_WARNINGS"  # Calling any one of the potentially unsafe methods in the Standard C++ Library
        /D"_CRT_SECURE_NO_WARNINGS"  # Calling any one of the potentially unsafe methods in the CRT Library
		/wd4251
    )
	add_definitions(${DEFAULT_COMPILE_DEFINITIONS})
endif()

# 3rd lib
set(3rdlib_DIR "D:/caffe-builder/build/libraries")

set( BOOST_ROOT ${3rdlib_DIR} CACHE PATH "")
set( BOOST_INCLUDEDIR ${BOOST_ROOT}/include/boost-1_61 CACHE PATH "")
set( BOOST_LIBRARYDIR ${BOOST_ROOT}/lib CACHE PATH "")
set( Boost_USE_MULTITHREADED ON CACHE BOOL "")
set( Boost_USE_STATIC_LIBS ON CACHE BOOL "")
set( Boost_USE_STATIC_RUNTIME OFF CACHE BOOL "")
# set( BOOST_ROOT "D:/libraries/boost_1_67_0" CACHE PATH "")
# set( BOOST_INCLUDEDIR ${BOOST_ROOT}/boost CACHE PATH "")
# set( BOOST_LIBRARYDIR ${BOOST_ROOT}/lib64-msvc-14 CACHE PATH "")
# set( Boost_USE_MULTITHREADED ON CACHE BOOL "")
# set( Boost_USE_STATIC_LIBS OFF CACHE BOOL "")

set( ZLIB_INCLUDE_DIR ${3rdlib_DIR}/include)
set( ZLIB_LIBRARY ${3rdlib_DIR}/lib/zlibstatic.lib)
set( TIFF_LIBRARY ${3rdlib_DIR}/x64/vc14/staticlib/libtiff.lib)
set( JPEG_INCLUDE_DIR "c:/Program Files/libjpeg-turbo64/include" )
set( JPEG_LIBRARY "c:/Program Files/libjpeg-turbo64/lib/jpeg-static.lib" )
set( JASPER_INCLUDE_DIR "c:/Program Files/JasPer/include" )
# set( JASPER_LIBRARY "c:/Program Files/JasPer/lib/jasper.lib" )
set( JASPER_LIBRARIES "c:/Program Files/JasPer/lib/jasper.lib" )
set( OPENSLIDE_INCLUDE_DIR "D:/libraries/openslide-win64-20171122/include/openslide" )
set( OPENSLIDE_LIBRARY  "D:/libraries/openslide-win64-20171122/lib/libopenslide.lib")
set( OpenCV_DIR  ${3rdlib_DIR})
set( SWIG_DIR  "D:/libraries/swigwin-3.0.12")
set( SWIG_EXECUTABLE  ${SWIG_DIR}/swig.exe)

FIND_PACKAGE( OpenCV REQUIRED )

# Use boost for cross-platform use of threading, file system and date_time handling
FIND_PACKAGE(Boost QUIET REQUIRED COMPONENTS date_time filesystem program_options regex system thread)

# # Jasper for JPEG2000 codec
# FIND_PACKAGE(Jasper REQUIRED)

# TIFF is the main multi-tiled format used in this package
FIND_PACKAGE(TIFF REQUIRED)

# # JPEG for the codecs
# FIND_PACKAGE(JPEG REQUIRED)

# ZLIB for codec support
FIND_PACKAGE(ZLIB REQUIRED)

# PugiXML required for LIF file format and reading/writing annotations
FIND_PACKAGE(PugiXML REQUIRED)

# ADD_LIBRARY(libtiff STATIC IMPORTED)
# SET_PROPERTY(TARGET libtiff PROPERTY IMPORTED_LOCATION ${TIFF_LIBRARY})

# ADD_LIBRARY(libjasper STATIC IMPORTED)
# SET_PROPERTY(TARGET libjasper PROPERTY IMPORTED_LOCATION ${JASPER_LIBRARY})

ADD_LIBRARY(libjpeg STATIC IMPORTED)
SET_PROPERTY(TARGET libjpeg PROPERTY IMPORTED_LOCATION ${JPEG_LIBRARY})

ADD_LIBRARY(zlib STATIC IMPORTED)
SET_PROPERTY(TARGET zlib PROPERTY IMPORTED_LOCATION ${ZLIB_LIBRARY})

# Required for additional format support
FIND_PACKAGE(OPENSLIDE REQUIRED)

# Build options for the pathology projects
OPTION(BUILD_DIAG_PATHOLOGY_EXECUTABLES "Builds executables" ON)
OPTION(BUILD_DIAG_PATHOLOGY_WORKSTATION "Builds workstation" ON)
OPTION(BUILD_DIAG_PATHOLOGY_IMAGEPROCESSING "Builds image processing routines" ON)
OPTION(BUILD_DIAG_PATHOLOGY_TESTS "Builds tests" OFF)
OPTION(PACKAGE_ON_INSTALL "Copies dependent DLLs and packages install" ON)

IF(BUILD_DIAG_PATHOLOGY_IMAGEPROCESSING)
  # SET(OpenCV_STATIC OFF)
  FIND_PACKAGE(OpenCV QUIET)
ENDIF()

ADD_SUBDIRECTORY(core)
ADD_SUBDIRECTORY(io)
ADD_SUBDIRECTORY(annotation)


IF(BUILD_DIAG_PATHOLOGY_EXECUTABLES)
  ADD_SUBDIRECTORY(executables)
ENDIF()

IF(BUILD_DIAG_PATHOLOGY_MEVISLAB)
  ADD_SUBDIRECTORY(mevislab)
ENDIF()

IF(BUILD_DIAG_PATHOLOGY_IMAGEPROCESSING)
  ADD_SUBDIRECTORY(imgproc)
ENDIF()

IF(BUILD_DIAG_PATHOLOGY_WORKSTATION)
  ADD_SUBDIRECTORY(ASAP)
ENDIF()

ADD_SUBDIRECTORY(buildtools)

set(INSTALL_CMAKE_DIR ${CMAKE_INSTALL_PREFIX}/cmake)
set(INSTALL_INCLUDE_DIR ${CMAKE_INSTALL_PREFIX}/include)
set(INSTALL_BINARY_DIR ${CMAKE_INSTALL_PREFIX}/bin)
set(INSTALL_LIBRARY_DIR ${CMAKE_INSTALL_PREFIX}/lib)

# Create the FooBarConfig.cmake and FooBarConfigVersion files
file(RELATIVE_PATH REL_INCLUDE_DIR "${INSTALL_CMAKE_DIR}" "${INSTALL_INCLUDE_DIR}")
file(RELATIVE_PATH REL_BINARY_DIR "${INSTALL_CMAKE_DIR}" "${INSTALL_BINARY_DIR}")
file(RELATIVE_PATH REL_LIBRARY_DIR "${INSTALL_CMAKE_DIR}" "${INSTALL_LIBRARY_DIR}")

configure_file(ASAPConfig.cmake.in
  "${PROJECT_BINARY_DIR}/ASAPConfig.cmake" @ONLY)
configure_file(ASAPConfigVersion.cmake.in
  "${PROJECT_BINARY_DIR}/ASAPConfigVersion.cmake" @ONLY)
 
# Install the FooBarConfig.cmake and FooBarConfigVersion.cmake
install(FILES
  "${PROJECT_BINARY_DIR}/ASAPConfig.cmake"
  "${PROJECT_BINARY_DIR}/ASAPConfigVersion.cmake"
  DESTINATION cmake)
 
# Install the export set for use with the install-tree
install(EXPORT asap-targets DESTINATION cmake)

IF(PACKAGE_ON_INSTALL)
set(CPACK_PACKAGE_NAME "ASAP")
set(CPACK_PACKAGE_VENDOR "Diagnostic Image Analysis Group")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "ASAP - Automated Slide Analysis Platform")
set(CPACK_PACKAGE_DESCRIPTION "This package provides viewing and analysis tools for whole slide images.")
set(CPACK_PACKAGE_VERSION "${CURRENT_MAJOR_VERSION}.${CURRENT_MINOR_VERSION}")
set(CPACK_PACKAGE_VERSION_MAJOR ${CURRENT_MAJOR_VERSION})
set(CPACK_PACKAGE_VERSION_MINOR ${CURRENT_MINOR_VERSION})
set(CPACK_PACKAGE_VERSION_PATCH ${CURRENT_PATCH_VERSION})
IF(WIN32)
set(CPACK_NSIS_ENABLE_UNINSTALL_BEFORE_INSTALL ON)
SET(CPACK_NSIS_DISPLAY_NAME "ASAP ${CPACK_PACKAGE_VERSION}")
set(CPACK_PACKAGE_EXECUTABLES asap "ASAP")
set(CPACK_NSIS_MUI_ICON ${DIAGPathology_SOURCE_DIR}/ASAP/application.ico)
set(CPACK_NSIS_MUI_UNIICON ${DIAGPathology_SOURCE_DIR}/ASAP/application.ico)
ELSE(WIN32)
SET(CPACK_GENERATOR "DEB")
SET(CPACK_DEBIAN_PACKAGE_MAINTAINER "Geert Litjens") 
set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "ASAP - Automated Slide Analysis Platform")
set(CPACK_PACKAGING_INSTALL_PREFIX /opt/ASAP)
ENDIF(WIN32)
INCLUDE(CPack)
ENDIF(PACKAGE_ON_INSTALL)
