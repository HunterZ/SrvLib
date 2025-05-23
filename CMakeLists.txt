cmake_minimum_required(VERSION 3.10)

project(SrvLib)

ADD_DEFINITIONS(-DUNICODE)
ADD_DEFINITIONS(-D_UNICODE)

# specify the C++ standard
set(SUPPORTED_CXX_STANDARDS 14 17)
if(NOT DEFINED CMAKE_CXX_STANDARD)
    message(STATUS "Setting C++ version to '14' as none was specified.")
    set(CMAKE_CXX_STANDARD 14)
    if (WIN32)
	    # Turn off Microsofts "security" warnings.
        add_compile_definitions(_CRT_SECURE_NO_WARNINGS NDEBUG _CONSOLE _SCL_SECURE_NO_WARNINGS NOGDICAPMASKS NOVIRTUALKEYCODES NOWINMESSAGES NOWINSTYLES NOSYSMETRICS NOMENUS NOICONS NOKEYSTATES NOSYSCOMMANDS NORASTEROPS NOSHOWWINDOW OEMRESOURCE NOATOM NOCLIPBOARD NOCOLOR NOCTLMGR NODRAWTEXT NOGDI NOKERNEL NOUSER NONLS NOMB NOMEMMGR NOMETAFILE NOMINMAX NOMSG NOOPENFILE NOSCROLL NOSOUND NOTEXTMETRIC NOWH NOWINOFFSETS NOCOMM NOKANJI NOHELP NOPROFILER NODEFERWINDOWPOS NOMCX)
        set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded")
	    add_compile_options("/EHa")
    else()
        string(FIND "${CMAKE_CXX_FLAGS_DEBUG}" "-O0" nPos)
        if(${nPos} EQUAL "-1")
            SET(CMAKE_CXX_FLAGS_DEBUG "-O0 -g3 -ggdb -D_DEBUG -DDEBUG")
    	    add_compile_options("-Wall" "-Wpedantic" "-Wextra" "-fexceptions")
        else()
            message(STATUS "${PROJECT_NAME}: CMAKE_CXX_FLAGS_DEBUG hat bereits -O0 enthalten")
        endif()
    endif()
else()
    message(STATUS "${PROJECT_NAME}: CMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD} set by parent project.")
endif()
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(srvlib STATIC)

set(targetSrc "${CMAKE_CURRENT_SOURCE_DIR}/ServMain.cpp")
if(("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC") OR WIN32)
list(APPEND targetSrc
    "${CMAKE_CURRENT_SOURCE_DIR}/BaseSrv.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/BaseSrv.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/SrvCtrl.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/SrvCtrl.h"
)
endif()

target_sources(srvlib
    PRIVATE ${targetSrc}
    PUBLIC FILE_SET HEADERS
    BASE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}"
    FILES Service.h
)

if(PROJECT_IS_TOP_LEVEL)
    add_executable(ExampleSrv  ExampleSrv.cpp)
    target_link_libraries(ExampleSrv srvlib)
    if (NOT MSVC)
        target_link_libraries(ExampleSrv pthread)
    endif()

    file(READ init.d/examplesrv FILE_CONTENTS)
    string(REPLACE "~" ${CMAKE_CURRENT_BINARY_DIR} NEW_FILE_CONTENTS ${FILE_CONTENTS})
    file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/examplesrv ${NEW_FILE_CONTENTS})

    file(READ example.service FILE_CONTENTS)
    string(REPLACE "~" ${CMAKE_CURRENT_BINARY_DIR} NEW_FILE_CONTENTS ${FILE_CONTENTS})
    file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/example.service ${NEW_FILE_CONTENTS})

    install(TARGETS srvlib DESTINATION lib)
endif()
