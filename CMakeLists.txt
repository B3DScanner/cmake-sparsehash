cmake_minimum_required(VERSION 3.10)
project(SparseHash VERSION 1.2 LANGUAGES C CXX)

option(INSTALL_DOC "Install documentation files" OFF)
option(BUILD_TESTS "Build and run tests" OFF)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -W -Wwrite-strings -Woverloaded-virtual -Wshadow")

include(CheckIncludeFiles)
check_include_files("inttypes.h" HAVE_INTTYPES_H)
check_include_files("stdint.h" HAVE_STDINT_H)
check_include_files("sys/types.h" HAVE_SYS_TYPES_H)

if(NOT HAVE_STDINT_H)
	message( FATAL_ERROR "inttypes.h Not Found" )
endif()

include(CheckFunctionExists)
check_function_exists(memcpy HAVE_MEMCPY)

include(CheckTypeSize)
check_type_size("uint16_t" HAVE_UINT16_T)
check_type_size("long long" HAVE_LONG_LONG)

if(MSVC)
    set(HASH_FUN_H "<hash_map>")
    set(HASH_NAMESPACE "stdext")
else()
    set(HASH_FUN_H "<unordered_map>")
    set(HASH_NAMESPACE "std")
endif()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/sparseconfig.h.in
               ${CMAKE_CURRENT_BINARY_DIR}/sparseconfig.h)
			   
include_directories(${CMAKE_CURRENT_BINARY_DIR})

set(SPARSEHASH_HEADERS
    src/sparsehash/dense_hash_map
    src/sparsehash/dense_hash_set
    src/sparsehash/sparse_hash_map
    src/sparsehash/sparse_hash_set
    src/sparsehash/sparsetable
    src/sparsehash/template_util.h
    src/sparsehash/type_traits.h
)

set(SPARSEHASH_INTERNAL_HEADERS
    src/sparsehash/internal/densehashtable.h
    src/sparsehash/internal/sparsehashtable.h
    src/sparsehash/internal/hashtable-common.h
    src/sparsehash/internal/libc_allocator_with_realloc.h
)

set(SPARSEHASH_INCLUDE_DIR "${CMAKE_INSTALL_PREFIX}/include")

install(FILES ${SPARSEHASH_HEADERS}
    DESTINATION ${SPARSEHASH_INCLUDE_DIR}/sparsehash
)

install(FILES ${SPARSEHASH_INTERNAL_HEADERS} ${CMAKE_CURRENT_BINARY_DIR}/sparseconfig.h
    DESTINATION ${SPARSEHASH_INCLUDE_DIR}/sparsehash/internal
)

if(INSTALL_DOC)
    set(DOC_FILES
        AUTHORS COPYING ChangeLog INSTALL NEWS README README_windows.txt TODO
        doc/dense_hash_map.html doc/dense_hash_set.html
        doc/sparse_hash_map.html doc/sparse_hash_set.html
        doc/sparsetable.html doc/implementation.html
        doc/performance.html doc/index.html doc/designstyle.css
    )
    install(FILES ${DOC_FILES} DESTINATION ${CMAKE_INSTALL_PREFIX}/share/doc/${PROJECT_NAME})
endif()

if(BUILD_TESTS)
    enable_testing()
    add_executable(template_util_unittest src/template_util_unittest.cc)
    target_include_directories(template_util_unittest PRIVATE src)
    add_test(NAME template_util_unittest COMMAND template_util_unittest)

    add_executable(type_traits_unittest src/type_traits_unittest.cc)
    target_include_directories(type_traits_unittest PRIVATE src)
    add_test(NAME type_traits_unittest COMMAND type_traits_unittest)

    add_executable(libc_allocator_with_realloc_test src/libc_allocator_with_realloc_test.cc)
    target_include_directories(libc_allocator_with_realloc_test PRIVATE src)
    add_test(NAME libc_allocator_with_realloc_test COMMAND libc_allocator_with_realloc_test)

    add_executable(sparsetable_unittest src/sparsetable_unittest.cc)
    target_include_directories(sparsetable_unittest PRIVATE src)
    add_test(NAME sparsetable_unittest COMMAND sparsetable_unittest)

    add_executable(simple_test src/simple_test.cc)
    target_include_directories(simple_test PRIVATE src)
    add_test(NAME simple_test COMMAND simple_test)
endif()

add_library(${PROJECT_NAME} INTERFACE)
target_include_directories(${PROJECT_NAME} INTERFACE ${SPARSEHASH_INCLUDE_DIR})

install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}Targets
    INCLUDES DESTINATION include
)

install(EXPORT ${PROJECT_NAME}Targets
    FILE ${PROJECT_NAME}Targets.cmake
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake/${PROJECT_NAME}
)

install(DIRECTORY ${SPARSEHASH_INCLUDE_DIR}/
    DESTINATION ${CMAKE_INSTALL_PREFIX}/include
    FILES_MATCHING PATTERN "*.h"
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    @ONLY
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    DESTINATION lib/cmake/${PROJECT_NAME}
)

export(PACKAGE ${PROJECT_NAME})