# Icinga 2
# Copyright (C) 2012-2016 Icinga Development Team (https://www.icinga.org/)
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software Foundation
# Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301, USA.

find_package(BISON 2.3.0 REQUIRED)
find_package(FLEX 2.5.31 REQUIRED)

bison_target(class_parser class_parser.yy ${CMAKE_CURRENT_BINARY_DIR}/class_parser.cc)
flex_target(class_lexer class_lexer.ll ${CMAKE_CURRENT_BINARY_DIR}/class_lexer.cc)
add_flex_bison_dependency(class_lexer class_parser)

if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  set_property(SOURCE ${CMAKE_CURRENT_BINARY_DIR}/class_parser.cc PROPERTY COMPILE_FLAGS "-Wno-deprecated-register")
  set_property(SOURCE ${CMAKE_CURRENT_BINARY_DIR}/class_lexer.cc PROPERTY COMPILE_FLAGS "-Wno-deprecated-register -Wno-null-conversion")
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR})

add_executable(mkclass mkclass.cpp classcompiler.cpp classcompiler.hpp ${FLEX_class_lexer_OUTPUTS} ${BISON_class_parser_OUTPUTS})

if(WIN32)
  target_link_libraries(mkclass shlwapi)
endif()

set_target_properties (
  mkclass PROPERTIES
  FOLDER Bin
)

macro(MKCLASS_TARGET ClassInput ClassImplOutput ClassHeaderOutput)
    add_custom_command(
        OUTPUT ${ClassImplOutput} ${ClassHeaderOutput}
        COMMAND mkclass
        ARGS ${ClassInput} ${CMAKE_CURRENT_BINARY_DIR}/${ClassImplOutput} ${CMAKE_CURRENT_BINARY_DIR}/${ClassHeaderOutput}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        DEPENDS mkclass ${ClassInput}
    )
endmacro()
