cmake_minimum_required(VERSION 2.8.11)

project(liblzma)

include(CheckCCompilerFlag)
include(CheckCSourceCompiles)
include(CheckFunctionExists)
include(CheckIncludeFiles)
include(CheckLibraryExists)
include(CheckStructHasMember)
include(CheckSymbolExists)
include(CheckTypeSize)
include(CMakePackageConfigHelpers)
include(GNUInstallDirs)
include(TestBigEndian)

set(LIBLZMA_MAJOR_VERSION 5)
set(LIBLZMA_MINOR_VERSION 2)
set(LIBLZMA_PATCH_VERSION 3)
set(LIBLZMA_VERSION ${LIBLZMA_MAJOR_VERSION}.${LIBLZMA_MINOR_VERSION}.${LIBLZMA_PATCH_VERSION})

find_package(Threads)

if(Threads_FOUND AND NOT TARGET Threads::Threads)
	add_library(Threads::Threads INTERFACE IMPORTED)
	if(THREADS_HAVE_PTHREAD_ARG)
		set_property(TARGET Threads::Threads PROPERTY INTERFACE_COMPILE_OPTIONS "$<$<COMPILE_LANGUAGE:CUDA>:SHELL:-Xcompiler -pthread>" "$<$<NOT:$<COMPILE_LANGUAGE:CUDA>>:-pthread>")
	endif()
	if(CMAKE_THREAD_LIBS_INIT)
		set_property(TARGET Threads::Threads PROPERTY INTERFACE_LINK_LIBRARIES "${CMAKE_THREAD_LIBS_INIT}")
	endif()
endif()

option(BUILD_SHARED_LIBS "Build shared libraries" ON)

option(LIBLZMA_COND_ASM_X86 "Use assembler optimizations if such exist for the architecture." ON)
option(LIBLZMA_COND_CHECK_CRC32 "Enable crc32 integrity check." ON)
option(LIBLZMA_COND_CHECK_CRC64 "Enable crc64 integrity check." ON)
option(LIBLZMA_COND_CHECK_SHA256 "Enable sha256 integrity check." ON)
option(LIBLZMA_COND_DECODER_DELTA "Enable delta decoder." ON)
option(LIBLZMA_COND_DECODER_LZ "Enable lz decoder." ON)
option(LIBLZMA_COND_DECODER_LZMA1 "Enable lzma1 decoder." ON)
option(LIBLZMA_COND_DECODER_LZMA2 "Enable lzma2 decoder." ON)
option(LIBLZMA_COND_DECODER_SIMPLE "Enable simple decoder." ON)
option(LIBLZMA_COND_ENCODER_DELTA "Enable delta encoder." ON)
option(LIBLZMA_COND_ENCODER_LZ "Enable lz encoder." ON)
option(LIBLZMA_COND_ENCODER_LZMA1 "Enable lzma1 encoder." ON)
option(LIBLZMA_COND_ENCODER_LZMA2 "Enable lzma2 encoder." ON)
option(LIBLZMA_COND_ENCODER_SIMPLE "Enable simple encoder." ON)
option(LIBLZMA_COND_FILTER_ARM "Enable arm filter." ON)
option(LIBLZMA_COND_FILTER_ARMTHUMB "Enable armthumb filter." ON)
option(LIBLZMA_COND_FILTER_DELTA "Enable delta filter." ON)
option(LIBLZMA_COND_FILTER_IA64 "Enable ia64 filter." ON)
option(LIBLZMA_COND_FILTER_LZ "Enable lz filter." ON)
option(LIBLZMA_COND_FILTER_LZMA1 "Enable lzma1 filter." ON)
option(LIBLZMA_COND_FILTER_POWERPC "Enable powerpc filter." ON)
option(LIBLZMA_COND_FILTER_SIMPLE "Enable simple filter." ON)
option(LIBLZMA_COND_FILTER_SPARC "Enable sparc filter." ON)
option(LIBLZMA_COND_FILTER_X86 "Enable x86 filter." ON)
option(LIBLZMA_COND_INTERNAL_SHA256 "Use internal SHA-256 code instead of code from the operating system. See INSTALL for possible subtle problems." ON)
option(LIBLZMA_COND_LZMAINFO "Build lzmainfo (it exists primarily for LZMA Utils compatibility)." ON)
option(LIBLZMA_COND_MAIN_DECODER "Enable main decoder." ON)
option(LIBLZMA_COND_MAIN_ENCODER "Enable main encoder." ON)
option(LIBLZMA_COND_SMALL "Make liblzma smaller and a little slower. This is disabled by default to optimize for speed." OFF)
option(LIBLZMA_COND_THREADS "Disabling this together with COND_SMALL makes liblzma thread unsafe." ON)
option(LIBLZMA_COND_XZ "Build the xz tool." ON)
option(LIBLZMA_ENABLE_NLS "Use Native Language Support." ON)
option(LIBLZMA_HAVE_MF_BT2 "Enable bt2 match finder. At least one match finder is required for encoding with the LZMA1 and LZMA2 filters." ON)
option(LIBLZMA_HAVE_MF_BT3 "Enable bt3 match finder. At least one match finder is required for encoding with the LZMA1 and LZMA2 filters." ON)
option(LIBLZMA_HAVE_MF_BT4 "Enable bt4 match finder. At least one match finder is required for encoding with the LZMA1 and LZMA2 filters." ON)
option(LIBLZMA_HAVE_MF_HC3 "Enable hc3 match finder. At least one match finder is required for encoding with the LZMA1 and LZMA2 filters." ON)
option(LIBLZMA_HAVE_MF_HC4 "Enable hc4 match finder. At least one match finder is required for encoding with the LZMA1 and LZMA2 filters." ON)

