cmake_minimum_required(VERSION 3.13)

project(compiler-rt-builtins LANGUAGES C ASM_NASM)

option(FORCE32_BIT "Force 32 bit" ON)
option(TARGET_ARCH "Target Arch" x86)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_COMPILER clang)

set(BUILTIN_COMPILER_FLAGS -fno-builtin -fvisibility=hidden -fomit-frame-pointer -ffreestanding -Wno-pedantic -Wno-undef -Wno-sign-conversion -Wno-double-promotion -Wno-float-equal -Wno-float-conversion -Wno-conversion -Wno-missing-noreturn -Wno-unused-parameter -Wno-format-nonliteral -Wno-unused-macros)

if (FORCE32_BIT)
    list(APPEND BUILTIN_COMPILER_FLAGS -m32)
else ()
    list(APPEND BUILTIN_COMPILER_FLAGS -m64)
endif ()

check_cxx_compiler_flag("-Wno-builtin-declaration-mismatch" NO_BUILTIN_DECLARATION_MISMATCH_SUPPORTED)
check_cxx_compiler_flag("-Wno-shift-sign-overflow" NO_SHIFT_SIGN_OVERFLOW_SUPPORTED)
check_cxx_compiler_flag("-Wno-shorten-64-to-32" NO_SHORTEN_64_TO_32_SUPPORTED)
check_cxx_compiler_flag("-Wno-unreachable-code-break" NO_UNREACHABLE_CODE_BREAK_SUPPORTED)
check_cxx_compiler_flag("-Wno-conditional-uninitialized" NO_CONDITIONAL_UNINITIALIZED_SUPPORTED)
check_cxx_compiler_flag("-Wno-missing-variable-declarations" NO_MISSING_VAR_DECLARATION_SUPPORTED)
check_cxx_compiler_flag("-Wno-reserved-id-macro" NO_RESERVED_ID_MACRO_SUPPORTED)
check_cxx_compiler_flag("-Wno-missing-prototypes" NO_MISSING_PROTOTYPES_SUPPORTED)

if (NO_BUILTIN_DECLARATION_MISMATCH_SUPPORTED)
    list(APPEND BUILTIN_COMPILER_FLAGS -Wno-builtin-declaration-mismatch)
endif ()

if (NO_SHIFT_SIGN_OVERFLOW_SUPPORTED)
    list(APPEND BUILTIN_COMPILER_FLAGS -Wno-shift-sign-overflow)
endif ()

if (NO_SHORTEN_64_TO_32_SUPPORTED)
    list(APPEND BUILTIN_COMPILER_FLAGS -Wno-shorten-64-to-32)
endif ()

if (NO_UNREACHABLE_CODE_BREAK_SUPPORTED)
    list(APPEND BUILTIN_COMPILER_FLAGS -Wno-unreachable-code-break)
endif ()

if (NO_CONDITIONAL_UNINITIALIZED_SUPPORTED)
    list(APPEND BUILTIN_COMPILER_FLAGS -Wno-conditional-uninitialized)
endif ()

if (NO_MISSING_VAR_DECLARATION_SUPPORTED)
    list(APPEND BUILTIN_COMPILER_FLAGS -Wno-missing-variable-declarations)
endif ()

if (NO_RESERVED_ID_MACRO_SUPPORTED)
    list(APPEND BUILTIN_COMPILER_FLAGS -Wno-reserved-id-macro)
endif ()

if (NO_MISSING_PROTOTYPES_SUPPORTED AND (NOT C_COMPILER MATCHES "gcc"))
    list(APPEND BUILTIN_COMPILER_FLAGS -Wno-missing-prototypes)
endif ()


file(GLOB_RECURSE SOURCES "builtins/*.c")
file(GLOB_RECURSE X86_64_SOURCES "builtins/x86_64/*.c" "builtins/x86_64.*.s")

if (TARGET_ARCH MATCHES "x86_64")
    list(APPEND SOURCES ${X86_64_SOURCES})
endif ()

add_library(${PROJECT_NAME} ${SOURCES})

target_include_directories(${PROJECT_NAME} PRIVATE include)

enable_language(ASM_NASM)
target_compile_options(${PROJECT_NAME} PRIVATE $<$<COMPILE_LANGUAGE:C>: ${BUILTIN_COMPILER_FLAGS}>)