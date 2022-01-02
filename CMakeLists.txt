# CMakeList.txt : CMake project for template-native-gui-app, include source and define
# project specific logic here.
#
cmake_minimum_required(VERSION 3.16)

# This is a C/C++ project
project(myapp VERSION 1.0 LANGUAGES C CXX)

# Use `pkg-config` to link needed libraries.
find_package(PkgConfig REQUIRED)

# Use SDL2 for window creation and event handling.
pkg_check_modules(SDL2 REQUIRED sdl2)

# Add source to this project's executable.
add_executable(myapp "src/myapp.c")

# Link SDL2
target_include_directories(myapp SYSTEM PRIVATE ${SDL2_INCLUDE_DIRS})
target_link_libraries(myapp PRIVATE ${SDL2_LIBRARIES})

add_subdirectory(third_party/lvgl EXCLUDE_FROM_ALL)

target_include_directories(lvgl SYSTEM PRIVATE ${SDL2_INCLUDE_DIRS})
target_link_libraries(lvgl PRIVATE ${SDL2_LIBRARIES})
target_compile_definitions(lvgl PUBLIC LV_CONF_PATH=../../../src/lv_conf.h)

target_link_libraries(myapp PRIVATE lvgl)

if ("$ENV{CROSS_COMPILE}" STREQUAL "arm-webos-linux-gnueabi-")
    install(TARGETS myapp RUNTIME DESTINATION .)
    install(FILES appinfo.json icon.png DESTINATION .)

    set(CPACK_GENERATOR "External")
    set(CPACK_EXTERNAL_PACKAGE_SCRIPT "${CMAKE_SOURCE_DIR}/AresPackage.cmake")
    set(CPACK_EXTERNAL_ENABLE_STAGING TRUE)
    set(CPACK_MONOLITHIC_INSTALL TRUE)

    include(CPack)

    add_custom_target(myapp-package COMMAND cpack)
endif()