cmake_minimum_required(VERSION 3.25)
set(CMAKE_CUDA_ARCHITECTURES native)
set(CMAKE_CUDA_COMPILER /usr/local/cuda/bin/nvcc)
project(DB LANGUAGES CXX CUDA)
set(CMAKE_TRY_COMPILE_TARGET_TYPE "STATIC_LIBRARY")
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pthread -g -O3 -fopenmp")

add_compile_options(-fPIC)
add_compile_options(-D_GLIBCXX_USE_CXX11_ABI=0)

find_package(CUDA  REQUIRED)
include_directories("${CUDA_INCLUDE_DIRS}")

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# Adjust the way CMake's find_package function works. This determines the order in which versions of packages are found.
SET(CMAKE_FIND_PACKAGE_SORT_ORDER NATURAL)
SET(CMAKE_FIND_PACKAGE_SORT_DIRECTION DEC)

include(FetchContent)

set(seal_patch_command git apply ${CMAKE_CURRENT_SOURCE_DIR}/seal.patch)
set(SEAL_THROW_ON_TRANSPARENT_CIPHERTEXT OFF CACHE BOOL "Allow transparent ctxt" FORCE)
FetchContent_Declare(SEAL
                     GIT_REPOSITORY https://github.com/microsoft/SEAL.git
                     GIT_TAG        v4.0.0
                     PATCH_COMMAND ${seal_patch_command}
)
FetchContent_MakeAvailable(SEAL)

# TFHE
add_library(glwe OBJECT)
include_directories(include)
add_subdirectory(include)
target_link_libraries(glwe PUBLIC SEAL::seal)

add_executable(tfhe_test tfhe_test.cc)
target_link_libraries(tfhe_test glwe gb_cuda)
