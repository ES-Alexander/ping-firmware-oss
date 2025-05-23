cmake_minimum_required(VERSION 3.22)

#
# This file is generated only once,
# and is not re-generated if converter is called multiple times.
#
# User is free to modify the file as much as necessary
#

# Setup compiler settings
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Define the build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Debug")
endif()

# Set the project name
set(CMAKE_PROJECT_NAME ping-firmware-oss)

# Include toolchain file
include("cmake/gcc-arm-none-eabi.cmake")

# Enable compile command to ease indexing with e.g. clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)

# Enable CMake support for ASM, C and C++ languages
enable_language(C ASM CXX)

# Core project settings
project(${CMAKE_PROJECT_NAME})
message("Build type: " ${CMAKE_BUILD_TYPE})

# Create an executable object type
add_executable(${CMAKE_PROJECT_NAME})

# Add STM32CubeMX generated sources
add_subdirectory(cmake/stm32cubemx)

# Include ping-cpp message directory
add_subdirectory(${CMAKE_SOURCE_DIR}/Lib/ping-cpp/src/message)

# Link directories setup
target_link_directories(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user defined library search paths
)

# Add sources to executable
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user sources here

    # DSP
    "Core/Src/DSP/echo_finder.c"
    "Core/Src/DSP/steady_state_finder.c"
    "Core/Src/DSP/u8_compress_profile.c"
    "Core/Src/DSP/u8_fast_abs_delta.c"
    "Core/Src/DSP/u8_fast_max.c"
    "Core/Src/DSP/u8_fast_mean.c"
    "Core/Src/DSP/u8_fast_normalize.c"
    "Core/Src/DSP/u8_fast_half_square.c"
    "Core/Src/DSP/u8_fast_std_dev.c"
    "Core/Src/DSP/u8_fast_threshold_cut.c"

    # Sonar
    "Core/Src/Sonar/board.cpp"
    "Core/Src/Sonar/sonar.cpp"
    "Core/Src/Sonar/server.cpp"
)

# Add include paths
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user defined include paths
)

# Add project symbols (macros)
target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user defined symbols
)

# Add linked libraries
target_link_libraries(
    ${CMAKE_PROJECT_NAME}
PRIVATE
    stm32cubemx
    PING_MESSAGES
)

# Create binary and hex files
add_custom_command(
    TARGET ${CMAKE_PROJECT_NAME}
    POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${CMAKE_PROJECT_NAME}.bin
    COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${CMAKE_PROJECT_NAME}.hex
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    COMMENT "Creating binary and hex file"
)

if(NOT DEFINED FLASH_DEVICE)
    set(FLASH_DEVICE "/dev/ttyUSB0")
endif()

# Add custom target for flashing the firmware
add_custom_target(flash
    DEPENDS ${CMAKE_PROJECT_NAME}
    COMMAND python ${CMAKE_CURRENT_SOURCE_DIR}/tools/flash_helper.py --hex ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.hex --device ${FLASH_DEVICE}
    COMMENT "Flashing firmware using flash_helper.py with " ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.hex "and device " ${FLASH_DEVICE}
)
