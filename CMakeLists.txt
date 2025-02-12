cmake_minimum_required(VERSION 3.22)

# Setup compiler settings
set(CMAKE_C_STANDARD                11)
set(CMAKE_C_STANDARD_REQUIRED       ON)
set(CMAKE_C_EXTENSIONS              ON)
set(CMAKE_CXX_STANDARD              20)
set(CMAKE_CXX_STANDARD_REQUIRED     ON)
set(CMAKE_CXX_EXTENSIONS            ON)
set(PROJ_PATH                       ${CMAKE_CURRENT_SOURCE_DIR})
message("Build type: "              ${CMAKE_BUILD_TYPE})


#
# Core project settings
#
project(WEATHER_STATION)
enable_language(C CXX ASM)

#
# Core MCU flags, CPU, instruction set and FPU setup
# Needs to be set properly for your MCU
#
set(CPU_PARAMETERS
    -mthumb

    # This needs attention to properly set for used MCU
    -mcpu=cortex-m4
    -mfpu=fpv4-sp-d16
    -mfloat-abi=hard
)

# Set linker script
set(linker_script_SRC               ${PROJ_PATH}/STM32F429ZITX_FLASH.ld)
set(EXECUTABLE                      ${CMAKE_PROJECT_NAME})

#
# List of source files to compile
#
set(sources_SRCS
    # Put here your source files, one in each line, relative to CMakeLists.txt file location
    ${PROJ_PATH}/Core/Src/main.c
    ${PROJ_PATH}/Core/Src/it.c
    ${PROJ_PATH}/Core/Src/config.c
    ${PROJ_PATH}/Core/Src/lps25h.c
    ${PROJ_PATH}/Core/Src/rtos_tasks.c
    ${PROJ_PATH}/Core/Src/leds.c
    ${PROJ_PATH}/Core/Src/dateTime.c
    ${PROJ_PATH}/Core/Src/sensors.c
    ${PROJ_PATH}/Core/Startup/system_stm32f4xx.c
    ${PROJ_PATH}/Core/Startup/startup_stm32f429zitx.s
    ${PROJ_PATH}/Core/PrivateDrivers/Src/gpio.c
    ${PROJ_PATH}/Core/PrivateDrivers/Src/spi.c
    ${PROJ_PATH}/Core/PrivateDrivers/Src/adc.c
    ${PROJ_PATH}/Core/PrivateDrivers/Src/uart.c
    ${PROJ_PATH}/Core/PrivateDrivers/Src/rtc.c
    ${PROJ_PATH}/Drivers/FreeRTOS/portable/GCC/ARM_CM4F/port.c
    ${PROJ_PATH}/Drivers/FreeRTOS/portable/MemMang/heap_4.c
    ${PROJ_PATH}/Drivers/FreeRTOS/croutine.c
    ${PROJ_PATH}/Drivers/FreeRTOS/event_groups.c
    ${PROJ_PATH}/Drivers/FreeRTOS/list.c
    ${PROJ_PATH}/Drivers/FreeRTOS/queue.c
    ${PROJ_PATH}/Drivers/FreeRTOS/stream_buffer.c
    ${PROJ_PATH}/Drivers/FreeRTOS/tasks.c
    ${PROJ_PATH}/Drivers/FreeRTOS/timers.c
    
)

#
# Include directories
#
set(include_path_DIRS
    # Put here your include dirs, one in each line, relative to CMakeLists.txt file location
    ${PROJ_PATH}/Core/Inc
    ${PROJ_PATH}/Core/PrivateDrivers/Inc
    ${PROJ_PATH}/Drivers/CMSIS/Device/ST/STM32F4xx/Include
    ${PROJ_PATH}/Drivers/CMSIS/Include
    ${PROJ_PATH}/Drivers/FreeRTOS
    ${PROJ_PATH}/Drivers/FreeRTOS/portable/GCC/ARM_CM4F
    ${PROJ_PATH}/Drivers/FreeRTOS/include
)

#
# Symbols definition
#
set(symbols_SYMB
    # Put here your symbols (preprocessor defines), one in each line
    # Encapsulate them with double quotes for safety purpose
    "DEBUG"
    "STM32F429xx"
)

# Executable files
add_executable(${EXECUTABLE} ${sources_SRCS})

# Include paths
target_include_directories(${EXECUTABLE} PRIVATE ${include_path_DIRS})

# Project symbols
target_compile_definitions(${EXECUTABLE} PRIVATE ${symbols_SYMB})

# Compiler options
target_compile_options(${EXECUTABLE} PRIVATE
    ${CPU_PARAMETERS}
    -Wall
    -Wextra
    -Wpedantic
    -Wno-unused-parameter
    # Full debug configuration
    -Og -g3 -ggdb
)

# Linker options
target_link_options(${EXECUTABLE} PRIVATE
    -T${linker_script_SRC}
    ${CPU_PARAMETERS}
    -Wl,-Map=${CMAKE_PROJECT_NAME}.map
    --specs=nosys.specs
    -u _printf_float 
    -Wl,--start-group
    -lc
    -lm
    -lstdc++
    -lsupc++
    -Wl,--end-group
    -Wl,--print-memory-usage
)

# Execute post-build to print size
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
    COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${EXECUTABLE}>
)

# Convert output to hex and binary
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${EXECUTABLE}> ${EXECUTABLE}.hex
)

# Conver to bin file -> add conditional check?
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${EXECUTABLE}> ${EXECUTABLE}.bin
)