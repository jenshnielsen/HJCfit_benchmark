CMAKE_MINIMUM_REQUIRED(VERSION 2.6.4 FATAL_ERROR)

PROJECT(HJCfit_benchmark)

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

if (EXISTS $ENV{VIRTUAL_ENV})
  set(CMAKE_INSTALL_PREFIX "$ENV{VIRTUAL_ENV}" CACHE string "" FORCE)
elseif (EXISTS $ENV{CONDA_ENV_PATH})
    set(CMAKE_INSTALL_PREFIX "$ENV{CONDA_ENV_PATH}" CACHE string "" FORCE)
endif()

add_executable(log10long log10long.cc)
include_directories(${CMAKE_INSTALL_PREFIX}/include/dcprogs)
find_library(liblikelihoodpath NAMES likelihood ${CMAKE_INSTALL_PREFIX}/lib)
target_link_libraries(log10long ${liblikelihoodpath})
