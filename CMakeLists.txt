cmake_minimum_required(VERSION 3.0.4)
project(openivi-html5)

set(ENABLE_ADVANCED_DEBUG OFF CACHE BOOL "Enable Advanced debug")

# Tell CMake to run moc when necessary:
set(CMAKE_AUTOMOC ON)
# Tell CMake to run the UI compiler when necessary
# SET(CMAKE_AUTOUIC ON)
# Only available in cmake 3.0. yocto only has cmake 2.8.12

# As moc files are generated in the binary dir, tell CMake
# to always look for includes there:
set(CMAKE_INCLUDE_CURRENT_DIR ON)

if (ENABLE_ADVANCED_DEBUG)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Werror -D_ADVANCED_DEBUG")
else(ENABLE_ADVANCED_DEBUG)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Werror")
endif(ENABLE_ADVANCED_DEBUG)


set(SocketCAN_DIR ${openivi-html5_SOURCE_DIR})

find_package(Qt5Widgets REQUIRED)
find_package(Qt5WebKitWidgets REQUIRED)
find_package(Qt5DBus REQUIRED)
find_package(X11)

find_program(CTAGS NAMES ctags)

qt5_wrap_ui(UI_MAINWINDOW mainwindow.ui)

set_source_files_properties(softwareloadingmanager.xml
    PROPERTIES INCLUDE softwareloadingmanagerdbus.h)
qt5_add_dbus_interfaces(DBUS_FILES softwareloadingmanager.xml)
qt5_add_resources(RCC_FILES resources/resources.qrc)

# Generate config.h from config.h.in
configure_file(config.h.in config.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR} ${openivi-html5_SOURCE_DIR}/can_reader)

# Add subdirectory for CAN bus reading
add_subdirectory(can_reader)

add_executable(openivi-html5
        main.cc mainwindow.cc webgraphicview.cc softwareloadingmanager.cc
        car.cc installdetail.cc packageid.cc
        virtualkeyboard.cc allowlocation_webpage.cc
        ${DBUS_FILES} ${RCC_FILES} ${UI_MAINWINDOW})

set_property(TARGET openivi-html5 PROPERTY CXX_STANDARD 11)

if (X11_FOUND)
  target_link_libraries(openivi-html5 canreader ${X11_LIBRARIES} Qt5::Widgets Qt5::DBus Qt5::WebKitWidgets)
  include_directories(${X11_INCLUDE_DIR})
else(X11_FOUND)
  target_link_libraries(openivi-html5 canreader Qt5::Widgets Qt5::DBus Qt5::WebKitWidgets)
endif(X11_FOUND)

# qt5_use_modules(openivi-html5 Widgets DBus WebKitWidgets)

message("Build type is ${CMAKE_BUILD_TYPE}")

# Export compile_commands.json for clang-check
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if (CTAGS)
    # Generate tags if the 'ctags' tool is available
    set_source_files_properties(tags PROPERTIES GENERATED true)
    add_custom_target(tags
        COMMAND ${CTAGS} -R --c++-kinds=+p --fields=+iaS --extra=+q .
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
    add_dependencies(openivi-html5 tags)
endif() # CTAGS

# vim: set tabstop=4 shiftwidth=4 expandtab:
