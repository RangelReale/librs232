# Copyright (C) 2007-2011 LuaDist.
# Created by David Manura
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.

project ( librs232 C )
cmake_minimum_required ( VERSION 2.6 )
include ( cmake/dist.cmake )
include ( lua )
install_header ( include/librs232/rs232.h include/librs232/rs232_posix.h INTO librs232 )

# LIBVERSION 0:1:0
set ( SOVERSION 0 )
set ( VERSION 0.0.1 )

if ( CMAKE_BUILD_TYPE STREQUAL Debug )
  add_definitions ( -DRS232_DEBUG )
endif ( )

include_directories ( include )
#unused: AM_CFLAGS = -Wall -ansi -pedantic -W -Wmissing-prototypes -Wmissing-declarations -Werror -std=gnu99 

if ( WIN32 OR CYGWIN )
  set ( SRCS src/rs232_windows.c src/resource.rc )
  add_definitions ( -DRS232_EXPORT )
else ( )
  set ( SRCS src/rs232_posix.c )
endif ( )

add_library ( rs232 src/rs232.c ${SRCS} )
#unused: set_target_properties ( rs232 PROPERTIES VERSION ${VERSION} )
#unused: set_target_properties ( rs232 PROPERTIES SOVERSION ${SOVERSION})
#set_target_properties ( rs232 PROPERTIES LINK_FLAGS -nostartfiles ) # is this really useful, other than saving nine bytes?
install_library ( rs232 )

if ( CMAKE_BUILD_TYPE STREQUAL Debug )
  add_executable ( rs232_test src/rs232_test.c )
  target_link_libraries ( rs232_test rs232 )
  install_executable ( rs232_test )
  if ( NOT SKIP_TESTING )
    include ( CTest )
    #add_test ( run_rs232_test rs232_test ) # fails: file not found
  endif ( )
endif ( )

# Install package config file (improve?)
#set ( prefix ${CMAKE_INSTALL_PREFIX} )
#set ( exec_prefix \${prefix} )
#set ( libdir \${exec_prefix}/lib )
#set ( includedir \${prefix}/include )
#set ( sysincludedir \${prefix}/include ) # improve?
#set ( LIBS )
#configure_file ( src/librs232.pc.in librs232.pc @ONLY )
#install ( FILES ${CMAKE_CURRENT_BINARY_DIR}/librs232.pc DESTINATION ${INSTALL_LIB}/pkgconfig )

# Lua binding (optional)
find_package ( Lua )
if ( LUA_FOUND )
  install_lua_module ( luars232 bindings/lua/luars232.c src/resource_lua.rc LINK rs232 )
  #set_target_properties ( luars232 PROPERTIES LINK_FLAGS -nostartfiles )
  # note: Makefile.am versions this (VERSION/SOVERSION) too, which creates symlinks, but I'll omit that.

  # This takes way too long to run.
  #add_lua_test ( bindings/lua/test.lua )
endif ( )

install_data ( AUTHORS ChangeLog COPYING TODO )
install_example ( doc/example.lua )

