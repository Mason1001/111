cmake_minimum_required(VERSION 3.20)
project(AccessibilityGamingAssistant VERSION 1.0.0 LANGUAGES CXX)

# =============================================================================
# C++ Standard
# =============================================================================
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# =============================================================================
# Build Configuration
# =============================================================================
# Set default build type if not specified (for single-config generators)
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type" FORCE)
    message(STATUS "Setting build type to 'Release' as none was specified.")
endif()

# Export compile commands for IDE support
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# =============================================================================
# Qt6 Configuration
# =============================================================================
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

# Find Qt6 - REQUIRED
find_package(Qt6 REQUIRED COMPONENTS Core Gui Widgets Network)
if(Qt6_FOUND)
    message(STATUS "Found Qt6 version: ${Qt6_VERSION}")
    message(STATUS "Qt6 Core library: ${Qt6Core_LIBRARIES}")
else()
    message(FATAL_ERROR "Qt6 not found! Please install Qt6 or set Qt6_DIR")
endif()

# =============================================================================
# OpenCV Configuration
# =============================================================================
find_package(OpenCV REQUIRED COMPONENTS core imgproc highgui videoio imgcodecs)
if(OpenCV_FOUND)
    message(STATUS "Found OpenCV version: ${OpenCV_VERSION}")
    message(STATUS "OpenCV libraries: ${OpenCV_LIBS}")
    message(STATUS "OpenCV include dirs: ${OpenCV_INCLUDE_DIRS}")
else()
    message(FATAL_ERROR "OpenCV not found! Please install via vcpkg: vcpkg install opencv4:x64-windows")
endif()

# =============================================================================
# Source Files
# =============================================================================
set(CORE_SOURCES
    src/core/ScreenCapture.cpp
    src/core/ColorDetection.cpp
    src/core/MouseController.cpp
    src/core/Tracker.cpp
    src/core/Overlay.cpp
)

set(UI_SOURCES
    src/ui/MainWindow.cpp
    src/ui/ColorPicker.cpp
    src/ui/AdvancedColorPicker.cpp
)

set(UTILS_SOURCES
    src/utils/ConfigManager.cpp
    src/utils/TranslationManager.cpp
    src/utils/StatsTracker.cpp
)

set(CORE_HEADERS
    include/core/ScreenCapture.h
    include/core/ColorDetection.h
    include/core/MouseController.h
    include/core/Tracker.h
    include/core/Overlay.h
)

set(UI_HEADERS
    include/ui/MainWindow.h
    include/ui/ColorPicker.h
    include/ui/AdvancedColorPicker.h
)

set(UTILS_HEADERS
    include/utils/ConfigManager.h
    include/utils/TranslationManager.h
    include/utils/StatsTracker.h
)

set(ALL_SOURCES
    src/main.cpp
    ${CORE_SOURCES}
    ${UI_SOURCES}
    ${UTILS_SOURCES}
)

set(ALL_HEADERS
    ${CORE_HEADERS}
    ${UI_HEADERS}
    ${UTILS_HEADERS}
)

# =============================================================================
# Create Executable
# =============================================================================
add_executable(${PROJECT_NAME} WIN32
    ${ALL_SOURCES}
    ${ALL_HEADERS}
    resources/resources.qrc
)

# Include directories
target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${OpenCV_INCLUDE_DIRS}
)

# =============================================================================
# Link Libraries
# =============================================================================
target_link_libraries(${PROJECT_NAME} PRIVATE
    Qt6::Core
    Qt6::Gui
    Qt6::Widgets
    Qt6::Network
    ${OpenCV_LIBS}
)

# =============================================================================
# Windows Specific Settings
# =============================================================================
if(WIN32)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        user32
        gdi32
        dwmapi
    )
    
    # Set Windows subsystem
    set_target_properties(${PROJECT_NAME} PROPERTIES
        WIN32_EXECUTABLE TRUE
    )
    
    # Use MSVC runtime library (dynamic)
    if(MSVC)
        set_property(TARGET ${PROJECT_NAME} PROPERTY
            MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL"
        )
    endif()
endif()

# =============================================================================
# Compiler Warnings
# =============================================================================
if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /W3 /WX-)
else()
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -Wpedantic)
endif()

# =============================================================================
# Install Rules
# =============================================================================
install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION bin
)

# =============================================================================
# Summary
# =============================================================================
message(STATUS "")
message(STATUS "=== Build Configuration Summary ===")
message(STATUS "Project: ${PROJECT_NAME} v${PROJECT_VERSION}")
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "C++ Standard: ${CMAKE_CXX_STANDARD}")
message(STATUS "Qt6 Version: ${Qt6_VERSION}")
message(STATUS "OpenCV Version: ${OpenCV_VERSION}")
message(STATUS "===================================")
