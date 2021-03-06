# WinPR: Windows Portable Runtime
# winpr cmake build script
#
# Copyright 2012 Marc-Andre Moreau <marcandre.moreau@gmail.com>
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required(VERSION 2.8)

project(WinPR C)

set(CMAKE_COLOR_MAKEFILE ON)

if(FREERDP_VERSION)
	set(FREERDP_BUILD 1)
endif()

# Include cmake modules
include(CheckIncludeFiles)
include(CheckLibraryExists)
include(CheckStructHasMember)
include(FindPkgConfig)
include(TestBigEndian)

# Include our extra modules
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../cmake/)

# Check for cmake compatibility (enable/disable features)
include(CheckCmakeCompat)
include(FindFeature)
include(AutoVersioning)
include(ConfigOptions)
include(CheckCCompilerFlag)
include(GNUInstallDirsWrapper)
include(CMakePackageConfigHelpers)

# Soname versioning
set(WINPR_VERSION_MAJOR "1")
set(WINPR_VERSION_MINOR "1")
set(WINPR_VERSION_REVISION "0")
set(WINPR_VERSION "${WINPR_VERSION_MAJOR}.${WINPR_VERSION_MINOR}")
set(WINPR_VERSION_FULL "${WINPR_VERSION}.${WINPR_VERSION_REVISION}")
set(WINPR_VERSION_FULL ${WINPR_VERSION_FULL} PARENT_SCOPE)

# Default to release build type
if(NOT CMAKE_BUILD_TYPE)
   set(CMAKE_BUILD_TYPE "Release")
endif()

# Default to build shared libs
if(NOT DEFINED BUILD_SHARED_LIBS)
	set(BUILD_SHARED_LIBS ON)
endif()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DWINPR_EXPORTS")

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
include_directories(${CMAKE_CURRENT_BINARY_DIR}/include)

add_subdirectory(include)

add_subdirectory(libwinpr)

if(NOT ANDROID AND NOT IOS)
	add_subdirectory(tools)
endif()

# Exporting

if(${CMAKE_VERSION} VERSION_GREATER "2.8.10")

	export(PACKAGE winpr)

	set(WINPR_CMAKE_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/WinPR")

	set(WINPR_INCLUDE_DIR "include")
	# Keep this for legacy builds
	set(WINPR_MONOLITHIC_BUILD OFF)

	configure_package_config_file(WinPRConfig.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/WinPRConfig.cmake
		INSTALL_DESTINATION ${WINPR_CMAKE_INSTALL_DIR}
		PATH_VARS WINPR_INCLUDE_DIR WINPR_MONOLITHIC_BUILD)

	write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/WinPRConfigVersion.cmake
		VERSION ${WINPR_VERSION} COMPATIBILITY SameMajorVersion)

	install(FILES ${CMAKE_CURRENT_BINARY_DIR}/WinPRConfig.cmake ${CMAKE_CURRENT_BINARY_DIR}/WinPRConfigVersion.cmake
		DESTINATION ${WINPR_CMAKE_INSTALL_DIR})

	install(EXPORT WinPRTargets DESTINATION ${WINPR_CMAKE_INSTALL_DIR})

endif()

