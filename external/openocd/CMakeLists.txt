PROJECT("openocd")

SET(VERSION_EXTERNAL "0.11.0")

INCLUDE(ExternalProject)

ExternalProject_Add(TARGET_openocd
                    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/openocd
                    URL ${CMAKE_CURRENT_SOURCE_DIR}/openocd-0.11.0.tar.bz2
                    URL_HASH SHA1=8d454145a8c22eb9d837ab3ba08dcef06cd10473
                    DEPENDS TARGET_libusb
                    PATCH_COMMAND "${PYTHON_INTERPRETER}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/apply_patches.py --working-folder ${CMAKE_CURRENT_BINARY_DIR}/openocd/src/TARGET_openocd --patch-folder ${CMAKE_CURRENT_SOURCE_DIR}/patches --strip 1 --copy-folder ${CMAKE_CURRENT_SOURCE_DIR}/copy
                    CONFIGURE_COMMAND ${CMAKE_COMMAND} -E env "CFLAGS=-fPIC" "PKG_CONFIG_PATH=${CMAKE_BINARY_DIR}/install/dev/lib/pkgconfig" ${CMAKE_CURRENT_BINARY_DIR}/openocd/src/TARGET_openocd/configure --prefix=${CMAKE_BINARY_DIR}/install/dev --enable-static=yes --enable-shared=no --with-pic ${CONFIGURE_HOST} --enable-ftdi --enable-stlink --enable-ti-icdi --enable-ulink --enable-usb-blaster-2 --enable-ft232r --enable-vsllink --enable-xds110 --enable-cmsis-dap-v2 --enable-osbdm --enable-opendous --enable-aice --enable-jlink
                    BUILD_COMMAND make
                    INSTALL_COMMAND make install
)

SET(OPENOCD_LIBRARY "${CMAKE_CURRENT_BINARY_DIR}/openocd/src/TARGET_openocd-build/src/.libs/libopenocd.a"
    CACHE INTERNAL "The path to the openocd library.")
SET(JIMTCL_LIBRARY "${CMAKE_CURRENT_BINARY_DIR}/openocd/src/TARGET_openocd-build/jimtcl/libjim.a"
    CACHE INTERNAL "The path to the jim library.")

IF(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
	SET(LIBOPENOCD_LDFLAGS -lws2_32
	    CACHE INTERNAL "The linker flags for the openocd library.")
	SET(OPENOCD_EXE "${CMAKE_BINARY_DIR}/install/dev/bin/openocd.exe"
	    CACHE INTERNAL "The path to the openocd executable.")
ELSE(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
	SET(LIBOPENOCD_LDFLAGS ""
	    CACHE INTERNAL "The linker flags for the openocd library.")
	SET(OPENOCD_EXE "${CMAKE_BINARY_DIR}/install/dev/bin/openocd"
	    CACHE INTERNAL "The path to the openocd executable.")
ENDIF(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")

SET(LIBOPENOCD_INCLUDE_DIRECTORIES ${CMAKE_BINARY_DIR}/external/openocd/openocd/src/TARGET_openocd/src ${CMAKE_BINARY_DIR}/external/openocd/openocd/src/TARGET_openocd/src/helper ${CMAKE_BINARY_DIR}/external/openocd/openocd/src/TARGET_openocd/src/jtag/drivers ${CMAKE_BINARY_DIR}/external/openocd/openocd/src/TARGET_openocd/src/jtag/drivers/libjaylink ${CMAKE_BINARY_DIR}/external/openocd/openocd/src/TARGET_openocd/jimtcl ${CMAKE_BINARY_DIR}/external/openocd/openocd/src/TARGET_openocd-build ${CMAKE_BINARY_DIR}/external/openocd/openocd/src/TARGET_openocd-build/jimtcl ${CMAKE_BINARY_DIR}/external/openocd/openocd/src/TARGET_openocd-build/src
    CACHE INTERNAL "The include paths for the openocd library.")
