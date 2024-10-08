cmake_minimum_required (VERSION 3.1.0)
project (XyGrib)
set(VERSION_MAJOR 2)
set(VERSION_MINOR 0)
set(VERSION_PATCH 0)
string(TIMESTAMP TODAY "%Y-%m-%d") 
set(XYGRIB_VERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
set(XYGRIB_VERSION_DATE "${TODAY}")

option(GNU_PACKAGE "Install into GNU standard directories" OFF)
option(GEN_TRANSLATIONS "Generate translations" ON)

set(CMAKE_VERBOSE_MAKEFILE ON)

# Find includes in corresponding build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON)
# Unfotunately cmake <3.9 is not able to handle the coding style of XyGrib and AUTOMOC misses many classes. Once the code is fixed or we can expect newer cmake everywhere, we can enable this and get rid of the QT5_WRAP_CPP() macros
## Instruct CMake to run moc automatically when needed
#set(CMAKE_AUTOMOC ON)
#set(CMAKE_AUTOMOC_RELAXED_MODE TRUE)

# Unfotunately AUTOUIC_SEARCH_PATHS is not available before cmake 3.9 so we can't use this on all the current platforms
# At some future moment, turn this on and simplify src/GUI/CMakeLists.txt
## Create code from a list of Qt designer ui files
#set(CMAKE_AUTOUIC ON)
#set(CMAKE_AUTOUIC_SEARCH_PATHS "${CMAKE_CURRENT_SOURCE_DIR}/src/GUI")

set(CMAKE_CXX_STANDARD 17)
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")
if(UNIX AND NOT APPLE)
add_compile_options(-Wall -fPIC)
endif()
add_compile_definitions(QT_DISABLE_DEPRECATED_UP_TO=0x050F00)
# Add support for address etc sanitizers, part 1/2 (other half after ADD_EXECUTABLE)
if ( CMAKE_VERSION VERSION_GREATER 3.4 ) 
  set(ENABLE_SANITIZER "none" CACHE STRING "Add clang sanitizer to the build")
  set_property(CACHE ENABLE_SANITIZER PROPERTY STRINGS none address memory thread undefined)
  if (NOT "${ENABLE_SANITIZER}" MATCHES "none")
    add_compile_options(-fsanitize=${ENABLE_SANITIZER})
  endif()
endif()

list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_PREFIX}/cmake") 

set(CMAKE_FIND_FRAMEWORK LAST)

# Find the QtWidgets library
find_package(Qt6Widgets CONFIG REQUIRED)
include_directories(${Qt6Widgets_INCLUDE_DIRS})
find_package(Qt6Core CONFIG REQUIRED)
find_package(Qt6Gui CONFIG REQUIRED)
find_package(Qt6Network CONFIG REQUIRED)
include_directories(${Qt6Network_INCLUDE_DIRS})
find_package(Qt6PrintSupport CONFIG REQUIRED)
include_directories(${Qt6PrintSupport_INCLUDE_DIRS})
find_package(Qt6Xml CONFIG REQUIRED)
include_directories(${Qt6Xml_INCLUDE_DIRS})

find_library(LIBNOVA_LIBRARY
     NAMES "libnova.a" "libnova.so"
     PATHS
     $ENV{EXTERNLIBS}/lib
     ~/Library/Frameworks
     /Library/Frameworks
     /usr/local
     /usr
     /sw
     /opt/local
     /opt/csw
     /opt
     PATH_SUFFIXES lib lib64 lib/x86_64-linux-gnu lib/i386-linux-gnu lib/armhf-linux-gnu lib/arm64-linux-gnu
     REQUIRED
 )
find_path(LIBNOVA_INCLUDE_DIR
     NAMES "libnova/libnova.h" "libnova.h"
     PATHS
     $ENV{EXTERNLIBS}/include
     ~/Library/Frameworks/include
     /Library/Frameworks/include
     /usr/local/include
     /usr/include
     /sw/include # Fink
     /opt/local/include # DarwinPorts
     /opt/csw/include # Blastwave
     /opt/include
     REQUIRED
 )
include_directories(${LIBNOVA_INCLUDE_DIR})

#find_package(OpenJPEG REQUIRED)
find_library(OPENJPEG_LIBRARIES
#     NAMES "libopenjp2.a" "libopenjp2.so" "libopenjpeg.so.2" "libopenjp2.dll.a"
     NAMES "libopenjp2.so.7" "libopenjp2.so" "libopenjp2.a" "libopenjpeg.so.2" "libopenjp2.dll.a"
     PATHS
     $ENV{EXTERNLIBS}/lib
     ~/Library/Frameworks
     /Library/Frameworks
     /usr/local
     /usr
     /sw
     /opt/local
     /opt/csw
     /opt
     PATH_SUFFIXES lib lib64
     REQUIRED
 )
find_path(OPENJPEG_INCLUDE_DIR
     NAMES "openjpeg.h"
     PATHS
     $ENV{EXTERNLIBS}/include
     ~/Library/Frameworks/include
     /Library/Frameworks/include
     /usr/local/include
     /usr/include
     /sw/include # Fink
     /opt/local/include # DarwinPorts
     /opt/csw/include # Blastwave
     /opt/include
     PATH_SUFFIXES openjpeg-2.3 openjpeg-2.2 openjpeg-2.1
     REQUIRED
 )
