#####################################################
## PROJECT SETTING 
#####################################################
cmake_minimum_required(VERSION 3.16)

if(COMMAND cmake_policy)
    cmake_policy(SET CMP0003 NEW)
endif()

# Version configuration - update this single variable to change all versions
set(PROJECT_VERSION_STRING "0.0.1")

project(MyQtProject VERSION ${PROJECT_VERSION_STRING} LANGUAGES CXX)

# Installation destination - configurable for bin and other outputs
set(INSTALL_BIN_DIR "." CACHE STRING "Installation directory for executables")
set(INSTALL_PLUGIN_DIR "plugins" CACHE STRING "Installation directory for plugins")
set(INSTALL_LIB_DIR "." CACHE STRING "Installation directory for DLLs")

# Update this path to your Qt6 installation
set(CMAKE_PREFIX_PATH "C:\\Qt\\6.9.1\\msvc2022_64\\lib\\cmake\\Qt6")
# set(CMAKE_PREFIX_PATH "/opt/Qt/6.5.0/gcc_64")

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
    ${PROJECT_BINARY_DIR}/${INSTALL_BIN_DIR}
    CACHE PATH
    "Single directory for all libraries"
)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY
    ${PROJECT_BINARY_DIR}/${INSTALL_BIN_DIR}
    CACHE PATH
    "Single directory for all executable and dll"
)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY
    ${PROJECT_BINARY_DIR}/${INSTALL_BIN_DIR}
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

set(CPACK_PACKAGE_INSTALL_REGISTRY_KEY "${PROJECT_NAME} ${PROJECT_VERSION_STRING}")
set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION_STRING}")
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})

set(CMAKE_INSTALL_SYSTEM_RUNTIME_DESTINATION ${INSTALL_BIN_DIR})
include(InstallRequiredSystemLibraries)

install(TARGETS ${PROJECT_NAME}
    DESTINATION ${INSTALL_BIN_DIR}
)

# Install Qt6 DLLs
if(WIN32)
    get_target_property(Qt6_BIN_DIR Qt6::Widgets IMPORTED_LOCATION)
    get_filename_component(Qt6_BIN_DIR ${Qt6_BIN_DIR} DIRECTORY)
    
    # Core Qt6 libraries
    set(QT_DLL_FILES
        ${Qt6_BIN_DIR}/Qt6Core.dll
        ${Qt6_BIN_DIR}/Qt6Gui.dll
        ${Qt6_BIN_DIR}/Qt6Widgets.dll
        ${Qt6_BIN_DIR}/Qt6Sql.dll
        ${Qt6_BIN_DIR}/Qt6Network.dll
        ${Qt6_BIN_DIR}/Qt6Xml.dll
    )
    
    foreach(DLL_FILE ${QT_DLL_FILES})
        if(EXISTS ${DLL_FILE})
            install(FILES ${DLL_FILE} DESTINATION ${INSTALL_LIB_DIR})
        endif()
    endforeach()
    
    # Qt6 platform plugins
    get_filename_component(Qt6_PLUGIN_DIR ${Qt6_BIN_DIR}/../plugins ABSOLUTE)
    if(EXISTS ${Qt6_PLUGIN_DIR}/platforms)
        install(DIRECTORY ${Qt6_PLUGIN_DIR}/platforms DESTINATION ${INSTALL_PLUGIN_DIR})
    endif()
    if(EXISTS ${Qt6_PLUGIN_DIR}/imageformats)
        install(DIRECTORY ${Qt6_PLUGIN_DIR}/imageformats DESTINATION ${INSTALL_PLUGIN_DIR})
    endif()
    if(EXISTS ${Qt6_PLUGIN_DIR}/sqldrivers)
        install(DIRECTORY ${Qt6_PLUGIN_DIR}/sqldrivers DESTINATION ${INSTALL_PLUGIN_DIR})
    endif()
    
    # Qt6 translations (optional, for i18n)
    get_filename_component(Qt6_TRANSLATIONS_DIR ${Qt6_BIN_DIR}/../translations ABSOLUTE)
    if(EXISTS ${Qt6_TRANSLATIONS_DIR})
        install(DIRECTORY ${Qt6_TRANSLATIONS_DIR} DESTINATION . FILES_MATCHING PATTERN "*.qm")
    endif()
endif()

include(CPack)