if(Threads_FOUND)
	list(APPEND CMAKE_REQUIRED_LIBRARIES Threads::Threads)
endif()

#cmakedefine AC_APPLE_UNIVERSAL_BUILD 1
set(ASSUME_RAM 128)
set(ENABLE_NLS ${LIBLZMA_ENABLE_NLS})
check_symbol_exists(bswap_16 "byteswap.h" HAVE_BSWAP_16)
check_symbol_exists(bswap_32 "byteswap.h" HAVE_BSWAP_32)
check_symbol_exists(bswap_64 "byteswap.h" HAVE_BSWAP_64)
check_include_files(byteswap.h HAVE_BYTESWAP_H)
#cmakedefine HAVE_CAPSICUM 1
#cmakedefine HAVE_CC_SHA256_CTX 1
#cmakedefine HAVE_CC_SHA256_INIT 1
check_c_source_compiles("
	#include <CoreFoundation/CFLocale.h>
	int main() { CFLocaleCopyCurrent(); return 0; }
" HAVE_CFLOCALECOPYCURRENT)
check_c_source_compiles("
	#include <CoreFoundation/CFPreferences.h>
	int main() { CFPreferencesCopyAppValue(NULL, NULL); return 0; }
" HAVE_CFPREFERENCESCOPYAPPVALUE)
set(HAVE_CHECK_CRC32 ${LIBLZMA_COND_CHECK_CRC32})
set(HAVE_CHECK_CRC64 ${LIBLZMA_COND_CHECK_CRC64})
set(HAVE_CHECK_SHA256 ${LIBLZMA_COND_CHECK_SHA256})
check_function_exists(clock_gettime HAVE_CLOCK_GETTIME)
check_include_files(CommonCrypto/CommonDigest.h HAVE_COMMONCRYPTO_COMMONDIGEST_H)
check_function_exists(dcgettext HAVE_DCGETTEXT)
#cmakedefine HAVE_DECL_CLOCK_MONOTONIC 1
check_symbol_exists(program_invocation_name "errno.h" HAVE_DECL_PROGRAM_INVOCATION_NAME)
set(HAVE_DECODERS ${LIBLZMA_COND_MAIN_DECODER})
set(HAVE_DECODER_ARM ${LIBLZMA_COND_FILTER_ARM})
set(HAVE_DECODER_ARMTHUMB ${LIBLZMA_COND_FILTER_ARMTHUMB})
set(HAVE_DECODER_DELTA ${LIBLZMA_COND_DECODER_DELTA})
set(HAVE_DECODER_IA64 ${LIBLZMA_COND_FILTER_IA64})
set(HAVE_DECODER_LZMA1 ${LIBLZMA_COND_DECODER_LZMA1})
set(HAVE_DECODER_LZMA2 ${LIBLZMA_COND_DECODER_LZMA2})
set(HAVE_DECODER_POWERPC ${LIBLZMA_COND_FILTER_POWERPC})
set(HAVE_DECODER_SPARC ${LIBLZMA_COND_FILTER_SPARC})
set(HAVE_DECODER_X86 ${LIBLZMA_COND_FILTER_X86})
check_include_files(dlfcn.h HAVE_DLFCN_H)
set(HAVE_ENCODERS ${LIBLZMA_COND_MAIN_ENCODER})
set(HAVE_ENCODER_ARM ${LIBLZMA_COND_FILTER_ARM})
set(HAVE_ENCODER_ARMTHUMB ${LIBLZMA_COND_FILTER_ARMTHUMB})
set(HAVE_ENCODER_DELTA ${LIBLZMA_COND_ENCODER_DELTA})
set(HAVE_ENCODER_IA64 ${LIBLZMA_COND_FILTER_IA64})
set(HAVE_ENCODER_LZMA1 ${LIBLZMA_COND_ENCODER_LZMA1})
set(HAVE_ENCODER_LZMA2 ${LIBLZMA_COND_ENCODER_LZMA2})
set(HAVE_ENCODER_POWERPC ${LIBLZMA_COND_FILTER_POWERPC})
set(HAVE_ENCODER_SPARC ${LIBLZMA_COND_FILTER_SPARC})
set(HAVE_ENCODER_X86 ${LIBLZMA_COND_FILTER_X86})
check_include_files(fcntl.h HAVE_FCNTL_H)
#cmakedefine HAVE_FUTIMENS 1
#cmakedefine HAVE_FUTIMES 1
#cmakedefine HAVE_FUTIMESAT 1
check_include_files(getopt.h HAVE_GETOPT_H)
#cmakedefine HAVE_GETOPT_LONG 1
#cmakedefine HAVE_GETTEXT 1
check_function_exists(iconv HAVE_ICONV)
check_include_files(immintrin.h HAVE_IMMINTRIN_H)
check_include_files(inttypes.h HAVE_INTTYPES_H)
check_include_files(limits.h HAVE_LIMITS_H)
check_function_exists(mbrtowc HAVE_MBRTOWC)
check_include_files(memory.h HAVE_MEMORY_H)
set(HAVE_MF_BT2 ${LIBLZMA_HAVE_MF_BT2})
set(HAVE_MF_BT3 ${LIBLZMA_HAVE_MF_BT3})
set(HAVE_MF_BT4 ${LIBLZMA_HAVE_MF_BT4})
set(HAVE_MF_HC3 ${LIBLZMA_HAVE_MF_HC3})
set(HAVE_MF_HC4 ${LIBLZMA_HAVE_MF_HC4})
#cmakedefine HAVE_OPTRESET 1
check_function_exists(posix_fadvise HAVE_POSIX_FADVISE)
check_function_exists(pthread_condattr_setclock HAVE_PTHREAD_CONDATTR_SETCLOCK)
#cmakedefine HAVE_PTHREAD_PRIO_INHERIT 1
#cmakedefine HAVE_SHA256INIT 1
#cmakedefine HAVE_SHA256_CTX 1
check_include_files(sha256.h HAVE_SHA256_H)
#cmakedefine HAVE_SHA256_INIT 1
#cmakedefine HAVE_SHA2_CTX 1
check_include_files(sha2.h HAVE_SHA2_H)
set(HAVE_SMALL ${LIBLZMA_COND_SMALL})
check_include_files(stdbool.h HAVE_STDBOOL_H)
check_include_files(stdint.h HAVE_STDINT_H)
check_include_files(stdlib.h HAVE_STDLIB_H)
check_include_files(strings.h HAVE_STRINGS_H)
check_include_files(string.h HAVE_STRING_H)
#cmakedefine HAVE_STRUCT_STAT_ST_ATIMENSEC 1
#cmakedefine HAVE_STRUCT_STAT_ST_ATIMESPEC_TV_NSEC 1
#cmakedefine HAVE_STRUCT_STAT_ST_ATIM_ST__TIM_TV_NSEC 1
#cmakedefine HAVE_STRUCT_STAT_ST_ATIM_TV_NSEC 1
#cmakedefine HAVE_STRUCT_STAT_ST_UATIME 1
check_include_files(sys/byteorder.h HAVE_SYS_BYTEORDER_H)
check_include_files(sys/capsicum.h HAVE_SYS_CAPSICUM_H)
check_include_files(sys/endian.h HAVE_SYS_ENDIAN_H)
check_include_files(sys/param.h HAVE_SYS_PARAM_H)
check_include_files(sys/stat.h HAVE_SYS_STAT_H)
check_include_files(sys/time.h HAVE_SYS_TIME_H)
check_include_files(sys/types.h HAVE_SYS_TYPES_H)
check_type_size("uintptr_t" UINTPTR_T)
check_include_files(unistd.h HAVE_UNISTD_H)
#cmakedefine HAVE_UTIME 1
#cmakedefine HAVE_UTIMES 1
check_c_compiler_flag("-fvisibility=hidden" HAVE_VISIBILITY)
#cmakedefine HAVE_WCWIDTH 1
check_type_size("_Bool" _BOOL)
#cmakedefine HAVE__FUTIME 1
#cmakedefine HAVE__MM_MOVEMASK_EPI8 1
#cmakedefine LT_OBJDIR @LT_OBJDIR@
if(Threads_FOUND)
	if(NOT WIN32)
		set(MYTHREAD_POSIX ON)
	else()
		set(MYTHREAD_WIN95 ON)
		set(MYTHREAD_VISTA ON)
	endif()
endif()
#cmakedefine NDEBUG 1
set(PACKAGE "xz")
set(PACKAGE_BUGREPORT "lasse.collin@tukaani.org")
set(PACKAGE_NAME "XZ Utils")
set(PACKAGE_STRING "${PACKAGE_NAME} ${LIBLZMA_VERSION}")
set(PACKAGE_TARNAME "xz")
set(PACKAGE_URL "http://tukaani.org/xz/")
set(PACKAGE_VERSION "${LIBLZMA_VERSION}")
#cmakedefine PTHREAD_CREATE_JOINABLE @PTHREAD_CREATE_JOINABLE@
check_type_size("size_t" SIZEOF_SIZE_T)
check_include_files("assert.h;ctype.h;errno.h;float.h;limits.h;locale.h;math.h;setjmp.h;signal.h;stdarg.h;stddef.h;stdio.h;stdlib.h;string.h;time.h" STDC_HEADERS)
check_c_source_compiles("
	#define _GNU_SOURCE
	#include <sys/param.h>
	#include <sys/cpuset.h>
	int
	main(void)
	{
		cpuset_t set;
		cpuset_getaffinity(CPU_LEVEL_WHICH, CPU_WHICH_PID, -1, sizeof(set), &set);
		return 0;
	}
" TUKLIB_CPUCORES_CPUSET)
check_c_source_compiles("
	#include <sys/param.h>
	#include <sys/pstat.h>
	int
	main(void)
	{
		struct pst_dynamic pst;
		pstat_getdynamic(&pst, sizeof(pst), 1, 0);
		(void)pst.psd_proc_cnt;
		return 0;
	}
" TUKLIB_CPUCORES_PSTAT_GETDYNAMIC)
check_c_source_compiles("
	#define _GNU_SOURCE
	#include <sched.h>
	int
	main(void)
	{
		cpu_set_t cpu_mask;
		sched_getaffinity(0, sizeof(cpu_mask), &cpu_mask);
		return CPU_COUNT(&cpu_mask);
	}
" TUKLIB_CPUCORES_SCHED_GETAFFINITY)
check_c_source_compiles("
	#include <unistd.h>
	int
	main(void)
	{
		long i;
	#ifdef _SC_NPROCESSORS_ONLN
		/* Many systems using sysconf() */
		i = sysconf(_SC_NPROCESSORS_ONLN);
	#else
		/* IRIX */
		i = sysconf(_SC_NPROC_ONLN);
	#endif
		return 0;
	}
" TUKLIB_CPUCORES_SYSCONF)
#cmakedefine TUKLIB_CPUCORES_SYSCONF 1
check_c_source_compiles("
	#ifdef __QNX__
	compile error
	#endif
	#include <sys/types.h>
	#ifdef HAVE_SYS_PARAM_H
	#include <sys/param.h>
	#endif
	#include <sys/sysctl.h>
	int
	main(void)
	{
		int name[2] = { CTL_HW, HW_NCPU };
		int cpus;
		size_t cpus_size = sizeof(cpus);
		sysctl(name, 2, &cpus, &cpus_size, NULL, 0);
		return 0;
	}
" TUKLIB_CPUCORES_SYSCTL)
if(CMAKE_SYSTEM_PROCESSOR MATCHES "i686.*|x86.*|x86_64.*|amd64.*|AMD64.*|powerpc.*|ppc.*|PPC.*|powerpc64.*|ppc64.*|PPC64.*")
	set(TUKLIB_FAST_UNALIGNED_ACCESS 1)
endif()
check_c_source_compiles("
	#include <sys/systemcfg.h>
	int
	main(void)
	{
		(void)_system_configuration.physmem;
		return 0;
	}
" TUKLIB_PHYSMEM_AIX)
check_c_source_compiles("
	#include <invent.h>
	int
	main(void)
	{
		inv_state_t *st = NULL;
		setinvent_r(&st);
		getinvent_r(st);
		endinvent_r(st);
		return 0;
	}
" TUKLIB_PHYSMEM_GETINVENT_R)
check_c_source_compiles("
	#include <sys/sysinfo.h>
	#include <machine/hal_sysinfo.h>
	int
	main(void)
	{
		int memkb;
		int start = 0;
		getsysinfo(GSI_PHYSMEM, (caddr_t)&memkb, sizeof(memkb), &start);
		return 0;
	}
" TUKLIB_PHYSMEM_GETSYSINFO)
check_c_source_compiles("
	#include <sys/param.h>
	#include <sys/pstat.h>
	int
	main(void)
	{
		struct pst_static pst;
		pstat_getstatic(&pst, sizeof(pst), 1, 0);
		(void)pst.physical_memory;
		(void)pst.page_size;
		return 0;
	}
" TUKLIB_PHYSMEM_PSTAT_GETSTATIC)
check_c_source_compiles("
	#include <unistd.h>
	int
	main(void)
	{
		long i;
		i = sysconf(_SC_PAGESIZE);
		i = sysconf(_SC_PHYS_PAGES);
		return 0;
	}
" TUKLIB_PHYSMEM_SYSCONF)
check_c_source_compiles("
	#include <sys/types.h>
	#ifdef HAVE_SYS_PARAM_H
	#include <sys/param.h>
	#endif
	#include <sys/sysctl.h>
	int
	main(void)
	{
		int name[2] = { CTL_HW, HW_PHYSMEM };
		unsigned long mem;
		size_t mem_ptr_size = sizeof(mem);
		sysctl(name, 2, &mem, &mem_ptr_size, NULL, 0);
		return 0;
	}
" TUKLIB_PHYSMEM_SYSCTL)
check_c_source_compiles("
	#include <sys/sysinfo.h>
	int
	main(void)
	{
		struct sysinfo si;
		sysinfo(&si);
		return 0;
	}
" TUKLIB_PHYSMEM_SYSINFO)
set(_ALL_SOURCE ON)
set(_GNU_SOURCE ON)
set(_POSIX_PTHREAD_SEMANTICS ON)
set(_TANDEM_SOURCE ON)
set(__EXTENSIONS__ ON)
set(VERSION "${LIBLZMA_VERSION}")
test_big_endian(WORDS_BIGENDIAN)
#cmakedefine _FILE_OFFSET_BITS @_FILE_OFFSET_BITS@
#cmakedefine _LARGE_FILES 1
#cmakedefine _MINIX 1
#cmakedefine _POSIX_1_SOURCE 2
#cmakedefine _POSIX_SOURCE 1
#cmakedefine _UINT32_T 1
#cmakedefine _UINT64_T 1
#cmakedefine _UINT8_T 1
#cmakedefine __GETOPT_PREFIX @__GETOPT_PREFIX@
#cmakedefine int32_t @int32_t@
#cmakedefine int64_t @int64_t@
#cmakedefine uint16_t @uint16_t@
#cmakedefine uint32_t @uint32_t@
#cmakedefine uint64_t @uint64_t@
#cmakedefine uint8_t @uint8_t@
#cmakedefine uintptr_t @uintptr_t@

configure_file(config.h.cmake.in config.h)

set(
	HDRS
	src/liblzma/api/lzma.h
)

set(
	HDRS_LZMA
	src/liblzma/api/lzma/base.h
	src/liblzma/api/lzma/bcj.h
	src/liblzma/api/lzma/block.h
	src/liblzma/api/lzma/check.h
	src/liblzma/api/lzma/container.h
	src/liblzma/api/lzma/delta.h
	src/liblzma/api/lzma/filter.h
	src/liblzma/api/lzma/hardware.h
	src/liblzma/api/lzma/index.h
	src/liblzma/api/lzma/index_hash.h
	src/liblzma/api/lzma/lzma12.h
	src/liblzma/api/lzma/stream_flags.h
	src/liblzma/api/lzma/version.h
	src/liblzma/api/lzma/vli.h
)

set(
	SRCS
	src/common/tuklib_physmem.c
	src/liblzma/check/check.c
	src/liblzma/check/check.h
	src/liblzma/check/crc_macros.h
	src/liblzma/common/block_util.c
	src/liblzma/common/common.c
	src/liblzma/common/common.h
	src/liblzma/common/easy_preset.c
	src/liblzma/common/easy_preset.h
	src/liblzma/common/filter_common.c
	src/liblzma/common/filter_common.h
	src/liblzma/common/hardware_physmem.c
	src/liblzma/common/index.c
	src/liblzma/common/index.h
	src/liblzma/common/memcmplen.h
	src/liblzma/common/stream_flags_common.c
	src/liblzma/common/stream_flags_common.h
	src/liblzma/common/vli_size.c
)

if(LIBLZMA_COND_THREADS)
	list(
		APPEND SRCS
		src/common/tuklib_cpucores.c
		src/liblzma/common/hardware_cputhreads.c
	)
endif()

if(LIBLZMA_COND_MAIN_ENCODER)
	list(
		APPEND SRCS
		src/liblzma/common/alone_encoder.c
		src/liblzma/common/block_buffer_encoder.c
		src/liblzma/common/block_buffer_encoder.h
		src/liblzma/common/block_encoder.c
		src/liblzma/common/block_encoder.h
		src/liblzma/common/block_header_encoder.c
		src/liblzma/common/easy_buffer_encoder.c
		src/liblzma/common/easy_encoder.c
		src/liblzma/common/easy_encoder_memusage.c
		src/liblzma/common/filter_buffer_encoder.c
		src/liblzma/common/filter_encoder.c
		src/liblzma/common/filter_encoder.h
		src/liblzma/common/filter_flags_encoder.c
		src/liblzma/common/index_encoder.c
		src/liblzma/common/index_encoder.h
		src/liblzma/common/stream_buffer_encoder.c
		src/liblzma/common/stream_encoder.c
		src/liblzma/common/stream_flags_encoder.c
		src/liblzma/common/vli_encoder.c
	)
	if(LIBLZMA_COND_THREADS)
		list(
			APPEND SRCS
			src/liblzma/common/outqueue.c
			src/liblzma/common/outqueue.h
			src/liblzma/common/stream_encoder_mt.c
		)
	endif()
endif()

if(LIBLZMA_COND_MAIN_DECODER)
	list(
		APPEND SRCS
		src/liblzma/common/alone_decoder.c
		src/liblzma/common/alone_decoder.h
		src/liblzma/common/auto_decoder.c
		src/liblzma/common/block_buffer_decoder.c
		src/liblzma/common/block_decoder.c
		src/liblzma/common/block_decoder.h
		src/liblzma/common/block_header_decoder.c
		src/liblzma/common/easy_decoder_memusage.c
		src/liblzma/common/file_info.c
		src/liblzma/common/filter_buffer_decoder.c
		src/liblzma/common/filter_decoder.c
		src/liblzma/common/filter_decoder.h
		src/liblzma/common/filter_flags_decoder.c
		src/liblzma/common/index_decoder.c
		src/liblzma/common/index_decoder.h
		src/liblzma/common/index_hash.c
		src/liblzma/common/stream_buffer_decoder.c
		src/liblzma/common/stream_decoder.c
		src/liblzma/common/stream_decoder.h
		src/liblzma/common/stream_flags_decoder.c
		src/liblzma/common/vli_decoder.c
	)
endif()

if(LIBLZMA_COND_CHECK_CRC32)
	if(LIBLZMA_COND_SMALL)
		list(APPEND SRCS src/liblzma/check/crc32_small.c)
	else()
		list(
			APPEND SRCS
			src/liblzma/check/crc32_table.c
			src/liblzma/check/crc32_table_le.h
			src/liblzma/check/crc32_table_be.h
		)
		if(LIBLZMA_COND_ASM_X86 AND CMAKE_SYSTEM_PROCESSOR MATCHES "i686.*|x86.*" AND CMAKE_SIZEOF_VOID_P EQUAL 4)
			list(APPEND SRCS src/liblzma/check/crc32_x86.S)
			set_source_files_properties(src/liblzma/check/crc32_x86.S PROPERTIES LANGUAGE C)
		else()
			list(APPEND SRCS src/liblzma/check/crc32_fast.c)
		endif()
	endif()
endif()

if(LIBLZMA_COND_CHECK_CRC64)
	if(LIBLZMA_COND_SMALL)
		list(APPEND SRCS src/liblzma/check/crc64_small.c)
	else()
		list(
			APPEND SRCS
			src/liblzma/check/crc64_table.c
			src/liblzma/check/crc64_table_le.h
			src/liblzma/check/crc64_table_be.h
		)
		if(LIBLZMA_COND_ASM_X86 AND CMAKE_SYSTEM_PROCESSOR MATCHES "i686.*|x86.*" AND CMAKE_SIZEOF_VOID_P EQUAL 4)
			list(APPEND SRCS src/liblzma/check/crc64_x86.S)
			set_source_files_properties(src/liblzma/check/crc64_x86.S PROPERTIES LANGUAGE C)
		else()
			list(APPEND SRCS src/liblzma/check/crc64_fast.c)
		endif()
	endif()
endif()

if(LIBLZMA_COND_CHECK_SHA256)
	if(LIBLZMA_COND_INTERNAL_SHA256)
		list(APPEND SRCS src/liblzma/check/sha256.c)
	endif()
endif()

if(LIBLZMA_COND_FILTER_LZ)
	if(LIBLZMA_COND_ENCODER_LZ)
		list(
			APPEND SRCS 
			src/liblzma/lz/lz_encoder.c
			src/liblzma/lz/lz_encoder.h
			src/liblzma/lz/lz_encoder_hash.h
			src/liblzma/lz/lz_encoder_hash_table.h
			src/liblzma/lz/lz_encoder_mf.c
		)
	endif()
	if(LIBLZMA_COND_DECODER_LZ)
		list(
			APPEND SRCS 
			src/liblzma/lz/lz_decoder.c
			src/liblzma/lz/lz_decoder.h
		)
	endif()
endif()

if(LIBLZMA_COND_FILTER_LZMA1)
	list(APPEND SRCS src/liblzma/lzma/lzma_common.h)
	if(LIBLZMA_COND_FILTER_LZMA1)
		list(APPEND SRCS src/liblzma/lzma/lzma_encoder_presets.c)
	endif()
	if(LIBLZMA_COND_ENCODER_LZMA1)
		list(
			APPEND SRCS
			src/liblzma/lzma/fastpos.h
			src/liblzma/lzma/lzma_encoder.h
			src/liblzma/lzma/lzma_encoder.c
			src/liblzma/lzma/lzma_encoder_private.h
			src/liblzma/lzma/lzma_encoder_optimum_fast.c
			src/liblzma/lzma/lzma_encoder_optimum_normal.c
		)
		if(NOT LIBLZMA_COND_SMALL)
			list(APPEND SRCS src/liblzma/lzma/fastpos_table.c)
		endif()
	endif()
	if(LIBLZMA_COND_DECODER_LZMA1)
		list(
			APPEND SRCS
			src/liblzma/lzma/lzma_decoder.c
			src/liblzma/lzma/lzma_decoder.h
		)
	endif()
	if(LIBLZMA_COND_ENCODER_LZMA2)
		list(
			APPEND SRCS
			src/liblzma/lzma/lzma2_encoder.c
			src/liblzma/lzma/lzma2_encoder.h
		)
	endif()
	if(LIBLZMA_COND_DECODER_LZMA2)
		list(
			APPEND SRCS
			src/liblzma/lzma/lzma2_decoder.c
			src/liblzma/lzma/lzma2_decoder.h
		)
	endif()
	list(APPEND SRCS src/liblzma/rangecoder/range_common.h)
	if(LIBLZMA_COND_ENCODER_LZMA1)
		list(
			APPEND SRCS
			src/liblzma/rangecoder/range_encoder.h
			src/liblzma/rangecoder/price.h
			src/liblzma/rangecoder/price_table.c
		)
	endif()
	if(LIBLZMA_COND_DECODER_LZMA1)
		list(APPEND SRCS src/liblzma/rangecoder/range_decoder.h)
	endif()
endif()

if(LIBLZMA_COND_FILTER_DELTA)
	list(
		APPEND SRCS
		src/liblzma/delta/delta_common.c
		src/liblzma/delta/delta_common.h
		src/liblzma/delta/delta_private.h
	)
	if(LIBLZMA_COND_ENCODER_DELTA)
		list(
			APPEND SRCS
			src/liblzma/delta/delta_encoder.c
			src/liblzma/delta/delta_encoder.h
		)
	endif()
	if(LIBLZMA_COND_DECODER_DELTA)
		list(
			APPEND SRCS
			src/liblzma/delta/delta_decoder.c
			src/liblzma/delta/delta_decoder.h
		)
	endif()
endif()

if(LIBLZMA_COND_FILTER_SIMPLE)
	list(
		APPEND SRCS
		src/liblzma/simple/simple_coder.c
		src/liblzma/simple/simple_coder.h
		src/liblzma/simple/simple_private.h
	)
	if(LIBLZMA_COND_ENCODER_SIMPLE)
		list(
			APPEND SRCS
			src/liblzma/simple/simple_encoder.c
			src/liblzma/simple/simple_encoder.h
		)
	endif()
	if(LIBLZMA_COND_DECODER_SIMPLE)
		list(
			APPEND SRCS
			src/liblzma/simple/simple_decoder.c
			src/liblzma/simple/simple_decoder.h
		)
	endif()
	if(LIBLZMA_COND_FILTER_X86)
		list(APPEND SRCS src/liblzma/simple/x86.c)
	endif()
	if(LIBLZMA_COND_FILTER_POWERPC)
		list(APPEND SRCS src/liblzma/simple/powerpc.c)
	endif()
	if(LIBLZMA_COND_FILTER_IA64)
		list(APPEND SRCS src/liblzma/simple/ia64.c)
	endif()
	if(LIBLZMA_COND_FILTER_ARM)
		list(APPEND SRCS src/liblzma/simple/arm.c)
	endif()
	if(LIBLZMA_COND_FILTER_ARMTHUMB)
		list(APPEND SRCS src/liblzma/simple/armthumb.c)
	endif()
	if(LIBLZMA_COND_FILTER_SPARC)
		list(APPEND SRCS src/liblzma/simple/sparc.c)
	endif()
endif()

if(WIN32)
	list(APPEND SRCS src/liblzma/liblzma_w32res.rc)
endif()

add_library(LibLZMA ${SRCS})

set_target_properties(
	LibLZMA
	PROPERTIES
	IMPORT_PREFIX lib
	OUTPUT_NAME lzma
	POSITION_INDEPENDENT_CODE ON
	PREFIX lib
	VERSION ${LIBLZMA_VERSION}
)

if(WIN32)
	if(BUILD_SHARED_LIBS)
		set_target_properties(
			LibLZMA
			PROPERTIES
			DEBUG_POSTFIX d
		)
	else()
		set_target_properties(
			LibLZMA
			PROPERTIES
			DEBUG_POSTFIX sd
			MINSIZEREL_POSTFIX s
			RELEASE_POSTFIX s
			RELWITHDEBINFO_POSTFIX s
		)
	endif()
endif()

target_compile_definitions(LibLZMA PRIVATE HAVE_CONFIG_H TUKLIB_SYMBOL_PREFIX=lzma_)

if(BUILD_SHARED_LIBS)
	target_compile_definitions(LibLZMA PRIVATE _USRDLL DLL_EXPORT LIBLZMADLL_EXPORTS)
else()
	target_compile_definitions(LibLZMA PRIVATE _LIB)
endif()

if(LIBLZMA_ENABLE_NLS)
	target_compile_definitions(LibLZMA PRIVATE ENABLE_NLS=1)
endif()

target_include_directories(
	LibLZMA
	PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/common>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/liblzma/api>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/liblzma/check>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/liblzma/common>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/liblzma/delta>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/liblzma/lz>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/liblzma/lzma>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/liblzma/rangecoder>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/liblzma/simple>
	$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>
)

if(LIBLZMA_COND_THREADS)
	target_link_libraries(LibLZMA Threads::Threads)
endif()

install(FILES ${HDRS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} COMPONENT development)
install(FILES ${HDRS_LZMA} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/lzma COMPONENT development)

install(
	TARGETS	LibLZMA
	EXPORT LibLZMA
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT development
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtime NAMELINK_SKIP
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT runtime
)

if(BUILD_SHARED_LIBS)
	install(
		TARGETS	LibLZMA
		EXPORT LibLZMA
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT development NAMELINK_ONLY
	)
endif()

if(MSVC AND BUILD_SHARED_LIBS AND NOT CMAKE_VERSION VERSION_LESS 3.1)
	install(FILES $<TARGET_PDB_FILE:LibLZMA> DESTINATION ${CMAKE_INSTALL_BINDIR} CONFIGURATIONS Debug RelWithDebInfo COMPONENT debug)
endif()

check_include_files(libintl.h HAVE_LIBINTL_H)

if(LIBLZMA_COND_XZ AND HAVE_LIBINTL_H AND HAVE_UNISTD_H)
	set(
		XZ_SRCS
		src/common/tuklib_exit.c
		src/common/tuklib_mbstr_fw.c
		src/common/tuklib_mbstr_width.c
		src/common/tuklib_open_stdxxx.c
		src/common/tuklib_progname.c
		src/xz/args.c
		src/xz/args.h
		src/xz/coder.c
		src/xz/coder.h
		src/xz/file_io.c
		src/xz/file_io.h
		src/xz/hardware.c
		src/xz/hardware.h
		src/xz/main.c
		src/xz/main.h
		src/xz/message.c
		src/xz/message.h
		src/xz/mytime.c
		src/xz/mytime.h
		src/xz/options.c
		src/xz/options.h
		src/xz/private.h
		src/xz/signals.c
		src/xz/signals.h
		src/xz/suffix.c
		src/xz/suffix.h
		src/xz/util.c
		src/xz/util.h
	)
	if(LIBLZMA_COND_MAIN_DECODER)
		list(
			APPEND XZ_SRCS
			src/xz/list.c
			src/xz/list.h
		)
	endif()
	if(WIN32)
		list(APPEND XZ_SRCS src/xz/xz_w32res.rc)
	endif()
	add_executable(xz ${XZ_SRCS})
	target_compile_definitions(xz PRIVATE HAVE_CONFIG_H LOCALEDIR="$(CMAKE_INSTALL_LOCALEDIR)")
	target_include_directories(
		xz
		PRIVATE
		$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/common>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/liblzma/api>
	)
	target_link_libraries(xz LibLZMA)
#	xz_LDADD = $(CAPSICUM_LIB)
	if(LIBLZMA_COND_GNULIB)
#		xz_LDADD += $(top_builddir)/lib/libgnu.a
	endif()
endif()

if(LIBLZMA_COND_LZMAINFO AND HAVE_LIBINTL_H)
	set(
		LZMAINFO_SRCS
		src/common/tuklib_exit.c
		src/common/tuklib_progname.c
		src/lzmainfo/lzmainfo.c
	)
	if(WIN32)
		list(APPEND LZMAINFO_SRCS src/lzmainfo/lzmainfo_w32res.rc)
	endif()
	add_executable(lzmainfo ${LZMAINFO_SRCS})
	target_compile_definitions(lzmainfo PRIVATE HAVE_CONFIG_H LOCALEDIR="$(CMAKE_INSTALL_LOCALEDIR)")
	target_include_directories(
		lzmainfo
		PRIVATE
		$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/common>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/liblzma/api>
	)
	target_link_libraries(lzmainfo LibLZMA)
	if(LIBLZMA_COND_GNULIB)
#		lzmainfo_LDADD += $(top_builddir)/lib/libgnu.a
	endif()
#	lzmainfo_LDADD += $(LTLIBINTL)
endif()

configure_package_config_file(
	liblzma-config.cmake.in liblzma-config.cmake
	INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/liblzma-${VERSION}
)

install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/liblzma-config.cmake
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/liblzma-${VERSION}
	COMPONENT development
)

write_basic_package_version_file(
	${CMAKE_CURRENT_BINARY_DIR}/liblzma-config-version.cmake
	VERSION ${VERSION}
	COMPATIBILITY ExactVersion
)

install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/liblzma-config-version.cmake
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/liblzma-${VERSION}
	COMPONENT development
)

install(
	EXPORT LibLZMA
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/liblzma-${VERSION}
	NAMESPACE LibLZMA::
	FILE liblzma-export.cmake
	COMPONENT development
)
