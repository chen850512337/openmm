#---------------------------------------------------
# OpenMM RPMD Plugin
#
# Creates OpenMMRPMD plugin library.
#
# Windows:
#   OpenMMRPMD.dll
#   OpenMMRPMD.lib
#   OpenMMRPMD_static.lib
# Unix:
#   libOpenMMRPMD.so
#   libOpenMMRPMD_static.a
#----------------------------------------------------

# ----------------------------------------------------------------------------
SET(CREATE_SERIALIZABLE_OPENMM_RPMD FALSE )
# ----------------------------------------------------------------------------

# The source is organized into subdirectories, but we handle them all from
# this CMakeLists file rather than letting CMake visit them as SUBDIRS.
SET(OPENMM_RPMD_PLUGIN_SOURCE_SUBDIRS . openmmapi)

SET(OPENMM_RPMD_LIBRARY_NAME OpenMMRPMD)
SET(SHARED_RPMD_TARGET ${OPENMM_RPMD_LIBRARY_NAME})
SET(STATIC_RPMD_TARGET ${OPENMM_RPMD_LIBRARY_NAME}_static)
IF( CREATE_SERIALIZABLE_OPENMM_RPMD )
    SET(SHARED_RPMD_SERIALIZABLE_TARGET ${OPENMM_RPMD_LIBRARY_NAME}_serializable)
ENDIF( CREATE_SERIALIZABLE_OPENMM_RPMD )

# These are all the places to search for header files which are
# to be part of the API.
SET(API_RPMD_INCLUDE_DIRS) # start empty
FOREACH(subdir ${OPENMM_RPMD_PLUGIN_SOURCE_SUBDIRS})
    # append
    SET(API_RPMD_INCLUDE_DIRS ${API_RPMD_INCLUDE_DIRS}
                                     ${CMAKE_CURRENT_SOURCE_DIR}/${subdir}/include
                                     ${CMAKE_CURRENT_SOURCE_DIR}/${subdir}/include/internal)
ENDFOREACH(subdir)

# We'll need both *relative* path names, starting with their API_INCLUDE_DIRS,
# and absolute pathnames.
SET(API_RPMD_REL_INCLUDE_FILES)   # start these out empty
SET(API_RPMD_ABS_INCLUDE_FILES)

