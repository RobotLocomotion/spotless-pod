cmake_minimum_required(VERSION 3.5)
project(spotless)

if(CMAKE_VERSION VERSION_LESS 3.7)
  set(CMAKE_MODULE_PATH
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules/3.7"
    ${CMAKE_MODULE_PATH})
endif()

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release CACHE STRING
    "Choose the type of build; options are Debug Release RelWithDebInfo MinSizeRel"
    FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY
    STRINGS
      Debug
      Release
      RelWithDebInfo
      MinSizeRel)
endif()

include(CTest)

set(MATLAB_ADDITIONAL_VERSIONS "R2017a=9.2" "R2016b=9.1")
find_package(Matlab MODULE REQUIRED
  COMPONENTS MAIN_PROGRAM MEX_COMPILER MX_LIBRARY)

set(SPOTLESS_MEX_DIR "${CMAKE_CURRENT_SOURCE_DIR}/spotless/mex")

matlab_add_mex(NAME spot_gset SRC
  "${SPOTLESS_MEX_DIR}/spot_gset.c")

matlab_add_mex(NAME spot_mex_msspoly_check_canonical SRC
  "${SPOTLESS_MEX_DIR}/spot_mex_msspoly_check_canonical.cpp")

matlab_add_mex(NAME spot_mex_msspoly_make_canonical_combine_coeffs SRC
  "${SPOTLESS_MEX_DIR}/spot_mex_msspoly_make_canonical_combine_coeffs.cpp"
  "${SPOTLESS_MEX_DIR}/spot_mex_helpers.cpp")

matlab_add_mex(NAME spot_mex_msspoly_make_canonical_combine_powers SRC
  "${SPOTLESS_MEX_DIR}/spot_mex_msspoly_make_canonical_combine_powers.cpp"
  "${SPOTLESS_MEX_DIR}/spot_mex_helpers.cpp")

set_target_properties(
  spot_gset
  spot_mex_msspoly_check_canonical
  spot_mex_msspoly_make_canonical_combine_coeffs
  spot_mex_msspoly_make_canonical_combine_powers
  PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY "${SPOTLESS_MEX_DIR}"
    LIBRARY_OUTPUT_DIRECTORY "${SPOTLESS_MEX_DIR}"
    RUNTIME_OUTPUT_DIRECTORY "${SPOTLESS_MEX_DIR}"
    ARCHIVE_OUTPUT_DIRECTORY_DEBUG "${SPOTLESS_MEX_DIR}"
    LIBRARY_OUTPUT_DIRECTORY_DEBUG "${SPOTLESS_MEX_DIR}"
    RUNTIME_OUTPUT_DIRECTORY_DEBUG "${SPOTLESS_MEX_DIR}"
    ARCHIVE_OUTPUT_DIRECTORY_MINSIZEREL "${SPOTLESS_MEX_DIR}"
    LIBRARY_OUTPUT_DIRECTORY_MINSIZEREL "${SPOTLESS_MEX_DIR}"
    RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL "${SPOTLESS_MEX_DIR}"
    ARCHIVE_OUTPUT_DIRECTORY_RELEASE "${SPOTLESS_MEX_DIR}"
    LIBRARY_OUTPUT_DIRECTORY_RELEASE "${SPOTLESS_MEX_DIR}"
    RUNTIME_OUTPUT_DIRECTORY_RELEASE "${SPOTLESS_MEX_DIR}"
    ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO "${SPOTLESS_MEX_DIR}"
    LIBRARY_OUTPUT_DIRECTORY_RELWITHDEBINFO "${SPOTLESS_MEX_DIR}"
    RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO "${SPOTLESS_MEX_DIR}")

if(BUILD_TESTING)
  set(SPOTLESS_ADDITIONAL_PATH
    "${CMAKE_CURRENT_SOURCE_DIR}/spotless"
    "${CMAKE_CURRENT_SOURCE_DIR}/spotless/mex"
    "${CMAKE_CURRENT_SOURCE_DIR}/spotless/mss"
    "${CMAKE_CURRENT_SOURCE_DIR}/spotless/spotopt"
    "${CMAKE_CURRENT_SOURCE_DIR}/spotless/spotopt/solvers"
    "${CMAKE_CURRENT_SOURCE_DIR}/spotless/spotopt/util"
    "${CMAKE_CURRENT_SOURCE_DIR}/spotless/util")

  matlab_add_unit_test(NAME spot_mex_test
    UNITTEST_FILE "${SPOTLESS_MEX_DIR}/spot_mex_test.m"
    TIMEOUT -1
    ADDITIONAL_PATH "${SPOTLESS_ADDITIONAL_PATH}"
    MATLAB_ADDITIONAL_STARTUP_OPTIONS -nojvm)

  set(SPOTLESS_TESTS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/spotless/tests")

  matlab_add_unit_test(NAME msspoly_tests
    UNITTEST_FILE "${SPOTLESS_TESTS_DIR}/msspoly_tests.m"
    TIMEOUT -1
    ADDITIONAL_PATH "${SPOTLESS_ADDITIONAL_PATH}"
    MATLAB_ADDITIONAL_STARTUP_OPTIONS -nojvm)

  option(HAVE_SEDUMI "SeDuMi is installed and in the MATLAB search path" OFF)

  if(HAVE_SEDUMI)
    matlab_add_unit_test(NAME mssprog_tests
      UNITTEST_FILE "${SPOTLESS_TESTS_DIR}/mssprog_tests.m"
      TIMEOUT -1
      ADDITIONAL_PATH "${SPOTLESS_ADDITIONAL_PATH}"
      MATLAB_ADDITIONAL_STARTUP_OPTIONS -nojvm)
  endif()
endif()

configure_file(addpath_spotless.m.in addpath_spotless.m @ONLY)
configure_file(rmpath_spotless.m.in rmpath_spotless.m @ONLY)

install(FILES
  "${CMAKE_CURRENT_BINARY_DIR}/addpath_spotless.m"
  "${CMAKE_CURRENT_BINARY_DIR}/rmpath_spotless.m"
  DESTINATION matlab)
