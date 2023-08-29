PROJECT("openocd")

SET(VERSION_EXTERNAL "0.11.0")
SET(VERSION_PACKAGE "1")

SET(PROJECT_VERSION "${VERSION_EXTERNAL}.${VERSION_PACKAGE}")

INCLUDE(ExternalProject)

#----------------------------------------------------------------------------

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


#----------------------------------------------------------------------------
#
# Build a package for the selected platform.
#
SET(CPACK_OUTPUT_FILE_PREFIX ${CMAKE_BINARY_DIR}/../repository/org/openocd/openocd/${PROJECT_VERSION})
SET(CPACK_INCLUDE_TOPLEVEL_DIRECTORY FALSE)
SET(CPACK_PACKAGE_FILE_NAME "openocd-${PROJECT_VERSION}-${JONCHKI_PLATFORM}")

SET(CPACK_GENERATOR "TXZ")
INCLUDE(CPack)

IF(${JONCHKI_PLATFORM_DIST_ID} STREQUAL "windows")
	# Package a windows build.
	CONFIGURE_FILE(installer/install_windows.lua
	               ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
	               @ONLY)

	INSTALL(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/openocd.dll
	        DESTINATION bin)
ELSE(${JONCHKI_PLATFORM_DIST_ID} STREQUAL "windows")
	# Package a linux build.
	CONFIGURE_FILE(installer/install_linux.lua
	               ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
	               @ONLY)

	INSTALL(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/openocd.so
	        DESTINATION bin)
ENDIF(${JONCHKI_PLATFORM_DIST_ID} STREQUAL "windows")

INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
        DESTINATION .)

INSTALL(PROGRAMS ${OPENOCD_EXE}
        DESTINATION bin)

INSTALL(FILES muhkuh_openocd.h DESTINATION include)


CONFIGURE_FILE(installer/openocd.xml
               ${CPACK_OUTPUT_FILE_PREFIX}/openocd-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.xml
               @ONLY)


#-----------------------------------------------------------------------------
#
# Create a custom target named "pack".
# It builds the package and creates the hashes.
#
ADD_CUSTOM_TARGET(pack
                  COMMAND ${CMAKE_MAKE_PROGRAM}
                  COMMAND ${CMAKE_MAKE_PROGRAM} package
                  COMMAND "${PYTHON_INTERPRETER}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/generate_hash.py ${CPACK_OUTPUT_FILE_PREFIX}/openocd-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.xml
                  COMMAND "${PYTHON_INTERPRETER}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/generate_hash.py ${CPACK_OUTPUT_FILE_PREFIX}/openocd-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.tar.xz
                  COMMENT "Pack the artifact and generate the hashes...")


#-----------------------------------------------------------------------------
#
# Filter the pom.xml file.
#
SET(POM_ARTIFACT_ID  "openocd")
SET(POM_GROUP_ID     "org.openocd")
SET(POM_VERSION      "${PROJECT_VERSION}")
SET(POM_PACKAGING    "tar.xz")
CONFIGURE_FILE(installer/pom.xml
               ${CPACK_OUTPUT_FILE_PREFIX}/openocd-${PROJECT_VERSION}.pom
               @ONLY)
