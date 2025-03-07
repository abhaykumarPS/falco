#
# Copyright (C) 2019 The Falco Authors.
#
#
# Licensed under the Apache License, Version 2.0 (the "License"); you may not
# use this file except in compliance with the License. You may obtain a copy of
# the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
# License for the specific language governing permissions and limitations under
# the License.
#
if(MINIMAL_BUILD)
  set(
    FALCO_TESTS_SOURCES
    test_base.cpp
    engine/test_token_bucket.cpp
    engine/test_rulesets.cpp
    engine/test_falco_utils.cpp
    falco/test_configuration.cpp
  )
else()
  set(
    FALCO_TESTS_SOURCES
    test_base.cpp
    engine/test_token_bucket.cpp
    engine/test_rulesets.cpp
    engine/test_falco_utils.cpp
    falco/test_configuration.cpp
    falco/test_webserver.cpp
  )
endif()

set(FALCO_TESTED_LIBRARIES falco_engine ${YAMLCPP_LIB})

SET(FALCO_TESTS_ARGUMENTS "" CACHE STRING "Test arguments to pass to the Falco test suite")

option(FALCO_BUILD_TESTS "Determines whether to build tests." ON)

if(FALCO_BUILD_TESTS)
  enable_testing()
  if(NOT TARGET catch)
    include(DownloadCatch)
  endif()

  if(NOT TARGET fakeit)
    include(DownloadFakeIt)
  endif()

  add_executable(falco_test ${FALCO_TESTS_SOURCES})

  target_link_libraries(falco_test PUBLIC ${FALCO_TESTED_LIBRARIES})

  if(MINIMAL_BUILD)
    target_include_directories(
      falco_test
      PUBLIC "${CATCH2_INCLUDE}"
            "${FAKEIT_INCLUDE}"
            "${PROJECT_SOURCE_DIR}/userspace/engine"
            "${PROJECT_BINARY_DIR}/userspace/falco"
            "${YAMLCPP_INCLUDE_DIR}"
            "${PROJECT_SOURCE_DIR}/userspace/falco")
  else()
    target_include_directories(
      falco_test
      PUBLIC "${CATCH2_INCLUDE}"
            "${FAKEIT_INCLUDE}"
            "${PROJECT_SOURCE_DIR}/userspace/engine"
            "${PROJECT_BINARY_DIR}/userspace/falco"
            "${YAMLCPP_INCLUDE_DIR}"
            "${CIVETWEB_INCLUDE_DIR}"
            "${PROJECT_SOURCE_DIR}/userspace/falco")
  endif()
  add_dependencies(falco_test catch2)

  include(CMakeParseArguments)
  include(CTest)
  include(Catch)
  catch_discover_tests(falco_test)
  separate_arguments(FALCO_TESTS_ARGUMENTS)
  add_custom_target(tests COMMAND ${CMAKE_CTEST_COMMAND} ${FALCO_TESTS_ARGUMENTS} DEPENDS falco_test)
endif()
