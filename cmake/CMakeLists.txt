set (PACKAGE dbus)
set (VERSION_MAJOR "1")
set (VERSION_MINOR "0")
set (VERSION_PATCH "0")
set (VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH} )
project(${PACKAGE})

# we need to be up to date
CMAKE_MINIMUM_REQUIRED(VERSION 2.4.4 FATAL_ERROR)

# where to look first for cmake modules, before ${CMAKE_ROOT}/Modules/ is checked
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/modules")


#enable building of shared library
SET(BUILD_SHARED_LIBS ON)

# search packages used by KDE
if (WIN32)
	find_package(GNUWIN32)
	find_package(LibIconv)
endif (WIN32)
find_package(LibXml2)
find_package(LibExpat)
find_package(X11)

# do config checks
INCLUDE(ConfigureChecks.cmake)

# @TODO: how to remove last dir from ${CMAKE_SOURCE_DIR} ? 
SET(DBUS_SOURCE_DIR ${CMAKE_SOURCE_DIR}/..)

# make some more macros available
include (MacroLibrary)

if(VCS)
	set(DBUS_VERBOSE_C_S 1 CACHE TYPE STRING FORCE)
	set(DBUS_VERBOSE_C_S 1)
endif(VCS)

if(MSVC)
	# controll folders in msvc projects
	include(ProjectSourceGroup)
	if(NOT GROUP_CODE)
		#set(GROUP_CODE split) #cmake default
		set(GROUP_CODE flat)
	endif(NOT GROUP_CODE)
	ADD_DEFINITIONS(-D_CRT_SECURE_NO_DEPRECATE -D_CRT_NONSTDC_NO_DEPRECATE)
	
	
	# Use the highest warning level
	if (WALL)
		set(WALL 1 CACHE TYPE STRING FORCE)
		set(CMAKE_CXX_WARNING_LEVEL 4 CACHE TYPE STRING FORCE)
		
		if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
			STRING(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
		else(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
			SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
		endif(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
		
		if(CMAKE_C_FLAGS MATCHES "/W[0-4]")
			STRING(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
		else(CMAKE_C_FLAGS MATCHES "/W[0-4]")
			SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /W4")
		endif(CMAKE_C_FLAGS MATCHES "/W[0-4]")
	else (WALL)
		set(CMAKE_CXX_WARNING_LEVEL 3 CACHE TYPE STRING FORCE)
	endif (WALL)
	
	SET(MSVC_W_ERROR   " /we4028 /we4013 /we4133 /we4047 /we4031 /we4002 /we4003 /we4114")
	SET(MSVC_W_DISABLE " /wd4127 /wd4090 /wd4101 /wd4244")

	SET(CMAKE_C_FLAGS_DEBUG   "${CMAKE_C_FLAGS_DEBUG}   /FIconfig.h ${MSVC_W_ERROR} ${MSVC_W_DISABLE}")
	SET(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /FIconfig.h ${MSVC_W_ERROR} ${MSVC_W_DISABLE}")
endif(MSVC)


#########################################################################
# Windows CE
#
# usage:
#      cmake ..\trunk\cmake -Dwince=1 -Dwcelibcex=c:\wcelibcex
#
# change configuration in Visual Studio to 'Pocket PC 2003 (ARMV4)'
#
if(wince)
	project(${PACKAGE}-wince)
	# don't forget parameters
	set(wince 1 CACHE TYPE STRING FORCE)
	set(wcelibcex ${wcelibcex} CACHE TYPE STRING FORCE)
	
	include_directories(${wcelibcex}/include/wcelibcex ${wcelibcex}/src)
	
	add_definitions(
# see config.h.cmake
#			-DDBUS_WINCE
			-DWINCE
			-DWIN32_PLATFORM_PSPC
			-D_WINDOWS
			-D_UNICODE
			-DUNICODE
			-DPOCKETPC2003_UI_MODEL
			)
			
	# Windows CE Version
	add_definitions(
			-D_WIN32_WCE=0x420
			-DWIN32_PLATFORM_PSPC=0x420
			-DUNDER_CE=0x420
			)
	
	# Architecture
	add_definitions(
			-DARM
			-D_ARM_
			)
	
	set(CMAKE_CXX_STANDARD_LIBRARIES "coredll.lib corelibc.lib ole32.lib oleaut32.lib uuid.lib commctrl.lib ws2.lib")
			
	set(CMAKE_SHARED_LINKER_FLAGS "/subsystem:windowsce,4.20 /machine:THUMB")

endif(wince)
#########################################################################


ENABLE_TESTING()

#########################################################################
# Disallow in-source build
#macro_ensure_out_of_source_build("dbus requires an out of source build. Please create a separate build directory and run 'cmake path_to_dbus [options]' there.")

# ... and warn in case of an earlier in-source build
#set(generatedFileInSourceDir EXISTS ${dbus_SOURCE_DIR}/config.h)
#if(${generatedFileInSourceDir})
#   message(STATUS "config.h exists in your source directory.")
#endif(${generatedFileInSourceDir})
#########################################################################

if (WIN32)
	set (LIBRARY_OUTPUT_PATH  ${CMAKE_BINARY_DIR}/bin)
else (WIN32)
	set (LIBRARY_OUTPUT_PATH  ${CMAKE_BINARY_DIR}/lib)
endif (WIN32)

set (EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)

# for including config.h and for includes like <dir/foo.h>
include_directories( ${CMAKE_SOURCE_DIR} ${CMAKE_BINARY_DIR} )
include_directories( ${CMAKE_INCLUDE_PATH} )

# linker search directories
link_directories(${DBUS_LIB_DIR} ${LIBRARY_OUTPUT_PATH} )
include_directories( ${CMAKE_LIBRARY_PATH}  )

set(DBUS_INCLUDES)

ENABLE_TESTING()


########### basic vars ###############

if (DBUSDIR)
	set(DBUS_INSTALL_DIR "${DBUSDIR}" CACHE TYPE STRING)
else (DBUSDIR)
	set(DBUS_INSTALL_DIR "$ENV{DBUSDIR}" CACHE TYPE STRING)
endif (DBUSDIR)

if (NOT DBUS_INSTALL_DIR)
	set(DBUS_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}" CACHE TYPE STRING)
endif (NOT DBUS_INSTALL_DIR)
	
if (DBUS_INSTALL_SYSTEM_LIBS)
	set(prefix                   ${DBUS_INSTALL_DIR})
	set(exec_prefix              ${prefix})
	set(EXPANDED_LIBDIR          ${DBUS_INSTALL_DIR}/lib)
	set(EXPANDED_INCLUDEDIR      ${DBUS_INSTALL_DIR}/include)
	set(EXPANDED_BINDIR          ${DBUS_INSTALL_DIR}/bin)
	set(EXPANDED_SYSCONFDIR      ${DBUS_INSTALL_DIR}/etc)
	set(EXPANDED_DATADIR         ${DBUS_INSTALL_DIR}/data)
	set(DBUS_BINDIR              ${EXPANDED_BINDIR})
	set(DBUS_MACHINE_UUID_FILE   ${DBUS_INSTALL_DIR}/lib/dbus/machine-id)
else (DBUS_INSTALL_SYSTEM_LIBS)
	set(EXPANDED_INCLUDEDIR      ${CMAKE_SOURCE_DIR}/include)
	set(EXPANDED_DATADIR         ${CMAKE_BINARY_DIR}/test/data)
	if (MSVC_IDE)
		set(EXPANDED_BINDIR      ${CMAKE_BINARY_DIR}/bin/debug)
	else (MSVC_IDE)
		set(EXPANDED_BINDIR      ${CMAKE_BINARY_DIR}/bin)	
	endif (MSVC_IDE)
	set(DBUS_BINDIR              ${EXPANDED_BINDIR})
	set(DBUS_MACHINE_UUID_FILE   ${CMAKE_BINARY_DIR}/lib/dbus/machine-id)
endif (DBUS_INSTALL_SYSTEM_LIBS)

########### command line options ###############
# TODO: take check from configure.in 

#AC_ARG_ENABLE(tests, AS_HELP_STRING([--enable-tests],[enable unit test code]),enable_tests=$enableval,enable_tests=$USE_MAINTAINER_MODE)
OPTION(DBUS_BUILD_TESTS "enable unit test code" ON)
if (DBUS_BUILD_TESTS)	
	if(NOT MSVC AND NOT CMAKE_BUILD_TYPE MATCHES Release)
		add_definitions(-g)
	endif(NOT MSVC AND NOT CMAKE_BUILD_TYPE MATCHES Release)
endif (DBUS_BUILD_TESTS)	

# win32 dbus service support - this support is not complete
OPTION(DBUS_SERVICE "enable dbus service installer" OFF)

#AC_ARG_ENABLE(ansi, AS_HELP_STRING([--enable-ansi],[enable -ansi -pedantic gcc flags]),enable_ansi=$enableval,enable_ansi=no)
OPTION(DBUS_ENABLE_ANSI "enable -ansi -pedantic gcc flags" OFF)
if(DBUS_ENABLE_ANSI)
   if(NOT MSVC)
        add_definitions(-ansi -D_POSIX_C_SOURCE=199309L -D_BSD_SOURCE -pedantic)
   else(NOT MSVC)
        add_definitions(-Za -D_POSIX_C_SOURCE=199309L -D_BSD_SOURCE -W4)
   endif(NOT MSVC) 
endif(DBUS_ENABLE_ANSI)

#AC_ARG_ENABLE(verbose-mode, AS_HELP_STRING([--enable-verbose-mode],[support verbose debug mode]),enable_verbose_mode=$enableval,enable_verbose_mode=$USE_MAINTAINER_MODE)
OPTION(DBUS_ENABLE_VERBOSE_MODE "support verbose debug mode" ON)

#AC_ARG_ENABLE(asserts, AS_HELP_STRING([--enable-asserts],[include assertion checks]),enable_asserts=$enableval,enable_asserts=$USE_MAINTAINER_MODE)
OPTION(DBUS_DISABLE_ASSERTS "Disable assertion checking" OFF)

#AC_ARG_ENABLE(checks, AS_HELP_STRING([--enable-checks],[include sanity checks on public API]),enable_checks=$enableval,enable_checks=yes)
OPTION(DBUS_DISABLE_CHECKS "Disable public API sanity checking" OFF)

#AC_ARG_ENABLE(xml-docs, AS_HELP_STRING([--enable-xml-docs],[build XML documentation (requires xmlto)]),enable_xml_docs=$enableval,enable_xml_docs=auto)
#xmldocs missing

#AC_ARG_ENABLE(doxygen-docs, AS_HELP_STRING([--enable-doxygen-docs],[build DOXYGEN documentation (requires Doxygen)]),enable_doxygen_docs=$enableval,enable_doxygen_docs=auto)
if(DOXYGEN)
  OPTION(DBUS_ENABLE_DOXYGEN_DOCS "build DOXYGEN documentation (requires Doxygen)" ON)
endif(DOXYGEN)

#AC_ARG_ENABLE(gcov, AS_HELP_STRING([--enable-gcov],[compile with coverage profiling instrumentation (gcc only)]),enable_gcov=$enableval,enable_gcov=no)
OPTION(DBUS_GCOV_ENABLED "compile with coverage profiling instrumentation (gcc only)" OFF)
if(DBUS_GCOV_ENABLED)
   if(NOT MSVC)
        add_definitions(-fprofile-arcs -ftest-coverage)
        # FIXME!!!!
        ## remove optimization
#        CFLAGS=`echo "$CFLAGS" | sed -e 's/-O[0-9]*//g'`
   endif(NOT MSVC) 
endif(DBUS_GCOV_ENABLED)

#AC_ARG_ENABLE(abstract-sockets, AS_HELP_STRING([--enable-abstract-sockets],[use abstract socket namespace (linux only)]),enable_abstract_sockets=$enableval,enable_abstract_sockets=auto)
#abstract sockets missing

#AC_ARG_ENABLE(selinux, AS_HELP_STRING([--enable-selinux],[build with SELinux support]),enable_selinux=$enableval,enable_selinux=auto)
#selinux missing

#AC_ARG_ENABLE(dnotify, AS_HELP_STRING([--enable-dnotify],[build with dnotify support (linux only)]),enable_dnotify=$enableval,enable_dnotify=auto)
if("${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")
    OPTION(DBUS_BUS_ENABLE_DNOTIFY_ON_LINUX "build with dnotify support (linux only)" ON) # add a check !
endif("${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")

#AC_ARG_ENABLE(kqueue, AS_HELP_STRING([--enable-kqueue],[build with kqueue support (FreeBSD only)]),enable_kqueue=$enableval,enable_kqueue=auto)
#missing

#AC_ARG_ENABLE(console-owner-file, AS_HELP_STRING([--enable-console-owner-file],[enable console owner file]),enable_console_owner_file=$enableval,enable_console_owner_file=auto)
STRING(TOUPPER ${CMAKE_SYSTEM_NAME} sysname)
if("${sysname}" MATCHES ".*SOLARIS.*")
    OPTION(HAVE_CONSOLE_OWNER_FILE "enable console owner file (solaris only)" ON)
    if(HAVE_CONSOLE_OWNER_FILE)
        SET(DBUS_CONSOLE_OWNER_FILE "/dev/console" CACHE STRING "Directory to check for console ownerhip")
    endif(HAVE_CONSOLE_OWNER_FILE)
endif("${sysname}" MATCHES ".*SOLARIS.*")

#AC_ARG_WITH(xml, AS_HELP_STRING([--with-xml=[libxml/expat]],[XML library to use]))
if(NOT LIBXML2_FOUND AND NOT LIBEXPAT_FOUND)
    message(FATAL "Neither expat nor libxml2 found!")
endif(NOT LIBXML2_FOUND AND NOT LIBEXPAT_FOUND)

if(LIBEXPAT_FOUND)
    OPTION(DBUS_USE_EXPAT "Use expat (== ON) or libxml2 (==OFF)" ON)
else(LIBEXPAT_FOUND)
    OPTION(DBUS_USE_EXPAT "Use expat (== ON) or libxml2 (==OFF)" OFF)
endif(LIBEXPAT_FOUND)

if(DBUS_USE_EXPAT)
    SET(XML_LIB "Expat")
    SET(XML_LIBRARY     ${LIBEXPAT_LIBRARIES})
    SET(XML_INCLUDE_DIR ${LIBEXPAT_INCLUDE_DIR})
else(DBUS_USE_EXPAT)
    SET(XML_LIB "LibXML2")
    SET(XML_LIBRARY     ${LIBXML2_LIBRARIES})
    SET(XML_INCLUDE_DIR ${LIBXML2_INCLUDE_DIR})
endif(DBUS_USE_EXPAT)


#AC_ARG_WITH(init-scripts, AS_HELP_STRING([--with-init-scripts=[redhat]],[Style of init scripts to install]))
#AC_ARG_WITH(session-socket-dir, AS_HELP_STRING([--with-session-socket-dir=[dirname]],[Where to put sockets for the per-login-session message bus]))
#AC_ARG_WITH(test-socket-dir, AS_HELP_STRING([--with-test-socket-dir=[dirname]],[Where to put sockets for make check]))
#AC_ARG_WITH(system-pid-file, AS_HELP_STRING([--with-system-pid-file=[pidfile]],[PID file for systemwide daemon]))
#AC_ARG_WITH(system-socket, AS_HELP_STRING([--with-system-socket=[filename]],[UNIX domain socket for systemwide daemon]))
#AC_ARG_WITH(console-auth-dir, AS_HELP_STRING([--with-console-auth-dir=[dirname]],[directory to check for console ownerhip]))
#AC_ARG_WITH(console-owner-file, AS_HELP_STRING([--with-console-owner-file=[filename]],[file whose owner determines current console owner]))
#AC_ARG_WITH(dbus_user, AS_HELP_STRING([--with-dbus-user=<user>],[User for running the DBUS daemon (messagebus)]))
#AC_ARG_WITH(dbus_daemondir, AS_HELP_STRING([--with-dbus-daemondir=[dirname]],[Directory for installing the DBUS daemon]))
# all missing or hardcoded for now

# 'hidden' ones
set(atomic_int OFF)
set(atomic_int486 OFF)
if(CMAKE_COMPILER_IS_GNUCC AND NOT DBUS_ENABLE_ANSI)
    FIND_PROGRAM(UNAME_EXECUTABLE
                    NAMES uname
                    PATHS /bin /usr/bin /usr/local/bin c:/Programme/MSys/bin d:/Programme/MSys/bin)

    if(UNAME_EXECUTABLE)
        EXECUTE_PROCESS(COMMAND ${UNAME_EXECUTABLE} "-m"
                        OUTPUT_VARIABLE UNAME_OUTPUT)

        if("UNAME_OUTPUT" MATCHES "^.*i[0123]86.*$")
            set(atomic_int ON)
        else("UNAME_OUTPUT" MATCHES "^.*i[0123]86.*$")
            if("UNAME_OUTPUT" MATCHES "^.*i?86.*$")
                set(atomic_int ON)
                set(atomic_int_486 ON)
            endif("UNAME_OUTPUT" MATCHES "^.*i?86.*$")
        endif("UNAME_OUTPUT" MATCHES "^.*i[0123]86.*$")
    endif(UNAME_EXECUTABLE)
endif(CMAKE_COMPILER_IS_GNUCC AND NOT DBUS_ENABLE_ANSI)

OPTION(DBUS_HAVE_ATOMIC_INT    "Some atomic integer implementation present" ${atomic_int})
OPTION(DBUS_USE_ATOMIC_INT_486 "Use atomic integer implementation for 486" ${atomic_int_486})

if(X11_FOUND)
  OPTION(DBUS_BUILD_X11 "Build X11-dependent code " ON)
endif(X11_FOUND)

# test binary names
if (WIN32)
	set (EXT ".exe")
endif(WIN32)

# confirure.in:
#TEST_PATH(SERVICE_DIR,          data/valid-service-files)
#TEST_PATH(SERVICE_BINARY,       test-service)
#TEST_PATH(SHELL_SERVICE_BINARY, test-shell-service)
#TEST_PATH(EXIT_BINARY,          test-exit)
#TEST_PATH(SEGFAULT_BINARY,      test-segfault)
#TEST_PATH(SLEEP_FOREVER_BINARY, test-sleep-forever)

set(TEST_SERVICE_DIR          ${CMAKE_BINARY_DIR}/test/data/valid-service-files     CACHE STRING "Full path to test file test/data/valid-service-files in builddir")
set(TEST_SERVICE_BINARY       ${EXPANDED_BINDIR}/test-service${EXT}       CACHE STRING "Full path to test file test/test-service in builddir")
set(TEST_SHELL_SERVICE_BINARY ${EXPANDED_BINDIR}/test-shell-service${EXT} CACHE STRING "Full path to test file test/test-shell-service in builddir")    
set(TEST_EXIT_BINARY          ${EXPANDED_BINDIR}/test-exit${EXT}          CACHE STRING "Full path to test file test/test-exit in builddir")
set(TEST_SEGFAULT_BINARY      ${EXPANDED_BINDIR}/test-segfault${EXT}      CACHE STRING "Full path to test file test/test-segfault in builddir")
set(TEST_SLEEP_FOREVER_BINARY ${EXPANDED_BINDIR}/test-sleep-forever${EXT} CACHE STRING "Full path to test file test/test-sleep-forever in builddir")

#### Find socket directories
if (NOT WIN32)
    if (NOT $ENV{TMPDIR} STREQUAL "")
        set (DBUS_SESSION_SOCKET_DIR $ENV{TMPDIR})
    else (NOT $ENV{TMPDIR} STREQUAL "")
        if (NOT $ENV{TEMP} STREQUAL "")
            set (DBUS_SESSION_SOCKET_DIR $ENV{TEMP})
        else (NOT $ENV{TEMP} STREQUAL "")
            if (NOT $ENV{TMP} STREQUAL "")
                set (DBUS_SESSION_SOCKET_DIR $ENV{TMP})
            else (NOT $ENV{TMP} STREQUAL "")
                set (DBUS_SESSION_SOCKET_DIR /tmp)
            endif (NOT $ENV{TMP} STREQUAL "")
        endif (NOT $ENV{TEMP} STREQUAL "")
    endif (NOT $ENV{TMPDIR} STREQUAL "")
endif (NOT WIN32)

#AC_ARG_WITH(test-socket-dir, AS_HELP_STRING([--with-test-socket-dir=[dirname]],[Where to put sockets for make check]))

#AC_ARG_WITH(system-pid-file, AS_HELP_STRING([--with-system-pid-file=[pidfile]],[PID file for systemwide daemon]))

#if ! test -z "$with_system_pid_file"; then
#   DBUS_SYSTEM_PID_FILE=$with_system_pid_file
#elif test x$operating_system = xredhat ; then
#   DBUS_SYSTEM_PID_FILE=${EXPANDED_LOCALSTATEDIR}/run/messagebus.pid
#else
#   DBUS_SYSTEM_PID_FILE=${EXPANDED_LOCALSTATEDIR}/run/dbus/pid
#fi
# TODO: fix redhet 
if (WIN32)
  # bus-test expects a non empty string
	set (DBUS_SYSTEM_PID_FILE "/dbus-pid")
else (WIN32)
	set (DBUS_SYSTEM_PID_FILE ${EXPANDED_LOCALSTATEDIR}/run/dbus/pid)
endif (WIN32)

#AC_ARG_WITH(system-socket, AS_HELP_STRING([--with-system-socket=[filename]],[UNIX domain socket for systemwide daemon]))

#AC_ARG_WITH(console-auth-dir, AS_HELP_STRING([--with-console-auth-dir=[dirname]],[directory to check for console ownerhip]))

if (WIN32)
	set (DBUS_CONSOLE_AUTH_DIR "")
else (WIN32)
	set (DBUS_CONSOLE_AUTH_DIR "/var/run/console/")
endif (WIN32)

#AC_ARG_WITH(dbus_user, AS_HELP_STRING([--with-dbus-user=<user>],[User for running the DBUS daemon (messagebus)]))

set (DBUS_USER )


if (WIN32)
  set (DBUS_SESSION_BUS_DEFAULT_ADDRESS "tcp:host=localhost,port=12434")
  set (DBUS_SYSTEM_BUS_DEFAULT_ADDRESS "tcp:host=localhost,port=12434")
  set (DBUS_SYSTEM_CONFIG_FILE "etc/system.conf")
  set (DBUS_SESSION_CONFIG_FILE "etc/session.conf")
  # bus-test expects a non empty string
  set (DBUS_USER "Administrator") 
  set (DBUS_DATADIR "data")
else (WIN32)
  set (DBUS_SYSTEM_BUS_DEFAULT_ADDRESS unix:tmpdir=)
  set (DBUS_SESSION_BUS_DEFAULT_ADDRESS unix:path=${DBUS_SESSION_SOCKET_DIR})
  set (sysconfdir "")
  set (configdir ${sysconfdir}/dbus-1 )
  set (DBUS_SYSTEM_CONFIG_FILE  ${configdir}/system.conf)
  set (DBUS_SESSION_CONFIG_FILE ${configdir}/session.conf)
  set (DBUS_USER "root") 
  set (DBUS_DATADIR ${EXPANDED_DATADIR})
endif (WIN32)

set (DAEMON_NAME dbus-daemon)  

########### create config.h ###############

#include(ConfigureChecks.cmake)

# better use flags for gcc 
if (MINGW)
	set (HAVE_GNUC_VARARGS 1)
endif(MINGW)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h )
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/dbus-env.bat.cmake ${CMAKE_BINARY_DIR}/bin/dbus-env.bat )
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/dbus-launch.bat.cmake ${CMAKE_BINARY_DIR}/bin/dbus-launch.bat )
install_files(/bin FILES ${CMAKE_BINARY_DIR}/bin/dbus-env.bat)	
install_files(/bin FILES ${CMAKE_BINARY_DIR}/bin/dbus-launch.bat)	

# compiler definitions 
add_definitions(-DHAVE_CONFIG_H=1)
add_definitions(${DBUS_BUS_CFLAGS} -DDBUS_API_SUBJECT_TO_CHANGE)


########### subdirs ###############

add_subdirectory( dbus )
add_subdirectory( bus )
if (DBUS_BUILD_TESTS)
	add_subdirectory( test )
endif (DBUS_BUILD_TESTS)
add_subdirectory( tools )


OPTION(DBUS_INSTALL_SYSTEM_LIBS "install required system libraries" OFF)
MESSAGE(" ")
MESSAGE("set -DDBUS_INSTALL_SYSTEM_LIBS=1 to install runtime libraries too")
MESSAGE("set DBUSDIR (environment or cmake option) to overwrite the default install directory ")
MESSAGE(" ")
MESSAGE(" ")
GET_FILENAME_COMPONENT(C_COMPILER ${CMAKE_C_COMPILER} NAME)
GET_FILENAME_COMPONENT(CXX_COMPILER ${CMAKE_CXX_COMPILER} NAME)

message("                  D-BUS ${VERSION}                                    ")
message("                  ==========                                          ")
message("                                                                      ")
message("        prefix:                   ${prefix}                           ")
message("        exec_prefix:              ${exec_prefix}                      ")
message("        libdir:                   ${EXPANDED_LIBDIR}                  ")
message("        bindir:                   ${EXPANDED_BINDIR}                  ")
message("        sysconfdir:               ${EXPANDED_SYSCONFDIR}              ")
message("        localstatedir:            ${EXPANDED_LOCALSTATEDIR}           ")
message("        datadir:                  ${EXPANDED_DATADIR}                 ")
message("        source code location:     ${DBUS_SOURCE_DIR}                  ")
message("        c compiler:               ${C_COMPILER}                       ")
message("        cflags:                   ${CMAKE_C_FLAGS}                    ")
message("        cflags debug:             ${CMAKE_C_FLAGS_DEBUG}              ")
message("        cflags release:           ${CMAKE_C_FLAGS_RELEASE}            ")
message("        cxx compiler:             ${CXX_COMPILER}                     ")
message("        cxxflags:                 ${CMAKE_CXX_FLAGS}                  ")
message("        cxxflags debug:           ${CMAKE_CXX_FLAGS_DEBUG}            ")
message("        cxxflags release:         ${CMAKE_CXX_FLAGS_RELEASE}          ")
message("        64-bit int:               ${DBUS_INT64_TYPE}                  ")
message("        32-bit int:               ${DBUS_INT32_TYPE}                  ")
message("        16-bit int:               ${DBUS_INT16_TYPE}                  ")
message("        Doxygen:                  ${DOXYGEN}                          ")
message("        xmlto:                    ${XMLTO}                            ")


#message("        Maintainer mode:          ${USE_MAINTAINER_MODE}              ")
message("        gcc coverage profiling:   ${DBUS_GCOV_ENABLED}                ")
message("        Building unit tests:      ${DBUS_BUILD_TESTS}                 ")
message("        Building verbose mode:    ${DBUS_ENABLE_VERBOSE_MODE}         ")
message("        Building w/o assertions:  ${DBUS_DISABLE_ASSERTS}             ")
message("        Building w/o checks:      ${DBUS_DISABLE_CHECKS}              ")
message("        installing system libs:   ${DBUS_INSTALL_SYSTEM_LIBS}         ")
#message("        Building SELinux support: ${have_selinux}                     ")
#message("        Building dnotify support: ${have_dnotify}                     ")
message("        Building Doxygen docs:    ${DBUS_ENABLE_DOXYGEN_DOCS}         ")
#message("        Building XML docs:        ${enable_xml_docs}                  ")
#message("        Gettext libs (empty OK):  ${INTLLIBS}                         ")
message("        Using XML parser:         ${XML_LIB}                          ")
if (WIN32)
message("        System bus address:       ${DBUS_SYSTEM_BUS_DEFAULT_ADDRESS}  ")
message("        Session bus address:      ${DBUS_SESSION_BUS_DEFAULT_ADDRESS} ")
else (WIN32)
#message("        Init scripts style:       ${with_init_scripts}                ")
#message("        Abstract socket names:    ${have_abstract_sockets}            ")
message("        System bus socket:        ${DBUS_SYSTEM_SOCKET}               ")
message("        System bus address:       ${DBUS_SYSTEM_BUS_DEFAULT_ADDRESS}  ")
message("        System bus PID file:      ${DBUS_SYSTEM_PID_FILE}             ")
message("        Session bus socket dir:   ${DBUS_SESSION_SOCKET_DIR}          ")
message("        Console auth dir:         ${DBUS_CONSOLE_AUTH_DIR}            ")
message("        System bus user:          ${DBUS_USER}                        ")
message("        'make check' socket dir:  ${TEST_SOCKET_DIR}                  ")
endif (WIN32)
MESSAGE(" ")
if (DBUS_BUILD_TESTS)
    message("NOTE: building with unit tests increases the size of the installed library and renders it insecure.")
endif(DBUS_BUILD_TESTS)

if (DBUS_BUILD_TESTS AND DBUS_DISABLE_ASSERTS)
    message("NOTE: building with unit tests but without assertions means tests may not properly report failures (this configuration is only useful when doing something like profiling the tests)")
endif(DBUS_BUILD_TESTS AND DBUS_DISABLE_ASSERTS)

if (DBUS_GCOV_ENABLED)
    message("NOTE: building with coverage profiling is definitely for developers only.")
endif(DBUS_GCOV_ENABLED)

if (DBUS_ENABLE_VERBOSE_MODE)
    message("NOTE: building with verbose mode increases library size, may slightly increase security risk, and decreases performance.")
endif(DBUS_ENABLE_VERBOSE_MODE)

if(NOT DBUS_DISABLE_ASSERTS)
    message("NOTE: building with assertions increases library size and decreases performance.")
endif(NOT DBUS_DISABLE_ASSERTS)

if (DBUS_DISABLE_CHECKS)
    message("NOTE: building without checks for arguments passed to public API makes it harder to debug apps using D-BUS, but will slightly decrease D-BUS library size and _very_ slightly improve performance.")
endif(DBUS_DISABLE_CHECKS)
MESSAGE(" ")


INCLUDE(modules/CPackInstallConfig.cmake)