include_directories(${OPENJPEG_INCLUDE_DIR})

if(NOT WIN32)
    include(cmake/FindPROJ.cmake)
    if(NOT PROJ_FOUND)
      message(FATAL_ERROR "PROJ library not found!")
    endif()
    include_directories(${PROJ_INCLUDE_DIRS})
endif()
if(WIN32)
    find_library(PROJ_LIBRARIES
        NAME "libproj.a"
        PATHS
        $ENV{EXTERNLIBS}/lib
         /opt/lib
        REQUIRED
    )
    find_path(PROJ_INCLUDE_DIRS
         NAMES "proj.h"
         PATHS
         $ENV{EXTERNLIBS}/include
         /opt/include
         REQUIRED
     )
endif()
if(WIN32)
    message(STATUS "Search WIN32 BZIP2")
    find_library(BZIP2_LIBRARIES
        NAME "libbz2.a"
        PATHS
        $ENV{EXTERNLIBS}/lib
         /opt/lib
        REQUIRED
    )
    find_path(BZIP2_INCLUDE_DIR
         NAMES "bzlib.h"
         PATHS
         $ENV{EXTERNLIBS}/include
         /opt/include
         REQUIRED
     )
else()
    find_package(BZip2 REQUIRED)
endif()
include_directories(${BZIP2_INCLUDE_DIR})

if(WIN32)
    find_library(ZLIB_LIBRARIES
        NAME "libz.a"
        PATHS
        $ENV{EXTERNLIBS}/lib
        /opt/lib
        REQUIRED
    )
    find_path(ZLIB_INCLUDE_DIR
         NAMES "zlib.h"
         PATHS
         $ENV{EXTERNLIBS}/include
         /opt/include
         REQUIRED
     )
else()
    find_package(ZLIB REQUIRED)
endif()

include_directories(${ZLIB_INCLUDE_DIR})

if(WIN32)
    find_library(PNG_LIBRARIES
        NAME "libpng.a"
        PATHS
        $ENV{EXTERNLIBS}/lib
        /opt/lib
        REQUIRED
    )
    find_path(PNG_INCLUDE_DIR
         NAMES "png.h"
         PATHS
         $ENV{EXTERNLIBS}/include
         /opt/include
         REQUIRED
     )
else()
    find_package(PNG REQUIRED)
endif()

include_directories(${PNG_INCLUDE_DIR})

configure_file(cmake/Version.h.in ${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/include/Version.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/include)

if(GNU_PACKAGE)
    include(GNUInstallDirs)
    set(PREFIX_BIN ${CMAKE_INSTALL_FULL_BINDIR})
    set(PREFIX_PKGDATA ${CMAKE_INSTALL_FULL_DATADIR}/openGribs/${PROJECT_NAME})
else()
    if(WIN32)
        set(PREFIX_BIN ".")
        set(PREFIX_PKGDATA ".")
    else()
        set(PREFIX_BIN ${PROJECT_NAME})
        set(PREFIX_PKGDATA ${PROJECT_NAME})
    endif()
endif()

add_subdirectory(src)
if(GEN_TRANSLATIONS)
    add_subdirectory(data/tr)
endif()

# Installation
# macOS bundle parameters
set_target_properties(${PROJECT_NAME} PROPERTIES MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/cmake/MacOSXBundleInfo.plist.in")
set_target_properties(${PROJECT_NAME} PROPERTIES MACOSX_BUNDLE_INFO_STRING "${PROJECT_NAME}")
set_target_properties(${PROJECT_NAME} PROPERTIES MACOSX_BUNDLE_ICON_FILE "xyGrib.icns")
set_target_properties(${PROJECT_NAME} PROPERTIES MACOSX_BUNDLE_GUI_IDENTIFIER "${PROJECT_NAME}")
set_target_properties(${PROJECT_NAME} PROPERTIES MACOSX_BUNDLE_LONG_VERSION_STRING "${XYGRIB_VERSION}")
set_target_properties(${PROJECT_NAME} PROPERTIES MACOSX_BUNDLE_BUNDLE_NAME "${PROJECT_NAME}")
set_target_properties(${PROJECT_NAME} PROPERTIES MACOSX_BUNDLE_SHORT_VERSION_STRING "${XYGRIB_VERSION}")
set_target_properties(${PROJECT_NAME} PROPERTIES MACOSX_BUNDLE_BUNDLE_VERSION "${XYGRIB_VERSION}")
set_target_properties(${PROJECT_NAME} PROPERTIES MACOSX_BUNDLE_COPYRIGHT "${PROJECT_NAME} Authors")
# Install data
install(DIRECTORY data DESTINATION ${PREFIX_PKGDATA}
        PATTERN "CMakeFiles" EXCLUDE
        PATTERN "CMakeLists.txt" EXCLUDE
        PATTERN "Makefile" EXCLUDE
        PATTERN "cmake_install.cmake" EXCLUDE)
if(GNU_PACKAGE)
    install(FILES debian/xygrib.desktop
            DESTINATION ${CMAKE_INSTALL_FULL_DATADIR}/applications)
    install(FILES debian/xygrib.png
            DESTINATION ${CMAKE_INSTALL_FULL_DATADIR}/pixmaps)
endif()
