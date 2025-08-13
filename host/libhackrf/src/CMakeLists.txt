#
# Copyright (c) 2012, Jared Boone <jared@sharebrained.com>
# Copyright (c) 2013, Benjamin Vernoux <titanmkd@gmail.com>
# Copyright (c) 2013, Michael Ossmann <mike@ossmann.com>
# 
# All rights reserved.
# 
# Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:
# 
#     Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.
#     Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the 
# 	documentation and/or other materials provided with the distribution.
#     Neither the name of Great Scott Gadgets nor the names of its contributors may be used to endorse or promote products derived from this software
# 	without specific prior written permission.
# 
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, 
# THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED.
# IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
# HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#

# Dynamic library
add_library(hackrf SHARED hackrf.c)
set_target_properties(hackrf PROPERTIES
  VERSION ${PROJECT_VERSION}
  SOVERSION ${PROJECT_VERSION_MAJOR})

# Static library
add_library(hackrf-static STATIC hackrf.c)
if(MSVC)
	set_target_properties(hackrf-static PROPERTIES OUTPUT_NAME "hackrf_static")
else()
	set_target_properties(hackrf-static PROPERTIES OUTPUT_NAME "hackrf")
endif()

target_compile_features(hackrf PRIVATE c_std_90)
target_compile_features(hackrf-static PRIVATE c_std_90)

set_target_properties(hackrf PROPERTIES CLEAN_DIRECT_OUTPUT 1)
set_target_properties(hackrf-static PROPERTIES CLEAN_DIRECT_OUTPUT 1)

target_compile_definitions(hackrf PRIVATE
  -DLIBRARY_VERSION="${PROJECT_VERSION}"
  -DLIBRARY_RELEASE="${RELEASE}")
target_compile_definitions(hackrf-static PRIVATE
  -DLIBRARY_VERSION="${PROJECT_VERSION}"
  -DLIBRARY_RELEASE="${RELEASE}")

target_include_directories(
  hackrf PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
                $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/libhackrf>)
target_include_directories(
  hackrf-static PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
                $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/libhackrf>)

# Dependencies
target_link_libraries(hackrf PRIVATE LIBUSB::LIBUSB Threads::Threads)
target_link_libraries(hackrf-static PUBLIC LIBUSB::LIBUSB Threads::Threads)

generate_export_header(hackrf)

# For cygwin just force UNIX OFF and WIN32 ON
if( ${CYGWIN} )
  SET(UNIX OFF)
  SET(WIN32 ON)
endif( ${CYGWIN} )

if( ${UNIX} )
   install(TARGETS hackrf EXPORT HackRFTargets
           LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
           COMPONENT sharedlibs
           )
   install(TARGETS hackrf-static EXPORT HackRFTargets
           ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
           COMPONENT staticlibs
           )
endif( ${UNIX} )

if( ${WIN32} )
   install(TARGETS hackrf
           DESTINATION bin
           COMPONENT sharedlibs
           )
   install(DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/$<CONFIGURATION>/"
           DESTINATION bin
           FILES_MATCHING
           PATTERN "libusb*.dll"
           PATTERN "pthread*.dll"
           )
   install(TARGETS hackrf-static
           DESTINATION bin
           COMPONENT staticlibs
           )
endif( ${WIN32} )

install(FILES hackrf.h
#  "${CMAKE_CURRENT_BINARY_DIR}/hackrf_export.h"
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
  COMPONENT headers
)
