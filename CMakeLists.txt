cmake_minimum_required(VERSION 3.10)
project(SparseHash VERSION 1.2 LANGUAGES C CXX)

option(INSTALL_DOC "Install documentation files" OFF)
option(BUILD_TESTS "Build and run tests" OFF)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall")
if(NOT MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wextra -Wpedantic -Wno-missing-field-initializers")
endif()
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g")          

# Tworzymy bibliotekę INTERFACE
add_library(${PROJECT_NAME} INTERFACE)

# Ustawiamy ścieżki do nagłówków
target_include_directories(${PROJECT_NAME} INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/sparsehash>  
    $<INSTALL_INTERFACE:include/sparsehash>
)

# Instalujemy bibliotekę i pliki nagłówkowe
install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}Targets
    PUBLIC_HEADER DESTINATION include
)

file(GLOB_RECURSE SPARSEHASH_HEADERS "sparsehash/**/*" "sparsehash/*" "sparsehash/**/*.h" "sparsehash/*.h")

# Eksportujemy pliki CMake, które umożliwiają użycie tej biblioteki w innych projektach
install(EXPORT ${PROJECT_NAME}Targets
    FILE ${PROJECT_NAME}Targets.cmake
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake/${PROJECT_NAME}
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/sparsehash/
    DESTINATION ${CMAKE_INSTALL_PREFIX}/include/sparsehash
    FILES_MATCHING PATTERN "*"
)


# Tworzymy plik wersji
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

# Konfiguracja pliku CMake dla użytkowników tej biblioteki
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    @ONLY
)

# Instalacja plików konfiguracyjnych
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    DESTINATION lib/cmake/${PROJECT_NAME}
)

# Eksportujemy paczkę
export(PACKAGE ${PROJECT_NAME})