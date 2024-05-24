cmake_minimum_required(VERSION 3.18)
project(cifra C)

cmake_policy(SET CMP0048 NEW)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_EXTENSIONS ON)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include(GNUInstallDirs)

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_C_FLAGS "-Wall -Wextra -Wno-missing-field-initializers -Wno-unused-parameter -pipe -fstack-protector-all")
set(CMAKE_C_FLAGS_DEBUG "-O0 -g -ggdb -gdwarf-2 -gstrict-dwarf -fno-omit-frame-pointer -fno-inline")
set(CMAKE_C_FLAGS_RELEASE "-O2")

option(CIFRA_BUILD_LIBRARY_SHARED "Build shared library"   ON)
option(CIFRA_BUILD_LIBRARY_STATIC "Build static library"   OFF)
option(CIFRA_BUILD_TEST           "Build test application" ON)

set(CIFRA_SOURCE_FILES
	src/aes.c
	src/sha256.c
	src/sha512.c
	src/chash.c
	src/hmac.c
	src/pbkdf2.c
	src/modes.c
	src/eax.c
	src/gf128.c
	src/blockwise.c
	src/cmac.c
	src/salsa20.c
	src/chacha20.c
	src/curve25519.c
	src/gcm.c
	src/cbcmac.c
	src/ccm.c
	src/sha3.c
	src/sha1.c
	src/poly1305.c
	src/norx.c
	src/chacha20poly1305.c
	src/drbg.c
	src/ocb.c
	src/sha3_shake.c
)

MESSAGE(STATUS "Build type:                    " ${CMAKE_BUILD_TYPE})
MESSAGE(STATUS "Build shared library:          " ${CIFRA_BUILD_LIBRARY_SHARED})
MESSAGE(STATUS "Build static library:          " ${CIFRA_BUILD_LIBRARY_STATIC})
MESSAGE(STATUS "Build test app:                " ${CIFRA_BUILD_TEST})

add_library(cifra OBJECT ${CIFRA_SOURCE_FILES})
set_target_properties(cifra PROPERTIES POSITION_INDEPENDENT_CODE ON)
target_include_directories(cifra PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include/cifra")
target_include_directories(cifra PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/src/ext")

configure_file(
	"${CMAKE_CURRENT_SOURCE_DIR}/cifra.pc.in"
	"${CMAKE_CURRENT_BINARY_DIR}/cifra.pc"
	@ONLY
)

# Shared lib
if (CIFRA_BUILD_LIBRARY_SHARED)
	add_library(cifra_shared SHARED $<TARGET_OBJECTS:cifra>)
	set_target_properties(cifra_shared PROPERTIES OUTPUT_NAME cifra)
	install(TARGETS cifra_shared LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif()

# Static lib
if (CIFRA_BUILD_LIBRARY_STATIC)
	add_library(cifra_static STATIC $<TARGET_OBJECTS:cifra>)
	set_target_properties(cifra_static PROPERTIES OUTPUT_NAME cifra)
	install(TARGETS cifra_static LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif()

if (CIFRA_BUILD_TEST)
	set(CIFRA_TEST_APPS
		testaes
		testmodes
		testsha1
		testsha2
		testsha3
		testshake
		testsalsa20
		testcurve25519
		testpoly1305
		testnorx
		testchacha20poly1305
		testdrbg
	)

	foreach(APP IN LISTS CIFRA_TEST_APPS)
		add_executable(${APP} "src/${APP}.c")
		target_link_libraries(${APP} cifra)
	endforeach()
endif()

if (CIFRA_BUILD_LIBRARY_SHARED OR CIFRA_BUILD_LIBRARY_STATIC)
	install(FILES ${CMAKE_CURRENT_BINARY_DIR}/cifra.pc DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig)
	install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/cifra"
		DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
	)
endif()
