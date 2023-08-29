PROJECT("openocd")

ADD_LIBRARY(LIBRARY_libopenocd SHARED
            muhkuh_openocd.c)
TARGET_INCLUDE_DIRECTORIES(LIBRARY_libopenocd
                           PRIVATE ${LIBUSB_INCLUDE_DIR}
                                   ${LIBOPENOCD_INCLUDE_DIRECTORIES})

TARGET_LINK_LIBRARIES(LIBRARY_libopenocd
                      PRIVATE ${OPENOCD_LIBRARY} ${JIMTCL_LIBRARY} ${LIBUSB_LIBRARIES} "${LIBOPENOCD_LDFLAGS}")
TARGET_COMPILE_DEFINITIONS(LIBRARY_libopenocd
                           PRIVATE HAVE_CONFIG_H
                                   RELSTR=""
                                   GITVERSION="")
# Set the name of the output file to "openocd".
SET_TARGET_PROPERTIES(LIBRARY_libopenocd PROPERTIES
                      PREFIX ""
                      OUTPUT_NAME "openocd")
ADD_DEPENDENCIES(LIBRARY_libopenocd TARGET_openocd)

IF(${JONCHKI_PLATFORM_DIST_ID} STREQUAL "windows")
	INSTALL(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/openocd.dll
	        DESTINATION ${OPENOCD_SO_INSTALL})
ELSE(${JONCHKI_PLATFORM_DIST_ID} STREQUAL "windows")
	INSTALL(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/openocd.so
	        DESTINATION ${OPENOCD_SO_INSTALL})
ENDIF(${JONCHKI_PLATFORM_DIST_ID} STREQUAL "windows")
