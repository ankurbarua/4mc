PROJECT(4MC C)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "4mc Four More Compression")
set(CPACK_PACKAGE_VERSION_MAJOR 1)
set(CPACK_PACKAGE_VERSION_MINOR 0)
set(CPACK_PACKAGE_VERSION_PATCH 0)
set(VERSION_STRING	" \"${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}\" ")
include(CPack)

cmake_minimum_required (VERSION 2.6)
INCLUDE (CheckTypeSize)
check_type_size("void *" SIZEOF_VOID_P)
IF( ${SIZEOF_VOID_P} STREQUAL  "8" )
    set (CMAKE_SYSTEM_PROCESSOR "64bit")
    MESSAGE( STATUS "64 bit architecture detected size of void * is " ${SIZEOF_VOID_P})
ENDIF()

option(BUILD_TOOLS "Build the command line tools" ON)
option(BUILD_LIBS  "Build the JNI native library for hadoop-4mc" ON)

if(UNIX AND NOT APPLE AND BUILD_LIBS)
    if(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64")
        add_definitions(-fPIC)
    endif()
endif()

set(LZ4_DIR ../lz4/)
set(PRG_DIR ../)
set(LZ4_SRCS ${LZ4_DIR}xxhash.h ${LZ4_DIR}xxhash.c ${LZ4_DIR}lz4.c ${LZ4_DIR}lz4hc.c ${LZ4_DIR}lz4.h ${LZ4_DIR}lz4hc.h ${LZ4_DIR}lz4mc.c ${LZ4_DIR}lz4mc.h)
set(4MC_SRCS ${PRG_DIR}4mccli.c ${PRG_DIR}4mc.c)
set(HADOOP4MC_SRCS ${PRG_DIR}jniCompressor.c ${PRG_DIR}jniDecompressor.c)

if(NOT WIN32)
	add_definitions(-O3)
	add_definitions("-std=c99")
endif()
	
if(BUILD_TOOLS)
    add_executable(4mc ${LZ4_SRCS} ${4MC_SRCS})
    install(TARGETS 4mc RUNTIME DESTINATION "bin/")
endif()

if(BUILD_LIBS)
	find_package(JNI)
	if (JNI_FOUND)
		message (STATUS "JNI_INCLUDE_DIRS=${JNI_INCLUDE_DIRS}")
		message (STATUS "JNI_LIBRARIES=${JNI_LIBRARIES}")
		INCLUDE_DIRECTORIES(${JNI_INCLUDE_DIRS})
	else()
		MESSAGE(FATAL_ERROR "Found JNI: failed")
	endif()	
    
    add_library(hadoop-4mc SHARED ${LZ4_SRCS} ${HADOOP4MC_SRCS})
    install(TARGETS 4mc RUNTIME DESTINATION "lib/")
endif()

#warnings

ADD_DEFINITIONS("-Wall")
ADD_DEFINITIONS("-DLZ4_VERSION=\"1.2.1\"")
INCLUDE_DIRECTORIES (${LZ4_DIR})



