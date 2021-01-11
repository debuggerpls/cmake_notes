#### CMake notes

* Outdated usage:
```
mkdir build
cd build
cmake ..
make
make test
```

* Basic modern usage:
```
# cd to root of <project>

# creates build dir (if not exist) with the source directory .
cmake -S . -B build

# builds in build dir
cmake --build build

# example: run tests
cmake --build build --target test
```

* Configure options
```
# show available generators
cmake --help

# create buildNinja with clang compilers and ninja generator
CC=clang CXX=clang++ cmake -S . -B buildNinja -GNinja

# build on multiple cores
cmake --build build -j 2

# verbose build
cmake --build build -v
```

* Setting options:
```
# list options with -L
cmake -S . -L
# OR
cmake build -L

# list human readable help with -LH
cmake -S . -LH

# set options with -D
cmake build -D VAR=VAL

# common options:
CMAKE_BUILD_TYPE
CMAKE_INSTALL_PREFIX
BUILD_SHARED_LIBS
BUILD_TESTING

# example: set install dir to local install and install it
cmake build -D CMAKE_INSTALL_PREFIX=install
cmake --install build
```

* Debugging CMake files
```
# see verbose CMake configure output with --trace
cmake build --trace-source="CMakeLists.txt"
```

* CMake simplest CMakeLists.txt
```
# CMakeLists.txt

cmake_minimum_required(VERSION 3.14)

project(MyProject)

add_executable(myexample simple.cpp)

-----------------------------------
# CMakeLists.txt with few arguments

cmake_minimum_required(VERSION 3.14...3.18)

project(MyProject
  VERSION
    1.0
  DESCRIPTION
    "Very nice project"
  LANGUAGES
    CXX
)

add_executable(myexample simple.cpp)
```

* Targets:
```
# Each target is like object that hold information (SOURCE property for example)
# Target names must be unique

# executable targets
add_executable(myexample simple.cpp)

# library targets
add_library(mylibrary simplelib.cpp)

# to libraries, you can add keywords STATIC, SHARED or MODULE
# default is sort of "auto" that is user selectable with BUILD_SHARED_LIBS
```

* Linking
```
# link using function
target_link_libraries(TARGET keyword LIB)

# keyword is one of: PRIVATE, PUBLIC and INTERFACE
# dont forget keyword when making a library!
```

* Include directories
```
When you run 
[target_include_directory(TargetA PRIVATE mydir)][target_include_directory], 
then the INCLUDE_DIRECTORIES property of TargetA has mydir appended. 
If you use the keyword INTERFACE instead, then INTERFACE_INCLUDE_DIRECTORIES 
is appended to, instead. If you use PUBLIC, then both properties are appended 
to at the same time.
```

* Things you can set on targets:
```
* target_link_libraries: Other targets; can also pass library names directly
* target_include_directories: Include directories
* target_compile_features: The compiler features you need activated, like cxx_std_11
* target_compile_definitions: Definitions
* target_compile_options: More general compile flags
* target_link_directories: Don’t use, give full paths instead (CMake 3.13+)
* target_link_options: General link flags (CMake 3.13+)
* target_sources: Add source files
```

* Other types of targets:
```
# header only library
add_library(some_header_only_lib INTERFACE)

# can only set INTERFACE properties on this
target_include_directories(some_header_only_lib INTERFACE inc)


# second situation is pre-build library, uses keyword IMPORTED
# they can also be INTERFACE libraries and have :: in their name.
# (ALIAS libraries, which simple rename some other library, are also
# allowed to have ::). Most of the times IMPORTED libraries you get
# from other places and not make your own.
```

* Scripts
```
# run script named example.cmake
cmake -P example.cmake

-----
# cached variables
# cache.cmake
set(MY_CACHE_VAR "I am a cached variable" CACHE STRING "Description")
message(STATUS "${MY_CACHE_VAR}")

# you can override with -D

----
# Other variables

# Options, very common for bools
option(MY_OPTION "On or off" OFF)


# Environment variables
# You can get environment variables with $ENV{name}. 
# You can check to see if an environment variable is defined 
with if(DEFINED ENV{name}) (notice the missing $).

# Set / Get properties
# properties are variables that are attached to targets
# Use get_property(), set_property() or get_target_properties(), set_target_properties()

# TIP:
# Use
include(CMakePrintHelpers)

# this adds cmake_print_properties() and cmake_print_variables() to save
# yourself typing when debugging

-----
# Globbing
file(GLOB OUTPUT_VAR *.cxx)

# Makes list of all files that match the pattern and save it in OUTPUT_VAR
# Use GLOB_RECURSE to recurse subdirectories. 

# MOST IMPORTANT: CONFIGURE_DEPENDS
# Unless you set it, then after build step (not the configure step)
# your build tool will not check if you added new files. 

#  new rule is “never glob, but if you have to, add CONFIGURE_DEPENDS”

# CMake 3.12+
file(GLOB HEADER_LIST CONFIGURE_DEPENDS "${ModernCMakeExample_SOURCE_DIR}/include/modern/*.hpp")

# older versions
set(HEADER_LIST "${ModernCMakeExample_SOURCE_DIR}/include/modern/lib.hpp")

```

