# I target a recent cmake, it shouldn't be a problem on a dev machine
cmake_minimum_required(VERSION 3.7)
# Project name
project("dendy2usb")

# Product filename
set(PRODUCT_NAME dendy2usb)

set(LUFA_ROOT_PATH lufa/LUFA)
set(ARCH AVR8)

## AVR Chip Configuration
set(F_CPU 16000000UL)
# CPU, you can find the list here:
# https://gcc.gnu.org/onlinedocs/gcc/AVR-Options.html
set(MCU at90usb162)

# Use AVR GCC toolchain
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_CXX_COMPILER avr-g++)
set(CMAKE_C_COMPILER avr-gcc)
set(CMAKE_ASM_COMPILER avr-gcc)

# Pass defines to compiler
add_definitions(
	-DF_CPU=${F_CPU}
	-DF_USB=${F_CPU}
	-DUSE_LUFA_CONFIG_HEADER
)
# mmcu MUST be passed to bot the compiler and linker, this handle the linker
set(CMAKE_EXE_LINKER_FLAGS -mmcu=${MCU})

add_compile_options(
	-mmcu=${MCU} # MCU
	-Os # optimize
)

include_directories(
	src
	src/Config
	lufa
)

file(GLOB SRC_FILES
	src/*.c
	# common
	${LUFA_ROOT_PATH}/Drivers/USB/Core/${ARCH}/USBController_${ARCH}.c
	${LUFA_ROOT_PATH}/Drivers/USB/Core/${ARCH}/USBInterrupt_${ARCH}.c
	${LUFA_ROOT_PATH}/Drivers/USB/Core/ConfigDescriptors.c
	${LUFA_ROOT_PATH}/Drivers/USB/Core/Events.c
	${LUFA_ROOT_PATH}/Drivers/USB/Core/USBTask.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Common/HIDParser.c
	# host
	${LUFA_ROOT_PATH}/Drivers/USB/Core/${ARCH}/Host_${ARCH}.c
	${LUFA_ROOT_PATH}/Drivers/USB/Core/${ARCH}/Pipe_${ARCH}.c
	${LUFA_ROOT_PATH}/Drivers/USB/Core/${ARCH}/PipeStream_${ARCH}.c
	${LUFA_ROOT_PATH}/Drivers/USB/Core/HostStandardReq.c
	# device
	${LUFA_ROOT_PATH}/Drivers/USB/Core/${ARCH}/Device_${ARCH}.c
	${LUFA_ROOT_PATH}/Drivers/USB/Core/${ARCH}/Endpoint_${ARCH}.c
	${LUFA_ROOT_PATH}/Drivers/USB/Core/${ARCH}/EndpointStream_${ARCH}.c
	${LUFA_ROOT_PATH}/Drivers/USB/Core/DeviceStandardReq.c
	# usbclass device
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Device/AudioClassDevice.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Device/CDCClassDevice.c
	${LUFA_ROOT_PATH}/Drivers/USB/Class/Device/HIDClassDevice.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Device/MassStorageClassDevice.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Device/MIDIClassDevice.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Device/PrinterClassDevice.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Device/RNDISClassDevice.c
	# usbclass host
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Host/AndroidAccessoryClassHost.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Host/AudioClassHost.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Host/CDCClassHost.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Host/HIDClassHost.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Host/MassStorageClassHost.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Host/MIDIClassHost.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Host/PrinterClassHost.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Host/RNDISClassHost.c
	#${LUFA_ROOT_PATH}/Drivers/USB/Class/Host/StillImageClassHost.c
)

SET_SOURCE_FILES_PROPERTIES(src/usbdrv/usbdrvasm.S PROPERTIES LANGUAGE C)

# Create one target
add_executable(${PRODUCT_NAME} ${SRC_FILES})

# Rename the output to .elf as we will create multiple files
set_target_properties(${PRODUCT_NAME} PROPERTIES OUTPUT_NAME ${PRODUCT_NAME}.elf)

# Strip binary for upload
add_custom_target(strip ALL avr-strip ${PRODUCT_NAME}.elf DEPENDS ${PRODUCT_NAME})

# Transform binary into hex file, we ignore the eeprom segments in the step
add_custom_target(hex ALL avr-objcopy -R .eeprom -O ihex ${PRODUCT_NAME}.elf ${PRODUCT_NAME}.hex DEPENDS strip)

# Clean extra files
set_directory_properties(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES "${PRODUCT_NAME}.hex;${PRODUCT_NAME}.eeprom;${PRODUCT_NAME}.lst")
