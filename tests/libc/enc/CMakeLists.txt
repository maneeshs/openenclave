# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License.

# Create a binary for each testcase listed in ../tests.supported
# These tests generate a large number of warnings, some of them specific to compiler versions.
# Disabling warning-as-errors for the specific warnings is painful across multiple different compilers.
# It was agreed to, that it is best to just disable warnings-as-errors for these tests specifically.

include(add_enclave_executable)
include(${CMAKE_CURRENT_LIST_DIR}/../../../cmake/get_testcase_name.cmake)

# helper lib to contain file needed by some tests
add_library(libctest-support
    mtest.c
    rand.c
    tls_align_dso.c
    )
target_compile_options(libctest-support PRIVATE
    -Wno-error
    )
target_link_libraries(libctest-support PUBLIC oelibc oeenclave oecore)

function(add_libc_test_enc NAME CFILE)
    add_executable(libctest-${NAME}_enc enc.c main.c)

    target_include_directories(libctest-${NAME}_enc PRIVATE
        ..
        ${PROJECT_SOURCE_DIR}/3rdparty/musl/libc-test/src/common
        )
    target_compile_options(libctest-${NAME}_enc PRIVATE
        -Wno-error
        )
    target_compile_definitions(libctest-${NAME}_enc PRIVATE -D__TEST__="${CFILE}")
    target_link_libraries(libctest-${NAME}_enc libctest-support)
endfunction(add_libc_test_enc)

if (ENABLE_FULL_LIBC_TESTS)
    file(STRINGS "../tests.supported" alltests)
else()
    file(STRINGS "../tests.supported.default" alltests)
endif()
foreach(testcase ${alltests})
    get_testcase_name(${testcase} name "../../3rdparty/musl/libc-test/src/")
    add_libc_test_enc("${name}" "${testcase}")
endforeach(testcase)