* Source
```
# in cmake 3.11+
add_library(modern_library)
target_sources(modern_library
  PRIVATE
    lib.cpp
  PUBLIC
    ${HEADER_LIST}
)

# otherwise
add_library(modern_library lib.cpp ${HEADER_LIST})
```

* Testing
```
# Testing only available if this is the main app
if(BUILD_TESTING)
  add_subdirectory(tests)
endif()
```

* Find packages
```
# Docs only available if this is the main app
  find_package(Doxygen)
  if(Doxygen_FOUND)
    add_subdirectory(docs)
  else()
    message(STATUS "Doxygen not found, not building docs")
  endif()
```

* Checks for main project or subdirectory
```
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)

  # Testing only available if this is the main app. Note this needs to be done
  # in the main CMakeLists since it calls enable_testing, which must be in the
  # main CMakeLists.
  include(CTest)

endif()
```

* Common problems
```
# 1. Low minimum CMake version
cmake_minimum_required(VERSION 3.0 FATAL_ERROR)


# 2. Building inplace
### Require out-of-source builds
file(TO_CMAKE_PATH "${PROJECT_BINARY_DIR}/CMakeLists.txt" LOC_PATH)
if(EXISTS "${LOC_PATH}")
    message(FATAL_ERROR "You cannot build in a source directory (or any directory with "
                        "CMakeLists.txt file). Please make a build subdirectory. Feel free to "
                        "remove CMakeCache.txt and CMakeFiles.")
endif()


# 3. Picking compiler
# CMake may pick the wrong compiler
# Use CC and CXX  
# MAKE SURE TO PICK IT ON THE FIRST RUN! You cannot just reconfigure it

# 3. Spaces in paths
set(VAR a b v)
# The value of VAR is a list with three elements, or the string "a;b;c" (the two things are exactly the same). 

# So, if you do:
set(MY_DIR "/path/with spaces/")
target_include_directories(target PRIVATE ${MY_DIR})
# that is identical to:
target_include_directories(target PRIVATE /path/with spaces/)

# Correct:
set(MY_DIR "/path/with spaces/")
target_include_directories(target PRIVATE "${MY_DIR}")
```

* Debugging:
```
# Printing variables
message(STATUS "MY_VARIABLE=${MY_VARIABLE}")

# build in module makes this easier:
include(CMakePrintHelpers)
cmake_print_variables(MY_VARIABLE)

# property printing is much nicer. No need to get properties one by one
# of of each target and print them. Instead list them and print them:
cmake_print_properties(
    TARGETS my_target
    PROPERTIES POSITION_INDEPENDENT_CODE
)

# WARNING: cannot print SOURCES, because it conflicts with SOURCES keyword
-----

# Tracing a run
# Trace mode on
cmake -S . -B build --trace-source=CMakeLists.txt

# Expanded
cmake -S . -B build --trace-source=CMakeLists.txt --trace-expand

# Full trace
cmake -S . -B build --trace
-----

# C++ debugging

# run CMAKE_BUILD_TYPE=TYPE , with type one of: Debug, Release (optimized release), 
# RelWithDebInfo (release with some extra debug info), 
# MinSizeRel (minimum size release)

# gdb debug
cmake -S . -B build-debug -DCMAKE_BUILD_TYPE=Debug
cmake --build build-debug
gdb build-debug/simple_example
```

* Find libraries
```
# Does -lm work? (notice this is find_library, not find_package) Saves the
# location of the m library in a variable named MATH_LIBRARY
find_library(MATH_LIBRARY m)

# If there is a -lm, let's use it
if(MATH_LIBRARY)
  target_link_libraries(simple_lib PUBLIC ${MATH_LIBRARY})
endif()

```

* Note about build type
```
# CMake defaults to an "empty" build type, which is neither optimized nor
# debug. You can fix this manually or always specify a build type.

# Setting it manually
set(default_build_type "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()
```

* Common utility needs:
```
CMAKE_CXX_COMPILER_LAUNCHER - can set up a compiler launcher, like ccache, to speed up your builds.
CMAKE_CXX_CLANG_TIDY - can run clang-tidy to help you clean up your code.
CMAKE_CXX_CPPCHECK - for cppcheck.
CMAKE_CXX_CPPLINT - for cpplint.
CMAKE_CXX_INCLUDE_WHAT_YOU_USE - for iwyu.
```

