# Copyright (c) Open Enclave SDK contributors.
# Licensed under the MIT License.

add_enclave_library(oedlmalloc_obj OBJECT allocator.c)

enclave_link_libraries(oedlmalloc_obj PRIVATE oe_includes oelibc_includes)

if (OE_TRUSTZONE)
  enclave_link_libraries(oedlmalloc_obj PUBLIC oelibutee_includes)
  set(TEE_C_FLAGS ${OE_TZ_TA_C_FLAGS})
else ()
  set(TEE_C_FLAGS "")
endif ()

enclave_compile_options(
  oedlmalloc_obj
  PRIVATE
  -ftls-model=local-exec
  -nostdinc
  -fPIE
  -ffreestanding
  -fvisibility=hidden
  ${TEE_C_FLAGS})

# Specify the warning options as source files properties so that
# they will appear last in the compiler command line and supercede
# other warning options.
# -Wno-unused-but-set-variable is supported by GCC and newer clang (12+)
# but not by clang-11, so probe before use.
include(CheckCCompilerFlag)
check_c_compiler_flag(
  "-Wno-unused-but-set-variable"
  OE_COMPILER_SUPPORTS_WNO_UNUSED_BUT_SET_VARIABLE)
if(OE_COMPILER_SUPPORTS_WNO_UNUSED_BUT_SET_VARIABLE)
  set(OE_DLMALLOC_EXTRA_FLAGS " -Wno-unused-but-set-variable")
else()
  set(OE_DLMALLOC_EXTRA_FLAGS "")
endif()

set_source_files_properties(
  allocator.c
  PROPERTIES
    COMPILE_FLAGS
    "-Wno-conversion -Wno-null-pointer-arithmetic${OE_DLMALLOC_EXTRA_FLAGS}")

maybe_build_using_clangw(oedlmalloc_obj)

install_enclaves(
  TARGETS
  oedlmalloc_obj
  EXPORT
  openenclave-targets
  ARCHIVE
  DESTINATION
  ${CMAKE_INSTALL_LIBDIR}/openenclave/enclave)
