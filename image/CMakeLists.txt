cmake_minimum_required(VERSION 3.6)

include(GNUInstallDirs)

find_library(EXIF_LIBRARY exif REQUIRED)
find_library(JPEG_LIBRARY jpeg REQUIRED)
find_library(TIFF_LIBRARY tiff REQUIRED)
find_library(PNG_LIBRARY png REQUIRED)

set(RPI_THIRDPARTY_INCLUDE)

add_library(images bmp.cpp yuv.cpp jpeg.cpp dng.cpp png.cpp)

target_include_directories(images 
     PUBLIC 
     ${RPI_3RDPARTY}/syslibs/image/include
     ${RPI_3RDPARTY}/libexif/include
)

target_link_directories(images
    PUBLIC
    ${RPI_3RDPARTY}/syslibs/lib 
)

target_link_libraries(images PRIVATE jpeg exif png png16 tiff libcamera_apps_core camera)

install(TARGETS images LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
