cmake_minimum_required(VERSION 2.8.11)
project(gsm)
include(CheckIncludeFile)
include(CheckCXXCompilerFlag)
include(CheckCCompilerFlag)
function(enable_cxx_compiler_flag_if_supported flag)
    string(FIND "${CMAKE_CXX_FLAGS}" "${flag}" flag_already_set)
    if(flag_already_set EQUAL -1)
        check_cxx_compiler_flag("${flag}" flag_supported)
        if(flag_supported)
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${flag}" PARENT_SCOPE)
        endif()
        unset(flag_supported CACHE)
    endif()
endfunction()

function(enable_c_compiler_flag_if_supported flag)
    string(FIND "${CMAKE_C_FLAGS}" "${flag}" flag_already_set)
    if(flag_already_set EQUAL -1)
        check_cxx_compiler_flag("${flag}" flag_supported)
        if(flag_supported)
            set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${flag}" PARENT_SCOPE)
        endif()
        unset(flag_supported CACHE)
    endif()
endfunction()

option(SASR "Define SASR if >> is a signed arithmetic shift (-1 >> 1 == -1)" ON)
option(FLOAT_MUL "Define this if your host multiplies floats faster than integers, e.g. on a SPARCstation." OFF)
option(FAST "Define together with USE_FLOAT_MUL to enable the GSM library's approximation option for incorrect, but good-enough results" OFF)
option(LTP_CUT "Define to enable the GSM library's long-term correlation approximation option---faster, but worse; works for both integer and floating point multiplications. This flag is still in the experimental stage." OFF)
option(WAV49 "Define to enable the GSM library's option to pack GSM frames in the style used by the WAV #49 format.  If you want to write a tool that produces .WAV files which contain GSM-encoded data, define this, and read about the GSM_OPT_WAV49 option in the manual page on gsm_option(3)." OFF)
option(NeedFunctionPrototypes "to compile without, function prototypes in the header files." OFF)
option(STUPID_COMPILER "to circumvent some compilers static limits regarding the number of subexpressions in a statement." OFF)
option(DEBUG "To enable/disable assertions" OFF)

set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/dist)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

set(CMAKE_INSTALL_LIBDIR ${CMAKE_INSTALL_LIBDIR}/lib)
set(CMAKE_INSTALL_MANDIR ${CMAKE_INSTALL_MANDIR}/man/man3)
set(CMAKE_INSTALL_INCLUDEDIR ${CMAKE_INSTALL_INCLUDEDIR}/inc)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS})
set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS})

enable_cxx_compiler_flag_if_supported("-pedantic")
enable_cxx_compiler_flag_if_supported("-Wall")
enable_cxx_compiler_flag_if_supported("-O2")
enable_cxx_compiler_flag_if_supported("-c")
enable_cxx_compiler_flag_if_supported("-ansi")
enable_cxx_compiler_flag_if_supported("-Wno-comment")

if (USE_SASR)
    set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} -DSASR)
endif (USE_SASR)

if (USE_DEBUG)
    set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} -DNDEBUG)
endif (USE_DEBUG)

if (USE_FLOAT_MUL)
    set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} -DUSE_FLOAT_MUL)
endif (USE_FLOAT_MUL)

if (USE_FAST)
    set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} -DFAST)
endif (USE_FAST)

if (USE_LTP_CUT)
    set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} -DLTP_CUT)
endif (USE_LTP_CUT)

if (USE_WAV49)
    set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} -DWAV49)
endif (USE_WAV49)

if (USE_NeedFunctionPrototypes)
    set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} -DNeedFunctionPrototypes=1)
endif (USE_NeedFunctionPrototypes)

if (USE_STUPID_COMPILER)
    set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} -DSTUPID_COMPILER)
endif (USE_STUPID_COMPILER)


set(ROOT .)
set(ADDTST ${ROOT}/add-test)
set(TST ${ROOT}/tst)
set(MAN ${ROOT}/man)
set(BIN ${ROOT}/bin)
set(SRC ${ROOT}/src)
set(LIB ${ROOT}/lib)
set(TLS ${ROOT}/tls)
set(INC ${ROOT}/inc)

set(GSM_HEADERS ${BIN}/gsm.h)

set(HEADERS
        ${INC}/proto.h
        ${INC}/unproto.h
        ${INC}/config.h
        ${INC}/private.h
        ${INC}/gsm.h
        ${INC}/toast.h
        ${TLS}/taste.h
        )

set(GSM_SOURCES
        ${SRC}/add.c
        ${SRC}/code.c
        ${SRC}/debug.c
        ${SRC}/decode.c
        ${SRC}/long_term.c
        ${SRC}/lpc.c
        ${SRC}/preprocess.c
        ${SRC}/rpe.c
        ${SRC}/gsm_destroy.c
        ${SRC}/gsm_decode.c
        ${SRC}/gsm_encode.c
        ${SRC}/gsm_explode.c
        ${SRC}/gsm_implode.c
        ${SRC}/gsm_create.c
        ${SRC}/gsm_print.c
        ${SRC}/gsm_option.c
        ${SRC}/short_term.c
        ${SRC}/table.c
        )

set(TOAST_SOURCES
        ${SRC}/toast.c
        ${SRC}/toast_lin.c
        ${SRC}/toast_ulaw.c
        ${SRC}/toast_alaw.c
        ${SRC}/toast_audio.c
        )

set(SOURCES
        ${GSM_SOURCES}
        ${TOAST_SOURCES}
        ${ADDTST}/add_test.c
        ${TLS}/sour.c
        ${TLS}/ginger.c
        ${TLS}/sour1.dta
        ${TLS}/sour2.dta
        ${TLS}/bitter.c
        ${TLS}/bitter.dta
        ${TLS}/taste.c
        ${TLS}/sweet.c
        ${TST}/cod2lin.c
        ${TST}/cod2txt.c
        ${TST}/gsm2cod.c
        ${TST}/lin2cod.c
        ${TST}/lin2txt.c
        )

set(GSM_LIB ${GSM_LIB} gcc)
add_library(GSM_OBJECTS OBJECT
        ${HEADERS}
        ${GSM_SOURCES}
        )
set_property(TARGET GSM_OBJECTS PROPERTY POSITION_INDEPENDENT_CODE 1)

add_library(gsm STATIC $<TARGET_OBJECTS:GSM_OBJECTS>)
target_link_libraries(gsm ${GSM_LIB})

add_library(TOAST_OBJECTS OBJECT
        ${TOAST_SOURCES}
        )

add_library(OBJECTS OBJECT
        $<TARGET_OBJECTS:GSM_OBJECTS>
        $<TARGET_OBJECTS:TOAST_OBJECTS>
        )

# GSM_MANUALS =	$(MAN)/gsm.3		\
# $(MAN)/gsm_explode.3	\
# $(MAN)/gsm_option.3	\
# $(MAN)/gsm_print.3

add_executable(toast $<TARGET_OBJECTS:TOAST_OBJECTS>)
target_link_libraries(toast gsm)

add_executable(untoast $<TARGET_OBJECTS:TOAST_OBJECTS>)
target_link_libraries(untoast gsm)

add_executable(tcat $<TARGET_OBJECTS:GSM_OBJECTS> $<TARGET_OBJECTS:TOAST_OBJECTS>)
target_link_libraries(tcat gsm)
