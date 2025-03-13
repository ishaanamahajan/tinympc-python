cmake_minimum_required(VERSION 3.15)
project(tinympc_codegen_ext)

# Enable C++14 and position independent code
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# Set architecture flags for Apple Silicon
if(APPLE)
    set(CMAKE_OSX_ARCHITECTURES "arm64;x86_64")  # Enable universal binary
    set(CMAKE_OSX_DEPLOYMENT_TARGET "11.0")
endif()

# Define source files based on actual directory structure
set(TINY_SOURCES
    src/tiny_data.cpp
    src/tiny_main.cpp
    # tinympc/admm.cpp
    # tinympc/tiny_api.cpp
)

# Create tinympc library
add_library(tinympcstatic STATIC ${TINY_SOURCES})
target_include_directories(tinympcstatic PUBLIC 
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_SOURCE_DIR}/tinympc
)

# Create example main executable
add_executable(tiny_codegen_example ${TINY_SOURCES})
target_link_libraries(tiny_codegen_example LINK_PUBLIC tinympcstatic)

# Create python module
include(FetchContent)
FetchContent_Declare(
  pybind11
  GIT_REPOSITORY https://github.com/pybind/pybind11.git
)
FetchContent_MakeAvailable(pybind11)

# Python extension sources
set(EXT_SOURCES
    src/tiny_data.cpp
    bindings.cpp
)

pybind11_add_module(tinympcgen ${EXT_SOURCES})
target_include_directories(tinympcgen PRIVATE 
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_SOURCE_DIR}/tinympc
)
target_link_libraries(tinympcgen PRIVATE tinympcstatic)

# Set output directories
set_target_properties(tinympcgen PROPERTIES 
    LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
)