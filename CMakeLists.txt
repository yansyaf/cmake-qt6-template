#####################################################
## PROJECT SETTING 
#####################################################
cmake_minimum_required(VERSION 3.16)

if(COMMAND cmake_policy)
    cmake_policy(SET CMP0003 NEW)
endif()

project(MyQtApplication)

# Update this path to your Qt6 installation
set(CMAKE_PREFIX_PATH "/opt/Qt/6.5.0/gcc_64")

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

find_package(Qt6 REQUIRED COMPONENTS Widgets Sql Network Xml)

set(CMAKE_BUILD_TYPE Release)

###############################################
## OUTPUT DIRECTORY SETUP
###############################################
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY
    ${PROJECT_BINARY_DIR}/bin
    CACHE PATH
    "Single directory for all libraries"
)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY
    ${PROJECT_BINARY_DIR}/bin
    CACHE PATH
    "Single directory for all executable and dll"
)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY
    ${PROJECT_BINARY_DIR}/lib
    CACHE PATH
    "Single directory for all static libraries"
)

#####################################################
## CPP COMPILER
#####################################################
include_directories(${Qt6Widgets_INCLUDE_DIRS})

add_definitions(-DQT_PLUGIN)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

set(MAIN_UI_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/mainui.cpp
)

set(MAIN_UI_MOC_HDRS 
    ${CMAKE_CURRENT_SOURCE_DIR}/src/mainui.h
)

add_executable(
    ${PROJECT_NAME} 
    ${MAIN_UI_SOURCES}
    ${MAIN_UI_RC_SRCS}
    ${MAIN_UI_MOC_SRCS}
    ${WIN_RESOURCE_FILE}
)

target_link_libraries(${PROJECT_NAME} 
    Qt6::Widgets
    Qt6::Sql
    Qt6::Network
    Qt6::Xml
)

#####################################################
## PACKAGE 
#####################################################
set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_VENDOR ${PROJECT_NAME})
set(CPACK_PACKAGE_EXECUTABLES ${PROJECT_NAME};${PROJECT_NAME})

set(CPACK_PACKAGE_INSTALL_REGISTRY_KEY "${PROJECT_NAME} 1.0.0")
set(CPACK_PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_VERSION_MAJOR "1")
set(CPACK_PACKAGE_VERSION_MINOR "0")
set(CPACK_PACKAGE_VERSION_PATCH "0")

include(InstallRequiredSystemLibraries)

install(TARGETS ${PROJECT_NAME}
    DESTINATION bin
)

include(CPack)