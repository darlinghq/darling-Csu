project(csu)

cmake_minimum_required(VERSION 3.10)

enable_language(C ASM)

if(COMMAND cmake_policy)
	cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

set(OS_MIN_V1 "-mmacosx-version-min=10.4")
set(OS_MIN_V2 "-mmacosx-version-min=10.5")
set(OS_MIN_V3 "-mmacosx-version-min=10.6")
set(OS_MIN_V4 "-mmacosx-version-min=10.6")

set(CMAKE_C_FLAGS_BACKUP ${CMAKE_C_FLAGS})

set(crt1_v1_source start.S crt.c dyld_glue.S)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS_BACKUP} -Os ${OS_MIN_V1}  -mdynamic-no-pic -nostdlib -keep_private_externs-DCRT -DOLD_LIBSYSTEM_SUPPORT ")
# add_darling_static_library(crt1.v1 FAT SOURCES ${crt1_v1_source})
add_darling_static_library(crt1 FAT SOURCES ${crt1_v1_source})

set(crt1_v2_source start.S crt.c dyld_glue.S)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS_BACKUP} -Os ${OS_MIN_V2} -nostdlib -keep_private_externs -DCRT ")
# add_darling_static_library(crt1.v2 FAT SOURCES ${crt1_v2_source})
add_darling_static_library(crt1.10.5 FAT SOURCES ${crt1_v2_source})

set(crt1_v3_source start.S crt.c)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS_BACKUP} -Os ${OS_MIN_V3} -nostdlib -keep_private_externs -DADD_PROGRAM_VARS ")
# add_darling_static_library(crt1.v3 FAT SOURCES ${crt1_v3_source})
add_darling_static_library(crt1.10.6 FAT SOURCES ${crt1_v3_source})

# set(crt1_v4_source start.S crt.c)
# set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS_BACKUP} -Os ${OS_MIN_V4} -nostdlib -keep_private_externs -DADD_PROGRAM_VARS ")
# add_darling_static_library(crt1.v4 FAT SOURCES ${crt1_v4_source})

set(crt0_source start.S crt.c)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS_BACKUP} -Os -nostdlib -keep_private_externs -DADD_PROGRAM_VARS ")
add_darling_static_library(crt0 FAT SOURCES ${crt0_source})

set(dylib1_v1_source dyld_glue.S icplusplus.c)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS_BACKUP} -Os ${OS_MIN_V1}  -nostdlib -keep_private_externs -DCFM_GLUE ")
# add_darling_static_library(dylib1.v1 FAT SOURCES ${dylib1_v1_source})
add_darling_static_library(dylib1 FAT SOURCES ${dylib1_v1_source})

set(dylib1_v2_source dyld_glue.S)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS_BACKUP} -Os ${OS_MIN_V2}  -nostdlib -keep_private_externs -DCFM_GLUE ")
# add_darling_static_library(dylib1.v2 FAT SOURCES ${dylib1_v2_source})
add_darling_static_library(dylib1.10.5 FAT SOURCES ${dylib1_v2_source})

set(bundle1_v1_source dyld_glue.S)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS_BACKUP} -Os ${OS_MIN_V1}  -nostdlib -keep_private_externs ")
# add_darling_static_library(bundle1.v1 FAT SOURCES ${bundle1_v1_source})
add_darling_static_library(bundle1 FAT SOURCES ${bundle1_v1_source})

set(lazydylib1_source lazy_dylib_helper.S lazy_dylib_loader.c)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS_BACKUP} -Os -nostdlib -keep_private_externs ")
add_darling_static_library(lazydylib1 FAT SOURCES ${lazydylib1_source})
