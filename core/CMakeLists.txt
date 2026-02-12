cmake_minimum_required(VERSION 3.6)

include(GNUInstallDirs)

find_package(Boost REQUIRED COMPONENTS program_options)

add_custom_target(VersionCpp ${CMAKE_COMMAND} -DSOURCE_DIR=${CMAKE_SOURCE_DIR} -P ${CMAKE_CURRENT_LIST_DIR}/version.cmake)
set_source_files_properties(version.cpp PROPERTIES GENERATED 1)

add_library(libcamera_app libcamera_app.cpp post_processor.cpp version.cpp options.cpp)
add_dependencies(libcamera_app VersionCpp)

set_target_properties(libcamera_app PROPERTIES PREFIX "" IMPORT_PREFIX "")

install(TARGETS libcamera_app LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

add_library(libcamera_apps_core libcamera_app.cpp post_processor.cpp version.cpp options.cpp)
set_target_properties(libcamera_apps_core PROPERTIES OUTPUT_NAME "camera_apps_core")
target_include_directories(libcamera_apps_core 
PUBLIC 
    ${CMAKE_CURRENT_LIST_DIR}/.. 
    ${RPI_3RDPARTY}/libcamera/include
    ${RPI_3RDPARTY}/boost/include
)

target_link_directories(libcamera_app 
    PRIVATE
    ${RPI_3RDPARTY}/syslibs/lib 
)

target_link_libraries(libcamera_app PRIVATE pthread preview ${LIBCAMERA_LINK_LIBRARIES} ${Boost_LIBRARIES} post_processing_stages libcamera_apps_core)