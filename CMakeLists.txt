CMAKE_MINIMUM_REQUIRED(VERSION 2.6.4 FATAL_ERROR)

PROJECT(HJCfit_benchmark)

option(openmp         "Enable OpenMP parallelization."          off)
# Set a default build type to Release if none was specified
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "" FORCE)
endif()

# Enable ExternalProject CMake module
INCLUDE(ExternalProject)
# Set default ExternalProject root directory
set(EXTERNAL_ROOT ${CMAKE_BINARY_DIR}/external)
set(CMAKE_SCRIPTS ${PROJECT_SOURCE_DIR}/cmake_modules)
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

# Downloads and installs GreatCMakeCookOff
# It contains a number of cmake recipes
include(LookUp-GreatCMakeCookOff)
include(PackageLookup)

# Install Eigen the CookOff way
lookup_package(Eigen3 REQUIRED)

# Adds flags to compiler to launch it into c++11 land
include(AddCPP11Flags)

# The following will print out all available features.
include(CheckCXX11Features)

# Check features (before REQUIRED they are optional)
cxx11_feature_check(constexpr unique_ptr random_device constructor_delegate
                    trivial_type_traits noexcept
                    REQUIRED auto lambda static_assert rvalue_references decltype
                    enable_if type_traits shared_ptr initialization)
include(CheckIsNaN)

if(EIGEN3_INCLUDE_DIR)
  include_directories(${EIGEN3_INCLUDE_DIR} ${PROJECT_BINARY_DIR})
endif()

if(openmp)
    FIND_PACKAGE(OpenMP REQUIRED)
    if(NOT (${CMAKE_C_FLAGS} MATCHES ${OpenMP_C_FLAGS}))
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}" CACHE STRING "" FORCE)
    endif()
    if(NOT (${CMAKE_CXX_FLAGS} MATCHES ${OpenMP_CXX_FLAGS}))
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}" CACHE STRING "" FORCE)
    endif()
    if(NOT ("${CMAKE_EXE_LINKER_FLAGS}" MATCHES "${OpenMP_EXE_LINKER_FLAGS}"))
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}" CACHE STRING "" FORCE)
    endif()
endif(openmp)

if (EXISTS $ENV{VIRTUAL_ENV})
  set(CMAKE_INSTALL_PREFIX "$ENV{VIRTUAL_ENV}" CACHE string "" FORCE)
elseif (EXISTS $ENV{CONDA_ENV_PATH})
    set(CMAKE_INSTALL_PREFIX "$ENV{CONDA_ENV_PATH}" CACHE string "" FORCE)
endif()

include_directories(${CMAKE_INSTALL_PREFIX}/include/dcprogs)
find_library(liblikelihoodpath NAMES likelihood HINTS ${CMAKE_INSTALL_PREFIX}/lib)
add_executable(log10long0 log10long0.cc)
target_link_libraries(log10long0 ${liblikelihoodpath})
add_executable(log10long1 log10long1.cc)
target_link_libraries(log10long1 ${liblikelihoodpath})
add_executable(log10long0_simple log10long0_simple.cc)
target_link_libraries(log10long0_simple ${liblikelihoodpath})
