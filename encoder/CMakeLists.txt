cmake_minimum_required(VERSION 3.6)

include(GNUInstallDirs)

add_library(encoders encoder.cpp null_encoder.cpp h264_encoder.cpp mjpeg_encoder.cpp)

target_include_directories(encoders 
    PUBLIC 
    ${RPI_3RDPARTY}/syslibs/image/include
)

target_link_directories(encoders 
    PRIVATE
    ${RPI_3RDPARTY}/syslibs/lib 
)
target_link_libraries(encoders PRIVATE jpeg libcamera_apps_core)

install(TARGETS encoders LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})