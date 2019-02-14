cmake_minimum_required(VERSION 3.3)
project(libusb-gadget VERSION 0.0.1 LANGUAGES C)

#include(CTest) may be used
option(BUILD_TESTING "Build tests" ON)

#Prevent in-source build
if ("${PROJECT_SOURCE_DIR}" STREQUAL "${PROJECT_BINARY_DIR}")
    message(SEND_ERROR "In-source builds are not allowed.")
endif ()

#Configure
include(CheckIncludeFile)
check_include_file("linux/usb/gadgetfs.h" HAVE_GADGETFS_H)
if (NOT HAVE_GADGETFS_H)
    check_include_file("linux/usb_gadgetfs.h" HAVE_GADGETFS_H)
    if (NOT HAVE_GADGETFS_H)
        message(FATAL_ERROR "Can't find linux/usb/gadgetfs.h nor linux/usb_gadgetfs.h")
    endif ()
else ()
    set(HAVE_LINUX_USB_SUBDIR "ON")
endif ()
configure_file("${PROJECT_SOURCE_DIR}/cmake/config.h.in" "${PROJECT_SOURCE_DIR}/src/config.h" @ONLY)

#Add target and properties
add_library(libusb-gadget
        src/usb-gadget.c src/usbstring.c
        )

add_library(LIBUSBgadget::libusb-gadget ALIAS libusb-gadget)

target_include_directories(libusb-gadget
        PUBLIC
        $<INSTALL_INTERFACE:include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
        )

target_compile_options(libusb-gadget PRIVATE $<$<C_COMPILER_ID:GNU>:-Wall>)
#target_compile_options(libusb-gadget PRIVATE $<$<C_COMPILER_ID:GNU>:-Wall -Werror>)
target_compile_options(libusb-gadget PRIVATE $<$<C_COMPILER_ID:Clang>:-Wall -Werror>)

##############################################
# Installation instructions
include(GNUInstallDirs)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/LIBUSBgadget)

install(TARGETS libusb-gadget
        EXPORT libusb-gadget-export
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        )

#This is required so that the exported target has the name LIBUSBgadget
set_target_properties(libusb-gadget PROPERTIES EXPORT_NAME LIBUSBgadget)
install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

#Export the targets to a script
install(EXPORT libusb-gadget-export
        FILE
        LIBUSBgadgetTargets.cmake
        NAMESPACE
        LIBUSBgadget::
        DESTINATION
        ${INSTALL_CONFIGDIR}
        )

#Create a ConfigVersion.cmake file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
        ${CMAKE_CURRENT_BINARY_DIR}/LIBUSBgadgetConfigVersion.cmake
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/cmake/LIBUSBgadgetConfig.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/LIBUSBgadgetConfig.cmake
        INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
        )

#Install the config, configversion and custom find modules
install(FILES
        ${CMAKE_CURRENT_BINARY_DIR}/LIBUSBgadgetConfig.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/LIBUSBgadgetConfigVersion.cmake
        DESTINATION ${INSTALL_CONFIGDIR}
        )

export(EXPORT libusb-gadget-export FILE ${CMAKE_CURRENT_BINARY_DIR}/LIBUSBgadgetTargets.cmake NAMESPACE LIBUSBgadget::)
#Register package in user's package registry
export(PACKAGE LIBUSBgadget)

#pkg-config
set(INSTALL_PKGCONFIG_DIR "${CMAKE_INSTALL_PREFIX}/share/pkgconfig" CACHE PATH "Installation directory for pkgconfig (.pc) files")
#
set(prefix "${CMAKE_INSTALL_PREFIX}")
set(exec_prefix "${CMAKE_INSTALL_BINDIR}")
set(libdir "${CMAKE_INSTALL_LIBDIR}")
set(includedir "${CMAKE_INSTALL_INCLUDEDIR}")
set(VERSION ${PROJECT_VERSION})
#
configure_file("${PROJECT_SOURCE_DIR}/libusb-gadget.pc.in" "${CMAKE_CURRENT_BINARY_DIR}/libusb-gadget.pc" @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/libusb-gadget.pc" DESTINATION "${INSTALL_PKGCONFIG_DIR}")

## tests

if (BUILD_TESTING)
    add_subdirectory(tests)
endif ()
