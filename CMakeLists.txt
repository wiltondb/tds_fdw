# Copyright 2024 alex@staticlibs.net
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required ( VERSION 3.0 )

project ( tds_fdw C )

if ( NOT DEFINED ENV{PGWIN_DEPS_DIR} )
    message( FATAL_ERROR "Environment variable 'PGWIN_DEPS_DIR' (pointing to the pgwin_deps installation directory) must be specified" )
endif ()
if ( NOT DEFINED ENV{PGWIN_INSTALL_DIR} )
    message( FATAL_ERROR "Environment variable 'PGWIN_INSTALL_DIR' (pointing to the postgresql_modified_for_babelfish installation directory) must be specified" )
endif ()

configure_file (
    sql/tds_fdw.sql
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}--2.0.3.sql
    COPYONLY )

add_library ( ${PROJECT_NAME} SHARED 
    src/deparse.c
    src/options.c
    src/tds_fdw.c

    include/deparse.h
    include/options.h
    include/tds_fdw.h
    include/visibility.h

    ${PROJECT_NAME}.def )

target_include_directories ( ${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}/include
    $ENV{PGWIN_DEPS_DIR}/freetds/include
    $ENV{PGWIN_INSTALL_DIR}/include/server/port/win32_msvc 
    $ENV{PGWIN_INSTALL_DIR}/include/server/port/win32
    $ENV{PGWIN_INSTALL_DIR}/include/server
    $ENV{PGWIN_INSTALL_DIR}/include )

target_compile_options ( ${PROJECT_NAME} PRIVATE
    /wd4244
    /wd4267 )

target_compile_definitions ( ${PROJECT_NAME} PRIVATE
    _CRT_SECURE_NO_WARNINGS )
 
target_link_libraries ( ${PROJECT_NAME} PRIVATE
    $ENV{PGWIN_DEPS_DIR}/freetds/lib/sybdb.lib
    $ENV{PGWIN_INSTALL_DIR}/lib/postgres.lib )

install ( TARGETS ${PROJECT_NAME} RUNTIME
    DESTINATION $ENV{PGWIN_INSTALL_DIR}/lib )

install ( FILES
    $ENV{PGWIN_DEPS_DIR}/iconv/bin/iconv.dll
    $ENV{PGWIN_DEPS_DIR}/freetds/bin/sybdb.dll
    DESTINATION $ENV{PGWIN_INSTALL_DIR}/bin )

install( FILES
    $<TARGET_PDB_FILE:${PROJECT_NAME}>
    $ENV{PGWIN_DEPS_DIR}/iconv/bin/iconv.pdb
    $ENV{PGWIN_DEPS_DIR}/freetds/bin/sybdb.pdb
    DESTINATION $ENV{PGWIN_INSTALL_DIR}/symbols )

install ( FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}--2.0.3.sql
    ${PROJECT_NAME}.control
    DESTINATION $ENV{PGWIN_INSTALL_DIR}/share/extension )