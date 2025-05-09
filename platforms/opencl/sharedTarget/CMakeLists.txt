#
# Include OpenCL related files.
#
# INCLUDE(${CMAKE_CURRENT_SOURCE_DIR}/../FindOpenCL.cmake)
INCLUDE_DIRECTORIES(${OPENCL_INCLUDE_DIR})

FILE(GLOB OPENCL_KERNELS ${KERNEL_SOURCE_DIR}/kernels/*.cl)
ADD_CUSTOM_COMMAND(OUTPUT ${KERNELS_CPP} ${KERNELS_H}
    COMMAND ${CMAKE_COMMAND}
    ARGS -D KERNEL_SOURCE_DIR=${KERNEL_SOURCE_DIR} -D KERNELS_CPP=${KERNELS_CPP} -D KERNELS_H=${KERNELS_H} -D KERNEL_SOURCE_CLASS=${KERNEL_SOURCE_CLASS} -D KERNEL_FILE_EXTENSION=cl -P ${CMAKE_SOURCE_DIR}/cmake_modules/EncodeKernelFiles.cmake
    DEPENDS ${OPENCL_KERNELS}
)
SET_SOURCE_FILES_PROPERTIES(${KERNELS_CPP} ${KERNELS_H} ${COMMON_KERNELS_CPP} PROPERTIES GENERATED TRUE)
ADD_LIBRARY(${SHARED_TARGET} SHARED ${SOURCE_FILES} ${SOURCE_INCLUDE_FILES} ${API_ABS_INCLUDE_FILES})
ADD_DEPENDENCIES(${SHARED_TARGET} CommonKernels)

TARGET_LINK_LIBRARIES(${SHARED_TARGET} ${OPENMM_LIBRARY_NAME} ${OPENCL_LIBRARIES})
SET_TARGET_PROPERTIES(${SHARED_TARGET} PROPERTIES LINK_FLAGS "${EXTRA_LINK_FLAGS}" COMPILE_FLAGS "${EXTRA_COMPILE_FLAGS} -DOPENMM_COMMON_BUILDING_SHARED_LIBRARY")

INSTALL_TARGETS(/lib/plugins RUNTIME_DIRECTORY /lib/plugins ${SHARED_TARGET})
