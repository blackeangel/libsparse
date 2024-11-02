cmake_minimum_required(VERSION 3.20)

project(libsparse)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_C_STANDARD 17)
set(DEFAULT_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS_RELEASE "-pipe -O2 -s")


set(BUILD_SHARED_LIBS OFF)
set(CMAKE_EXE_LINKER_FLAGS "-static")

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/out)

include(GNUInstallDirs)

include_directories(
		${CMAKE_CURRENT_SOURCE_DIR}/android-base
		${CMAKE_CURRENT_SOURCE_DIR})

# Section: zlib
set(ZLIB_SRC_FILES ${CMAKE_CURRENT_SOURCE_DIR}/zlib/adler32.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/compress.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/crc32.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/deflate.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/gzclose.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/gzlib.c
${CMAKE_CURRENT_SOURCE_DIR}//zlib/gzread.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/gzwrite.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/infback.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/inffast.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/inflate.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/inftrees.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/trees.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/uncompr.c
${CMAKE_CURRENT_SOURCE_DIR}/zlib/zutil.c)
add_library(zlib STATIC ${ZLIB_SRC_FILES})

# Section: libsparse
set(SPARSE_SRC_FILES ${CMAKE_CURRENT_SOURCE_DIR}/backed_block.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/output_file.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/sparse.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/sparse_crc32.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/sparse_err.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/sparse_read.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/android-base/stringprintf.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/android-base/mapped_file.cpp)
add_library(sparse STATIC ${SPARSE_SRC_FILES})
target_link_libraries(sparse zlib)
set_target_properties(sparse PROPERTIES
	VERSION ${PROJECT_VERSION}
	SOVERSION ${PROJECT_VERSION}
	PUBLIC_HEADER sparse.h)

set(CMAKE_FIND_LIBRARY_SUFFIXES ".a")

# Section: simg2img
set(SIMG2IMG_SRC_FILES ${CMAKE_CURRENT_SOURCE_DIR}/simg2img.cpp)
add_executable(simg2img ${SIMG2IMG_SRC_FILES})
target_link_libraries(simg2img sparse)

# Section: simg2simg
set(SIMG2SIMG_SRC_FILES ${CMAKE_CURRENT_SOURCE_DIR}/simg2simg.cpp)
add_executable(simg2simg ${SIMG2SIMG_SRC_FILES})
target_link_libraries(simg2simg sparse)

# Section: img2simg
set(IMG2SIMG_SRC_FILES ${CMAKE_CURRENT_SOURCE_DIR}/img2simg.cpp)
add_executable(img2simg ${IMG2SIMG_SRC_FILES})
target_link_libraries(img2simg sparse)

# Section: append2simg
set(APPEND2SIMG_SRC_FILES ${CMAKE_CURRENT_SOURCE_DIR}/append2simg.cpp
		android-base/mapped_file.cpp)
add_executable(append2simg ${APPEND2SIMG_SRC_FILES})
target_link_libraries(append2simg sparse)

# Install products
install(TARGETS sparse simg2img simg2simg img2simg append2simg
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# Install pkg-config files
#configure_file(libsparse.pc.in libsparse.pc @ONLY)
#install(FILES ${CMAKE_BINARY_DIR}/libsparse.pc
#	DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig)
