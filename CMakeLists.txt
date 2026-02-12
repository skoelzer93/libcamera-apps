cmake_minimum_required(VERSION 3.6)

project(libcamera-apps)

if (NOT EXISTS ${CMAKE_BINARY_DIR}/CMakeCache.txt)
    if (NOT CMAKE_BUILD_TYPE)
        set(CMAKE_BUILD_TYPE "Release" CACHE STRING "" FORCE)
        message(STATUS "No previous build - default to Release build")
    endif()
endif()

set (CMAKE_EXPORT_COMPILE_COMMANDS ON)
set (CMAKE_CXX_STANDARD 17)
add_compile_options(-Wall -Wextra -pedantic -Wno-unused-parameter -faligned-new -Werror -Wfatal-errors)
add_definitions(-D_FILE_OFFSET_BITS=64)

if (CMAKE_COMPILER_IS_GNUCXX)
    add_compile_options(-Wno-psabi)
endif()

option(BUILD_SHARED_LIBS "Build using shared libraries" ON)

IF (NOT ENABLE_COMPILE_FLAGS_FOR_TARGET)
    # On a Pi this will give us armhf or arm64.
    execute_process(COMMAND dpkg-architecture -qDEB_HOST_ARCH
        OUTPUT_VARIABLE ENABLE_COMPILE_FLAGS_FOR_TARGET OUTPUT_STRIP_TRAILING_WHITESPACE)
endif()
message(STATUS "Platform: ${ENABLE_COMPILE_FLAGS_FOR_TARGET}")
if ("${ENABLE_COMPILE_FLAGS_FOR_TARGET}" STREQUAL "arm64")
    # 64-bit binaries can be fully optimised.
    add_definitions(-ftree-vectorize)
elseif ("${ENABLE_COMPILE_FLAGS_FOR_TARGET}" STREQUAL "armv8-neon")
    # Only build with 32-bit Pi 3/4 specific optimisations if requested on the command line.
    add_definitions(-mfpu=neon-fp-armv8 -ftree-vectorize)
endif()

# Source package generation setup.
set(CPACK_GENERATOR "TXZ")
set(CPACK_PACKAGE_FILE_NAME "libcamera-apps-build")
set(CPACK_SOURCE_GENERATOR "TXZ")
set(CPACK_INSTALL_SCRIPTS ${CMAKE_SOURCE_DIR}/package.cmake)
set(CPACK_SOURCE_PACKAGE_FILE_NAME "libcamera-apps-src")
set(CPACK_SOURCE_IGNORE_FILES "/\.git*;/build;")
include(CPack)

if(NOT CMAKE_CROSSCOMPILING)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(LIBCAMERA REQUIRED libcamera)
    message(STATUS "libcamera library found:")
    message(STATUS "    version: ${LIBCAMERA_VERSION}")
    message(STATUS "    libraries: ${LIBCAMERA_LINK_LIBRARIES}")
    message(STATUS "    include path: ${LIBCAMERA_INCLUDE_DIRS}")
else()
    message(STATUS "CrossCompile: skipping PkgConfig")
    #pkg_check_modules gets used deeper in, but we are linking dependencies manually, so make sure the function can get called properly when cross-compiling
    if(NOT COMMAND pkg_check_modules)
        function(pkg_check_modules)
            message(STATUS "Cross-Compile: ignoring pkg_check_modules(${ARGV})")
        endfunction()
    endif()
endif()

if(CMAKE_CROSSCOMPILING)
    set(RPI_THIRDPARTY_LIB "C:/rpi-thirdparty/syslibs/lib")
    set(EXIF_LIBRARY "${RPI_THIRDPARTY_LIB}/libexif.so" CACHE FILEPATH "libexif for cross-compile" FORCE)
    message(STATUS "Cross-Compile: EXIF_LIBRARY preset to: ${EXIF_LIBRARY}")
    set(JPEG_LIBRARY "${RPI_THIRDPARTY_LIB}/libjpeg.so" CACHE FILEPATH "libjpeg for cross-compile" FORCE)
    message(STATUS "Cross-Compile: JPEG_LIBRARY preset to: ${JPEG_LIBRARY}")
    set(TIFF_LIBRARY "${RPI_THIRDPARTY_LIB}/libtiff.so" CACHE FILEPATH "libtiff for cross-compile" FORCE)
    message(STATUS "Cross-Compile: TIFF_LIBRARY preset to: ${TIFF_LIBRARY}")
    set(PNG_LIBRARY "${RPI_THIRDPARTY_LIB}/libpng.so" CACHE FILEPATH "libpng for cross-compile" FORCE)
    message(STATUS "Cross-Compile: PNG_LIBRARY preset to: ${PNG_LIBRARY}")
endif()

include_directories(${CMAKE_SOURCE_DIR} ${LIBCAMERA_INCLUDE_DIRS})

add_subdirectory(core)
add_subdirectory(encoder)
add_subdirectory(image)
add_subdirectory(output)
add_subdirectory(preview)
add_subdirectory(post_processing_stages)
add_subdirectory(utils)
add_subdirectory(apps)
