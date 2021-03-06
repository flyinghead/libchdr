cmake_minimum_required(VERSION 3.1)

project(chdr C)

option(BUILD_SHARED_LIBS "Build libchdr also as a shared library" ON)
option(WITH_SYSTEM_FLAC "Use system provided FLAC library" OFF)
option(WITH_SYSTEM_ZLIB "Use system provided zlib library" OFF)

include(FindPkgConfig)

#--------------------------------------------------
# dependencies
#--------------------------------------------------

# crypto
set(CRYPTO_SOURCES
  deps/crypto/md5.c
  deps/crypto/sha1.c)

if (WITH_SYSTEM_FLAC)
  pkg_check_modules(FLAC REQUIRED flac)
  list(APPEND CHDR_INCLUDES ${FLAC_INCLUDE_DIRS})
  list(APPEND CHDR_LIBS ${FLAC_LINK_LIBRARIES})
else()
  set(WITH_OGG OFF CACHE BOOL "ogg support (default: test for libogg)")
  set(BUILD_SHARED_LIBS_SAVED "${BUILD_SHARED_LIBS}")
  set(BUILD_SHARED_LIBS OFF)
  add_subdirectory(deps/flac-1.3.3)
  set(BUILD_SHARED_LIBS "${BUILD_SHARED_LIBS_SAVED}")
  list(APPEND CHDR_LIBS FLAC)
endif()

add_library(crypto-static STATIC ${CRYPTO_SOURCES})
list(APPEND CHDR_INCLUDES deps/crypto)
list(APPEND CHDR_LIBS crypto-static)

# lzma
set(LZMA_SOURCES
  #deps/lzma-16.04/C/7zAlloc.c
  #deps/lzma-16.04/C/7zArcIn.c
  #deps/lzma-16.04/C/7zBuf2.c
  #deps/lzma-16.04/C/7zBuf.c
  #deps/lzma-16.04/C/7zCrc.c
  #deps/lzma-16.04/C/7zCrcOpt.c
  #deps/lzma-16.04/C/7zDec.c
  #deps/lzma-16.04/C/7zFile.c
  #deps/lzma-16.04/C/7zStream.c
  #deps/lzma-16.04/C/Aes.c
  #deps/lzma-16.04/C/AesOpt.c
  deps/lzma-16.04/C/Alloc.c
  #deps/lzma-16.04/C/Bcj2.c
  #deps/lzma-16.04/C/Bcj2Enc.c
  deps/lzma-16.04/C/Bra86.c
  deps/lzma-16.04/C/Bra.c
  deps/lzma-16.04/C/BraIA64.c
  deps/lzma-16.04/C/CpuArch.c
  deps/lzma-16.04/C/Delta.c
  #deps/lzma-16.04/C/DllSecur.c
  deps/lzma-16.04/C/LzFind.c
  #deps/lzma-16.04/C/LzFindMt.c
  #deps/lzma-16.04/C/Lzma2Dec.c
  #deps/lzma-16.04/C/Lzma2Enc.c
  deps/lzma-16.04/C/Lzma86Dec.c
  deps/lzma-16.04/C/Lzma86Enc.c
  deps/lzma-16.04/C/LzmaDec.c
  deps/lzma-16.04/C/LzmaEnc.c
  deps/lzma-16.04/C/LzmaLib.c
  #deps/lzma-16.04/C/MtCoder.c
  #deps/lzma-16.04/C/Ppmd7.c
  #deps/lzma-16.04/C/Ppmd7Dec.c
  #deps/lzma-16.04/C/Ppmd7Enc.c
  #deps/lzma-16.04/C/Sha256.c
  deps/lzma-16.04/C/Sort.c
  #deps/lzma-16.04/C/Threads.c
  #deps/lzma-16.04/C/Xz.c
  #deps/lzma-16.04/C/XzCrc64.c
  #deps/lzma-16.04/C/XzCrc64Opt.c
  #deps/lzma-16.04/C/XzDec.c
  #deps/lzma-16.04/C/XzEnc.c
  #deps/lzma-16.04/C/XzIn.c
				)

add_library(lzma-static STATIC ${LZMA_SOURCES})
target_compile_options(lzma-static PRIVATE -D_7ZIP_ST)
list(APPEND CHDR_INCLUDES deps/lzma-16.04/C)
list(APPEND CHDR_LIBS lzma-static)

# zlib
if (WITH_SYSTEM_ZLIB)
  pkg_check_modules(ZLIB REQUIRED zlib)
  list(APPEND CHDR_INCLUDES ${ZLIB_INCLUDE_DIRS})
  list(APPEND CHDR_LIBS ${ZLIB_LINK_LIBRARIES})
else()
  add_subdirectory(deps/zlib-1.2.11 EXCLUDE_FROM_ALL)
  list(APPEND CHDR_INCLUDES deps/zlib-1.2.11 ${CMAKE_CURRENT_BINARY_DIR}/deps/zlib-1.2.11)
  list(APPEND CHDR_LIBS zlibstatic)
endif()

#--------------------------------------------------
# chdr
#--------------------------------------------------

set(CHDR_SOURCES
  src/bitstream.c
  src/bitstream.h
  src/cdrom.c
  src/cdrom.h
  src/chd.c
  src/chd.h
  src/coretypes.h
  src/flac.c
  src/flac.h
  src/huffman.c
  src/huffman.h)

add_library(chdr-static STATIC ${CHDR_SOURCES})
target_include_directories(chdr-static PRIVATE ${CHDR_INCLUDES})
target_compile_definitions(chdr-static PRIVATE ${CHDR_DEFS})
target_link_libraries(chdr-static ${CHDR_LIBS})

if (BUILD_SHARED_LIBS)
  add_library(chdr-shared SHARED ${CHDR_SOURCES})
  target_include_directories(chdr-shared PRIVATE ${CHDR_INCLUDES})
  target_compile_definitions(chdr-shared PRIVATE ${CHDR_DEFS})
  target_link_libraries(chdr-shared ${CHDR_LIBS})
endif()
