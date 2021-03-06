project(prodatum)
cmake_minimum_required(VERSION 2.6)

SET( SYSEX_MAX_SIZE 1024 )
SET( RINGBUFFER_WRITE 2048 )
SET( RINGBUFFER_READ 2048 )
SET( PATH_MAX 1024 )
SET( LOG_BUFFER_SIZE 1048576 )
CONFIGURE_FILE( ${CMAKE_SOURCE_DIR}/config.h.in ${CMAKE_SOURCE_DIR}/config.h )

include_directories (${prodatum_SOURCE_DIR}/include)
link_directories (${prodatum_SOURCE_DIR}/lib)

set(RES_FILES "")

if(MINGW)
  set(RES_FILES "windows.rc")
  set(CMAKE_RC_COMPILER_INIT windres)
  ENABLE_LANGUAGE(RC)
  SET(CMAKE_RC_COMPILE_OBJECT
    "<CMAKE_RC_COMPILER> -O coff <DEFINES> -i <SOURCE> -o <OBJECT>")
endif(MINGW)

file (GLOB prodatum_SOURCES *.C)
add_executable (prodatum ${prodatum_SOURCES} ${RES_FILES})

if(WIN32)
  ADD_DEFINITIONS(-DWIN32 -DSYNCLOG -Wall -W -Wno-char-subscripts -Wno-write-strings -Wno-format)
  set(CMAKE_EXE_LINKER_FLAGS "-mwindows -m32 -static-libgcc -static-libstdc++")
  set(ADDITIONAL_LIBRARIES winmm ole32 uuid comctl32 wsock32)
endif(WIN32)

if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  ADD_DEFINITIONS(-DUSE_MLOCK -DOSX -DSYNCLOG -Wall -W -Wno-char-subscripts -Wno-write-strings)
  set(ADDITIONAL_LIBRARIES "-framework Carbon -framework ApplicationServices -framework CoreMIDI -framework CoreAudio")
endif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

IF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
	ADD_DEFINITIONS(-DUSE_MLOCK -D__linux -DSYNCLOG -Wall -W -Wno-char-subscripts -Wno-write-strings)
	set(CMAKE_EXE_LINKER_FLAGS "-static-libgcc -static-libstdc++")
  set(ADDITIONAL_LIBRARIES X11 Xinerama Xft fontconfig asound pthread)
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

target_link_libraries (prodatum fltk.a portmidi_s.a ${ADDITIONAL_LIBRARIES} )
