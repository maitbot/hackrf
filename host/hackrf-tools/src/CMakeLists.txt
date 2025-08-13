# Copyright 2012 Jared Boone
# Copyright 2013 Benjamin Vernoux
#
# This file is part of HackRF.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2, or (at your option)
# any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; see the file COPYING.  If not, write to
# the Free Software Foundation, Inc., 51 Franklin Street,
# Boston, MA 02110-1301, USA.
#

find_package(FFTW3f REQUIRED)

SET(TOOLS
	hackrf_transfer
	hackrf_spiflash
	hackrf_cpldjtag
	hackrf_info
	hackrf_debug
	hackrf_clock
	hackrf_sweep
	hackrf_operacake
	hackrf_biast
)

if(NOT libhackrf_SOURCE_DIR)
        LIST(APPEND TOOLS_LINK_LIBS HackRF::hackrf)
else()
	LIST(APPEND TOOLS_LINK_LIBS hackrf)
endif()

if(MSVC)
	add_library(libgetopt_static STATIC
	    ../getopt/getopt.c
	)
	LIST(APPEND TOOLS_LINK_LIBS libgetopt_static)
endif()

foreach(tool ${TOOLS})
        add_executable(${tool} ${tool}.c)
        target_compile_features(${tool} PRIVATE c_std_90)
	target_link_libraries(${tool} ${TOOLS_LINK_LIBS} fftw3f::fftw3f)
	install(TARGETS ${tool} RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
endforeach(tool)

if( ${WIN32} )
	install(DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/$<CONFIGURATION>/"
	  DESTINATION ${CMAKE_INSTALL_BINDIR}
	  FILES_MATCHING
	  PATTERN "fftw*.dll")
endif( ${WIN32} )
