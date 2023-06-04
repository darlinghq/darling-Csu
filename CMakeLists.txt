project(csu)

cmake_minimum_required(VERSION 3.10)

enable_language(C ASM)

if(COMMAND cmake_policy)
	cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)


function(build_csu_object csu_build_name)
	cmake_parse_arguments(BUILD_CSU_OBJECT "" "OBJ_NAME;OS_MIN_VERSION;TARGET;ARCH" "SOURCES;DEFINITIONS;ADDITIONAL_FLAGS" ${ARGN})
	message("csu_build_name: ${csu_build_name}")
	message("BUILD_CSU_OBJECT_OBJ_NAME: ${BUILD_CSU_OBJECT_OBJ_NAME}")
	message("BUILD_CSU_OBJECT_OS_MIN_VERSION: ${BUILD_CSU_OBJECT_OS_MIN_VERSION}")
	message("BUILD_CSU_OBJECT_TARGET: ${BUILD_CSU_OBJECT_TARGET}")
	message("BUILD_CSU_OBJECT_ARCH: ${BUILD_CSU_OBJECT_ARCH}")
	message("BUILD_CSU_OBJECT_SOURCES: ${BUILD_CSU_OBJECT_SOURCES}")
	message("BUILD_CSU_OBJECT_DEFINITIONS: ${BUILD_CSU_OBJECT_DEFINITIONS}")
	message("BUILD_CSU_OBJECT_ADDITIONAL_FLAGS: ${BUILD_CSU_OBJECT_ADDITIONAL_FLAGS}")

	set(DARLING_BUILD_FLAGS
		-target ${BUILD_CSU_OBJECT_TARGET}
		-arch ${BUILD_CSU_OBJECT_ARCH}
		-isysroot "${DARLING_TOP_DIRECTORY}/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk"
		-fuse-ld="${CMAKE_BINARY_DIR}/src/external/cctools-port/cctools/ld64/src/${APPLE_TARGET_TRIPLET_PRIMARY}-ld"
	)

	foreach(SOURCE_FILE ${BUILD_CSU_OBJECT_SOURCES})
		list(APPEND CSU_ABSOLUTE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/${SOURCE_FILE}")
	endforeach()
	message("CSU_ABSOLUTE_SOURCES: ${CSU_ABSOLUTE_SOURCES}")

	foreach(DEFINITION ${BUILD_CSU_OBJECT_DEFINITIONS})
		list(APPEND CSU_DEFINITION_FLAGS "-D${DEFINITION}")
	endforeach()
	message("CSU_DEFINITION_FLAGS: ${CSU_DEFINITION_FLAGS}")
	
	set(CSU_OBJ_PARENT_PATH "${CMAKE_CURRENT_BINARY_DIR}/${BUILD_CSU_OBJECT_ARCH}")
	file(MAKE_DIRECTORY "${CSU_OBJ_PARENT_PATH}")
	set(CSU_OBJ_PATH "${CSU_OBJ_PARENT_PATH}/${BUILD_CSU_OBJECT_OBJ_NAME}")
	message("CSU_OBJ_PATH: ${CSU_OBJ_PATH}")

	if(NOT "${BUILD_CSU_OBJECT_OS_MIN_VERSION}" STREQUAL "")
		set(CSU_OS_MIN_VERSION "-mmacosx-version-min=${BUILD_CSU_OBJECT_OS_MIN_VERSION}")
	endif(NOT "${BUILD_CSU_OBJECT_OS_MIN_VERSION}" STREQUAL "")
	message("CSU_OS_MIN_VERSION: ${CSU_OS_MIN_VERSION}")

	add_custom_target(${csu_build_name}_${BUILD_CSU_OBJECT_ARCH} ALL
		COMMAND
			${CMAKE_C_COMPILER} -r ${DARLING_BUILD_FLAGS} -Os ${CSU_OS_MIN_VERSION} ${BUILD_CSU_OBJECT_ADDITIONAL_FLAGS} -nostdlib -keep_private_externs ${CSU_ABSOLUTE_SOURCES} -o "${CSU_OBJ_PATH}" ${CSU_DEFINITION_FLAGS}
		SOURCES
			${BUILD_CSU_OBJECT_SOURCES}
		BYPRODUCTS
			${CSU_OBJ_PATH}
	)
	add_dependencies(${csu_build_name}_${BUILD_CSU_OBJECT_ARCH} ${APPLE_TARGET_TRIPLET_PRIMARY}-ld)
endfunction(build_csu_object)


function(merge_csu_objects csu_build_name)
	cmake_parse_arguments(MERGE_CSU_OBJECTS "" "OBJ_NAME" "ARCHS" ${ARGN})
	message("csu_build_name: ${csu_build_name}")
	message("MERGE_CSU_OBJECTS_OBJ_NAME: ${MERGE_CSU_OBJECTS_OBJ_NAME}")
	message("MERGE_CSU_OBJECTS_ARCHS: ${MERGE_CSU_OBJECTS_ARCHS}")

	foreach(ARCH ${MERGE_CSU_OBJECTS_ARCHS})
		set(CSU_ARCH_OBJ_PATH "${CMAKE_CURRENT_BINARY_DIR}/${ARCH}/${MERGE_CSU_OBJECTS_OBJ_NAME}")

		list(APPEND CSU_LIPO_OBJ_SOURCES "${CSU_ARCH_OBJ_PATH}")
		list(APPEND CSU_LIPO_DEPENDENCIES "${csu_build_name}_${ARCH}")
		list(APPEND CSU_LIPO_ARCH_ARGUMENTS -arch ${ARCH} "${CSU_ARCH_OBJ_PATH}")
	endforeach()
	message("CSU_LIPO_OBJ_SOURCES: ${CSU_LIPO_OBJ_SOURCES}")
	message("CSU_LIPO_DEPENDENCIES: ${CSU_LIPO_DEPENDENCIES}")
	message("CSU_LIPO_ARCH_ARGUMENTS: ${CSU_LIPO_ARCH_ARGUMENTS}")

	set(CSU_OBJ_PATH "${CMAKE_CURRENT_BINARY_DIR}/${MERGE_CSU_OBJECTS_OBJ_NAME}")
	message("CSU_OBJ_PATH: ${CSU_OBJ_PATH}")

	add_custom_target(${csu_build_name} ALL
		COMMAND
			"${CMAKE_BINARY_DIR}/src/external/cctools-port/cctools/misc/lipo" ${CSU_LIPO_ARCH_ARGUMENTS} -create -output ${CSU_OBJ_PATH}
		SOURCES
			${CSU_LIPO_OBJ_SOURCES}
		BYPRODUCTS
			${CSU_OBJ_PATH}
	)
	add_dependencies(${csu_build_name} lipo ${CSU_LIPO_DEPENDENCIES})
endfunction(merge_csu_objects)


function(copy_csu_object csu_build_name)
	cmake_parse_arguments(COPY_CSU_OBJECT "" "OBJ_NAME;ARCH" "" ${ARGN})
	message("COPY_CSU_OBJECT_OBJ_NAME: ${COPY_CSU_OBJECT_OBJ_NAME}")
	message("COPY_CSU_OBJECT_ARCH: ${COPY_CSU_OBJECT_ARCH}")

	set(CSU_ARCH_OBJ_PATH "${CMAKE_CURRENT_BINARY_DIR}/${COPY_CSU_OBJECT_ARCH}/${COPY_CSU_OBJECT_OBJ_NAME}")
	set(CSU_OBJ_PATH "${CMAKE_CURRENT_BINARY_DIR}/${COPY_CSU_OBJECT_OBJ_NAME}")
	message("CSU_ARCH_OBJ_PATH: ${CSU_ARCH_OBJ_PATH}")
	message("CSU_OBJ_PATH: ${CSU_OBJ_PATH}")

	add_custom_target(${csu_build_name} ALL
		COMMAND
			cp ${CSU_ARCH_OBJ_PATH} ${CSU_OBJ_PATH}
		SOURCES
			${CSU_ARCH_OBJ_PATH}
		BYPRODUCTS
			${CSU_OBJ_PATH}
	)
	add_dependencies(${csu_build_name} ${csu_build_name}_${COPY_CSU_OBJECT_ARCH})
endfunction(copy_csu_object)


function(csu_object csu_object_name)
	cmake_parse_arguments(CSU_OBJECT "" "OBJ_NAME;OS_MIN_VERSION" "SOURCES;DEFINITIONS;ADDITIONAL_FLAGS" ${ARGN})
	message("csu_object_name: ${csu_object_name}")
	message("CSU_OBJECT_OBJ_NAME: ${CSU_OBJECT_OBJ_NAME}")
	message("CSU_OBJECT_OS_MIN_VERSION: ${CSU_OBJECT_OS_MIN_VERSION}")
	message("CSU_OBJECT_SOURCES: ${CSU_OBJECT_SOURCES}")
	message("CSU_OBJECT_DEFINITIONS: ${CSU_OBJECT_DEFINITIONS}")
	message("CSU_OBJECT_ADDITIONAL_FLAGS: ${CSU_OBJECT_ADDITIONAL_FLAGS}")

	if (BUILD_TARGET_32BIT)
		build_csu_object(${csu_object_name}
			OBJ_NAME ${CSU_OBJECT_OBJ_NAME}
			OS_MIN_VERSION ${CSU_OBJECT_OS_MIN_VERSION}
			TARGET ${APPLE_TARGET_TRIPLET_32BIT}
			ARCH ${APPLE_ARCH_32BIT}

			SOURCES ${CSU_OBJECT_SOURCES}
			DEFINITIONS ${CSU_OBJECT_DEFINITIONS}
			ADDITIONAL_FLAGS ${CSU_OBJECT_ADDITIONAL_FLAGS}
		)
	endif (BUILD_TARGET_32BIT)

	if (BUILD_TARGET_64BIT)
		build_csu_object(${csu_object_name}
			OBJ_NAME ${CSU_OBJECT_OBJ_NAME}
			OS_MIN_VERSION ${CSU_OBJECT_OS_MIN_VERSION}
			TARGET ${APPLE_TARGET_TRIPLET_64BIT}
			ARCH ${APPLE_ARCH_64BIT}

			SOURCES ${CSU_OBJECT_SOURCES}
			DEFINITIONS ${CSU_OBJECT_DEFINITIONS}
			ADDITIONAL_FLAGS ${CSU_OBJECT_ADDITIONAL_FLAGS}
		)
	endif(BUILD_TARGET_64BIT)

	if (BUILD_TARGET_32BIT AND BUILD_TARGET_64BIT)
		merge_csu_objects(${csu_object_name}
			OBJ_NAME ${CSU_OBJECT_OBJ_NAME}
			
			ARCHS
				${APPLE_ARCH_32BIT}
				${APPLE_ARCH_64BIT}
		)
	elseif (BUILD_TARGET_64BIT)
		copy_csu_object(
			OBJ_NAME ${CSU_OBJECT_OBJ_NAME}
			ARCH ${APPLE_ARCH_64BIT}
		)
	elseif (BUILD_TARGET_32BIT)
		copy_csu_object(
			OBJ_NAME ${CSU_OBJECT_OBJ_NAME}
			ARCH ${APPLE_ARCH_32BIT}
		)
	endif (BUILD_TARGET_32BIT AND BUILD_TARGET_64BIT)
endfunction(csu_object)


# OS_MIN_V1	= -mmacosx-version-min=10.4
# $(SYMROOT)/crt1.v1.o: start.s crt.c dyld_glue.s 
# 	 $(CC) -r $(ARCH_CFLAGS) -Os $(OS_MIN_V1) -mdynamic-no-pic -nostdlib -keep_private_externs $^ -o $@  -DCRT -DOLD_LIBSYSTEM_SUPPORT
csu_object(crt1_v1
	OBJ_NAME "crt1.v1.o"
	OS_MIN_VERSION "10.4"

	SOURCES
		start.S
		crt.c
		dyld_glue.S

	DEFINITIONS
		CRT
		OLD_LIBSYSTEM_SUPPORT
	
	ADDITIONAL_FLAGS
		"-mdynamic-no-pic"
)

# OS_MIN_V2	= -mmacosx-version-min=10.5
# $(SYMROOT)/crt1.v2.o: start.s crt.c dyld_glue.s 
# 	$(CC) -r $(ARCH_CFLAGS) -Os $(OS_MIN_V2) -nostdlib -keep_private_externs $^ -o $@  -DCRT
csu_object(crt1_v2
	OBJ_NAME "crt1.v2.o"
	OS_MIN_VERSION "10.5"

	SOURCES
		start.S
		crt.c
		dyld_glue.S

	DEFINITIONS
		CRT
)

# OS_MIN_V3	= -mmacosx-version-min=10.6
# $(SYMROOT)/crt1.v3.o: start.s crt.c
# 	$(CC) -r $(ARCH_CFLAGS) -Os $(OS_MIN_V3) -nostdlib -keep_private_externs $^ -o $@  -DADD_PROGRAM_VARS 
csu_object(crt1_v3
	OBJ_NAME "crt1.v3.o"
	OS_MIN_VERSION "10.6"

	SOURCES
		start.S
		crt.c

	DEFINITIONS
		ADD_PROGRAM_VAR
)

# OS_MIN_V4	= -mmacosx-version-min=10.6
# $(SYMROOT)/crt1.v4.o: start.s crt.c
# 	$(CC) -r $(ARCH_CFLAGS) -Os $(OS_MIN_V4) -nostdlib -keep_private_externs $^ -o $@  -DADD_PROGRAM_VARS 
csu_object(crt1_v4
	OBJ_NAME "crt1.v4.o"
	OS_MIN_VERSION "10.6"

	SOURCES
		start.S
		crt.c

	DEFINITIONS
		ADD_PROGRAM_VAR
)

# $(SYMROOT)/crt0.o: start.s crt.c
# 	$(CC) -r $(ARCH_CFLAGS) -Os -static -Wl,-new_linker -nostdlib -keep_private_externs $^ -o $@ 
csu_object(crt0
	OBJ_NAME "crt0.o"

	SOURCES
		start.S
		crt.c
	
	ADDITIONAL_FLAGS
		"-static"
		"-Wl,-new_linker"
)

# OS_MIN_V1	= -mmacosx-version-min=10.4
# $(SYMROOT)/dylib1.v1.o: dyld_glue.s icplusplus.c
# 	$(CC) -r $(ARCH_CFLAGS) -Os $(OS_MIN_V1)  -nostdlib -keep_private_externs $^ -o $@  -DCFM_GLUE
csu_object(dylib1_v1
	OBJ_NAME "dylib1.v1.o"
	OS_MIN_VERSION "10.4"

	SOURCES
		dyld_glue.S
		icplusplus.c
	
	DEFINITIONS
		CFM_GLUE
)

# OS_MIN_V2	= -mmacosx-version-min=10.5
# $(SYMROOT)/dylib1.v2.o: dyld_glue.s
# 	$(CC) -r $(ARCH_CFLAGS) -Os $(OS_MIN_V2)  -nostdlib -keep_private_externs $^ -o $@  -DCFM_GLUE
csu_object(dylib1_v2
	OBJ_NAME "dylib1.v2.o"
	OS_MIN_VERSION "10.5"

	SOURCES
		dyld_glue.S
	
	DEFINITIONS
		CFM_GLUE
)

# OS_MIN_V1	= -mmacosx-version-min=10.4
# $(SYMROOT)/bundle1.v1.o: dyld_glue.s
# 	$(CC) -r $(ARCH_CFLAGS) -Os $(OS_MIN_V1)  -nostdlib -keep_private_externs $^ -o $@ 
csu_object(bundle1_v1
	OBJ_NAME "bundle1.v1.o"
	OS_MIN_VERSION "10.4"

	SOURCES
		dyld_glue.S
)

# $(SYMROOT)/lazydylib1.o: lazy_dylib_helper.s lazy_dylib_loader.c 
# 	$(CC) -r $(ARCH_CFLAGS) -Os -nostdlib -keep_private_externs $^ -o $@ 
csu_object(lazydylib1
	OBJ_NAME "lazydylib1.o"

	SOURCES
		lazy_dylib_helper.S
		lazy_dylib_loader.c 
)
