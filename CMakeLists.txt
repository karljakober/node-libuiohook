cmake_minimum_required(VERSION 3.5)
project(node-libuiohook)

################################################################################
# System & Utilities
################################################################################
# Detect Libary Suffix
IF(WIN32)
	SET(libExtension ".dll")
	add_definitions(-DWINVER=0x502 -D_WIN32_WINNT=0x502)
ELSEIF(APPLE)
	SET(libExtension ".dylib")
ELSE()
		SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
    SET(libExtension ".so")
ENDIF()

# Detect Architecture (Bitness)
math(EXPR BITS "8*${CMAKE_SIZEOF_VOID_P}")

################################################################################
# Code
################################################################################

# Includes
include_directories(
	${CMAKE_JS_INC}
	${UIOHOOKPATH}/include/
	${PROJECT_SOURCE_DIR}/node_modules/nan/
)

# Source
SET(PROJECT_HEADERS
	"${PROJECT_SOURCE_DIR}/source/hook.h"
)
SET(PROJECT_SOURCE
	"${PROJECT_SOURCE_DIR}/source/hook.cpp"
	"${PROJECT_SOURCE_DIR}/source/module.cpp"
)

ADD_LIBRARY(${PROJECT_NAME} SHARED ${PROJECT_SOURCE} ${PROJECT_HEADERS})

# Libraries
SET(node-libuiohook_LIBRARIES
)

# Binary Dependencies
SET(node-libuiohook_THIRDPARTY
	"${PROJECT_SOURCE_DIR}/index.js"
	"${PROJECT_SOURCE_DIR}/package.json"
)

################################################################################
# Platform Dependencies
################################################################################
IF(WIN32)
	# Windows
	add_definitions(-D_CRT_SECURE_NO_WARNINGS)

	LIST(APPEND node-libuiohook_THIRDPARTY
	)
endif()
################################################################################
# Building
################################################################################
SET_TARGET_PROPERTIES(${PROJECT_NAME} PROPERTIES PREFIX "" SUFFIX ".node")

TARGET_LINK_LIBRARIES(${PROJECT_NAME}
	${LIBUIOHOOK_LIBRARIES}
	${node-libuiohook_LIBRARIES}
	${CMAKE_JS_LIB}
)

add_library(node-libuiohook_THIRDPARTY SHARED IMPORTED)

# Distributable Version
ADD_CUSTOM_COMMAND(TARGET node-libuiohook POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy
	"$<TARGET_FILE:node-libuiohook>"
	"${node-libuiohook_BINARY_DIR}/distribute/node-libuiohook/$<TARGET_FILE_NAME:node-libuiohook>"
)
ADD_CUSTOM_COMMAND(TARGET node-libuiohook POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy
	${node-libuiohook_THIRDPARTY}
	"${node-libuiohook_BINARY_DIR}/distribute/node-libuiohook"
)