FOREACH(dir ${API_RPMD_INCLUDE_DIRS})
    FILE(GLOB fullpaths ${dir}/*.h)	# returns full pathnames
    SET(API_RPMD_ABS_INCLUDE_FILES ${API_RPMD_ABS_INCLUDE_FILES} ${fullpaths})

    FOREACH(pathname ${fullpaths})
        GET_FILENAME_COMPONENT(filename ${pathname} NAME)
        SET(API_RPMD_REL_INCLUDE_FILES ${API_RPMD_REL_INCLUDE_FILES} ${dir}/${filename})
    ENDFOREACH(pathname)
ENDFOREACH(dir)

# collect up source files
SET(SOURCE_RPMD_FILES) # empty
SET(SOURCE_RPMD_INCLUDE_FILES)

FOREACH(subdir ${OPENMM_RPMD_PLUGIN_SOURCE_SUBDIRS})
    FILE(GLOB src_files  ${CMAKE_CURRENT_SOURCE_DIR}/${subdir}/src/*.cpp ${CMAKE_CURRENT_SOURCE_DIR}/${subdir}/src/*/*.cpp)
    FILE(GLOB incl_files ${CMAKE_CURRENT_SOURCE_DIR}/${subdir}/src/*.h)
    SET(SOURCE_RPMD_FILES         ${SOURCE_RPMD_FILES}         ${src_files})   #append
    SET(SOURCE_RPMD_INCLUDE_FILES ${SOURCE_RPMD_INCLUDE_FILES} ${incl_files})

    ## Make sure we find these locally before looking in OpenMM/include if
    ## OpenMM was previously installed there.
    INCLUDE_DIRECTORIES(BEFORE ${CMAKE_CURRENT_SOURCE_DIR}/${subdir}/include)
ENDFOREACH(subdir)

# ----------------------------------------------------------------------------

INCLUDE_DIRECTORIES(BEFORE ${CMAKE_CURRENT_SOURCE_DIR}/src)

ADD_LIBRARY(${SHARED_RPMD_TARGET} SHARED ${SOURCE_RPMD_FILES} ${SOURCE_RPMD_INCLUDE_FILES} ${API_RPMD_ABS_INCLUDE_FILES})
SET_TARGET_PROPERTIES(${SHARED_RPMD_TARGET} PROPERTIES LINK_FLAGS "${EXTRA_LINK_FLAGS}" COMPILE_FLAGS "${EXTRA_COMPILE_FLAGS} -DOPENMM_RPMD_BUILDING_SHARED_LIBRARY -DLEPTON_BUILDING_SHARED_LIBRARY")

IF(OPENMM_BUILD_STATIC_LIB)
  ADD_LIBRARY(${STATIC_RPMD_TARGET} STATIC ${SOURCE_RPMD_FILES} ${SOURCE_RPMD_INCLUDE_FILES} ${API_RPMD_ABS_INCLUDE_FILES})
  SET_TARGET_PROPERTIES(${STATIC_RPMD_TARGET} PROPERTIES LINK_FLAGS "${EXTRA_LINK_FLAGS}" COMPILE_FLAGS "${EXTRA_COMPILE_FLAGS} -DOPENMM_USE_STATIC_LIBRARIES -DOPENMM_BUILDING_STATIC_LIBRARY -DLEPTON_USE_STATIC_LIBRARIES -DLEPTON_BUILDING_STATIC_LIBRARY")
ENDIF(OPENMM_BUILD_STATIC_LIB)

# ----------------------------------------------------------------------------

# On Linux need to link to libdl
IF(NOT APPLE)
    FIND_LIBRARY(DL_LIBRARY dl)
    IF(DL_LIBRARY)
      TARGET_LINK_LIBRARIES(${SHARED_RPMD_TARGET} ${DL_LIBRARY})
      IF( CREATE_SERIALIZABLE_OPENMM_RPMD )
          TARGET_LINK_LIBRARIES(${SHARED_RPMD_SERIALIZABLE_TARGET} ${DL_LIBRARY})
      ENDIF( CREATE_SERIALIZABLE_OPENMM_RPMD )
      IF(OPENMM_BUILD_STATIC_LIB)
        TARGET_LINK_LIBRARIES(${STATIC_RPMD_TARGET} ${DL_LIBRARY})
      ENDIF(OPENMM_BUILD_STATIC_LIB)
    ENDIF(DL_LIBRARY)
ENDIF()

SET( OpenMMLib OpenMM )

TARGET_LINK_LIBRARIES( ${SHARED_RPMD_TARGET} ${SHARED_TARGET} )
IF( CREATE_SERIALIZABLE_OPENMM_RPMD )
    TARGET_LINK_LIBRARIES( ${SHARED_RPMD_SERIALIZABLE_TARGET} ${SHARED_TARGET} )
ENDIF( CREATE_SERIALIZABLE_OPENMM_RPMD )
IF(OPENMM_BUILD_STATIC_LIB)
    TARGET_LINK_LIBRARIES( ${STATIC_RPMD_TARGET} ${STATIC_TARGET} )
ENDIF(OPENMM_BUILD_STATIC_LIB)

# Which hardware platforms to build

ADD_SUBDIRECTORY(platforms/reference)
ADD_SUBDIRECTORY(platforms/common)

IF(OPENMM_BUILD_OPENCL_LIB)
    SET(OPENMM_BUILD_RPMD_OPENCL_LIB ON CACHE BOOL "Build RPMD implementation for OpenCL")
ELSE(OPENMM_BUILD_OPENCL_LIB)
    SET(OPENMM_BUILD_RPMD_OPENCL_LIB OFF CACHE BOOL "Build RPMD implementation for OpenCL")
ENDIF(OPENMM_BUILD_OPENCL_LIB)
IF(OPENMM_BUILD_RPMD_OPENCL_LIB)
    ADD_SUBDIRECTORY(platforms/opencl)
ENDIF(OPENMM_BUILD_RPMD_OPENCL_LIB)

IF(OPENMM_BUILD_CUDA_LIB)
    SET(OPENMM_BUILD_RPMD_CUDA_LIB ON CACHE BOOL "Build RPMD implementation for CUDA")
ELSE(OPENMM_BUILD_CUDA_LIB)
    SET(OPENMM_BUILD_RPMD_CUDA_LIB OFF CACHE BOOL "Build RPMD implementation for CUDA")
ENDIF(OPENMM_BUILD_CUDA_LIB)
IF(OPENMM_BUILD_RPMD_CUDA_LIB)
    ADD_SUBDIRECTORY(platforms/cuda)
ENDIF(OPENMM_BUILD_RPMD_CUDA_LIB)

IF(OPENMM_BUILD_HIP_LIB)
    SET(OPENMM_BUILD_RPMD_HIP_LIB ON CACHE BOOL "Build RPMD implementation for HIP")
ELSE(OPENMM_BUILD_HIP_LIB)
    SET(OPENMM_BUILD_RPMD_HIP_LIB OFF CACHE BOOL "Build RPMD implementation for HIP")
ENDIF(OPENMM_BUILD_HIP_LIB)
IF(OPENMM_BUILD_RPMD_HIP_LIB)
    ADD_SUBDIRECTORY(platforms/hip)
ENDIF(OPENMM_BUILD_RPMD_HIP_LIB)

INSTALL_TARGETS(/lib RUNTIME_DIRECTORY /lib ${SHARED_RPMD_TARGET})
IF( CREATE_SERIALIZABLE_OPENMM_RPMD )
    INSTALL_TARGETS(/lib/plugins RUNTIME_DIRECTORY /lib/plugins ${SHARED_RPMD_SERIALIZABLE_TARGET})
ENDIF( CREATE_SERIALIZABLE_OPENMM_RPMD )
IF(OPENMM_BUILD_STATIC_LIB)
  INSTALL_TARGETS(/lib RUNTIME_DIRECTORY /lib ${STATIC_RPMD_TARGET})
ENDIF(OPENMM_BUILD_STATIC_LIB)
FILE(GLOB CORE_HEADERS     include/*.h          */include/*.h)
FILE(GLOB TOP_HEADERS      include/openmm/*.h          */include/openmm/*.h)
FILE(GLOB INTERNAL_HEADERS include/openmm/internal/*.h */include/openmm/internal/*.h )
INSTALL_FILES(/include                 FILES ${CORE_HEADERS})
INSTALL_FILES(/include/openmm          FILES ${TOP_HEADERS})
INSTALL_FILES(/include/openmm/internal FILES ${INTERNAL_HEADERS})

#
# Testing
#

ENABLE_TESTING()

IF (EXECUTABLE_OUTPUT_PATH)
  SET (TEST_PATH ${EXECUTABLE_OUTPUT_PATH})
ELSE (EXECUTABLE_OUTPUT_PATH)
  SET (TEST_PATH .)
ENDIF (EXECUTABLE_OUTPUT_PATH)

#IF (OPENMM_BUILD_SERIALIZATION_SUPPORT)
#    ADD_SUBDIRECTORY(serialization)
#ENDIF (OPENMM_BUILD_SERIALIZATION_SUPPORT)