* Finding packages
```
# CMake searches for packages in two ways. Both use the same interface:
find_package(MyPackage 1.2)

# This will look for a file in the CMAKE_MODULE_PATH that is named FindMyPackage.cmake
# If it does not find one, it will look for a file named MyPackageConfig.cmake in several places,
# including MyPackage_DIR if that variable exists. 
# You can only perform one of these searches with MODULE or CONFIG, respectively.

# You can add COMPONENTS (if package supports it), QUIT(to hide extra text),
# or REQUIRED(to cause a missing package to fail the configure step)

-----

# Environment Hints
# hint where the software package is installed outside of a system paths.
# in 3.12+, individual packages location can be hinted by setting their installation
# root path in <PackageName>_ROOT (bash)
export HDF5_ROOT=$HOME/software/hdf5-1.12.0

# CMAKE_PREFIX_PATH can be used to hint a list of installation root paths at once:
export CMAKE_PREFIX_PATH=$HOME/software/hdf5-1.12.0:$HOME/software/boost-1.74.0:$CMAKE_PREFIX_PATH
-----

# FindPackage
# The older method for finding packages is the FindPackage.cmake method (MODULE)
# This is a CMake or user supplied search script that knows how to look for a package.
# A package should at least set the variable Package_FOUND.
# There are 100 or so find packages included in CMake, refer to the documentation for each.

-----

# PackageConfig
# The “better” way to do things is to have an installed package provide its own details to CMake
# these are “CONFIG” files and come with many packages. 

# To be clear: If you are a package author, never supply a Find<package>.cmake, 
# but instead always supply a <package>Config.cmake with all your builds. 
# If you are depending on another package, try to look for a Config first, 
# and if that is not available, or often not available, 
# then write a find package for it for your use.
```

* Functions:
```
function(EXAMPLE_FUNCTION AN_ARGUMENT)
    set(${AN_ARGUMENT}_LOCAL "I'm in the local scope")
    set(${AN_ARGUMENT}_PARENT "I'm in the parent scope" PARENT_SCOPE)
endfunction()

#example_function() # Error
example_function(ONE)
example_function(TWO THREE) # Not error

message(STATUS "${ONE_LOCAL}") # empty
message(STATUS "${ONE_PARENT}") # prints

# ANOTHER example
function(SIMPLE REQUIRED_ARG)
    message(STATUS "Simple arguments: ${REQUIRED_ARG}, followed by ${ARGV}")
    set(${REQUIRED_ARG} "From SIMPLE" PARENT_SCOPE)
endfunction()

simple(This)
message("Output: ${This}")

# Prints:
-- Simple arguments: This, followed by This
Output: From SIMPLE

# You can specify required positional arguments after the name; 
# all other arguments are set in ARGN; ARGV holds all arguments, 
# even the listed positional ones. Since you name variables with strings,
# you can set variables using names. This is enough to recreate any of 
# the CMake commands.
-----

# Parsing arguments
# This handling is standardized in the cmake_parse_arguments() command. 
# Here’s how it works: first lists have no args(SINGLE;ANOTHER), 
# second list(ONE_VALUE;..) have one arg, third list have multiple arguments

function(COMPLEX)
    cmake_parse_arguments(
        COMPLEX_PREFIX
        "SINGLE;ANOTHER"
        "ONE_VALUE;ALSO_ONE_VALUE"
        "MULTI_VALUES"
        ${ARGN}
    )
endfunction()

complex(SINGLE ONE_VALUE value MULTI_VALUES some other values)
# Inside the function after this call:
COMPLEX_PREFIX_SINGLE = TRUE
COMPLEX_PREFIX_ANOTHER = FALSE
COMPLEX_PREFIX_ONE_VALUE = "value"
COMPLEX_PREFIX_ALSO_ONE_VALUE = <UNDEFINED>
COMPLEX_PREFIX_MULTI_VALUES = "some;other;values"


# OR
function(COMPLEX required_arg_1)
    cmake_parse_arguments(
        PARSE_ARGV
        1
        COMPLEX_PREFIX
        "SINGLE;ANOTHER"
        "ONE_VALUE;ALSO_ONE_VALUE"
        "MULTI_VALUES"
    )
endfunction()

complex(SINGLE ONE_VALUE value MULTI_VALUES some other values)

# After call, in function you will find:
COMPLEX_PREFIX_SINGLE = TRUE
COMPLEX_PREFIX_ANOTHER = FALSE
COMPLEX_PREFIX_ONE_VALUE = "value"
COMPLEX_PREFIX_ALSO_ONE_VALUE = <UNDEFINED>
COMPLEX_PREFIX_MULTI_VALUES = "some;other;values"
COMPLEX_PREFIX_UNPARSED_ARGUMENTS = <UNDEFINED>
```
