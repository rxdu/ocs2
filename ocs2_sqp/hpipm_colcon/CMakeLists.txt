cmake_minimum_required(VERSION 3.18)
project(hpipm_colcon)

if (CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif ()

## generate symbols for IDE indexer
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

## find dependencies
# system dependencies
find_package(Boost REQUIRED COMPONENTS system filesystem log_setup log)

# ros dependencies
find_package(ament_cmake REQUIRED)
set(ros_dependencies
    ocs2_qp_solver
    blasfeo_colcon
)
foreach (dep IN ITEMS ${ros_dependencies})
  find_package(${dep} REQUIRED)
endforeach ()

## add targets
include(FetchContent)

# Define directories
set(HPIPM_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX} CACHE STRING "HPIPM install path")
set(HPIPM_INCLUDE_DIR ${HPIPM_INSTALL_PREFIX}/include)
set(HPIPM_LIB_DIR ${HPIPM_INSTALL_PREFIX}/lib)
set(HPIPM_DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}/download)
set(HPIPM_BUILD_DIR ${CMAKE_CURRENT_BINARY_DIR}/build)

# Create directories if they do not exist
file(MAKE_DIRECTORY ${HPIPM_INCLUDE_DIR})
file(MAKE_DIRECTORY ${HPIPM_LIB_DIR})
file(MAKE_DIRECTORY ${HPIPM_DOWNLOAD_DIR})
file(MAKE_DIRECTORY ${HPIPM_BUILD_DIR})

message(STATUS "BLASFEO_PATH: " ${HPIPM_INCLUDE_DIR})

# HPIPM Settings
set(BLASFEO_PATH ${BLASFEO_PATH} CACHE STRING "BLASFEO installation path" FORCE)
set(BLASFEO_INCLUDE_DIR ${BLASFEO_PATH}/include CACHE STRING "Path to BLASFEO header files." FORCE)
set(BUILD_SHARED_LIBS ON CACHE STRING "Build shared libraries" FORCE)
set(HPIPM_TESTING OFF CACHE BOOL "Examples enabled")

if (CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
  message(STATUS "Building hpipm for Intel x86_64 architecture")
  set(TARGET "AVX")
elseif (CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64")
  message(STATUS "Building hpipm for ARM64 architecture")
  set(TARGET "GENERIC")
else ()
  message(STATUS "Building hpipm for unknown architecture")
  set(TARGET "GENERIC")
endif ()

# Download & build source
FetchContent_Declare(hpipmDownload
    GIT_REPOSITORY https://github.com/giaf/hpipm
    GIT_TAG 255ffdf38d3a5e2c3285b29568ce65ae286e5faf
    UPDATE_COMMAND ""
    SOURCE_DIR ${HPIPM_DOWNLOAD_DIR}
    BINARY_DIR ${HPIPM_BUILD_DIR}
    BUILD_COMMAND "$(MAKE) TARGET=${TARGET}"
    INSTALL_COMMAND "$(MAKE) install_shared")
FetchContent_MakeAvailable(hpipmDownload)

# Copy header to where ament_cmake expects them
file(GLOB_RECURSE HEADERS "${HPIPM_DOWNLOAD_DIR}/include/*.h")
foreach (HEADER_FILE ${HEADERS})
  message(STATUS "FOUND HEADER: " ${HEADER_FILE})
  file(COPY ${HEADER_FILE} DESTINATION ${HPIPM_INCLUDE_DIR})
endforeach ()

# Install the library where catkin expects them
#set_target_properties(hpipm PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${HPIPM_LIB_DIR})

# Hpipm interface
add_library(${PROJECT_NAME}
    src/HpipmInterface.cpp
    src/HpipmInterfaceSettings.cpp)
ament_target_dependencies(${PROJECT_NAME} ${ros_dependencies})
target_link_libraries(${PROJECT_NAME} hpipm)
target_include_directories(${PROJECT_NAME} PUBLIC
    ${HPIPM_INCLUDE_DIR}
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:include/${PROJECT_NAME}>")

## TODO (rxdu): clang tooling
#find_package(cmake_clang_tools QUIET)
#if (cmake_clang_tools_FOUND)
#  message(STATUS "Run clang tooling for target " ${PROJECT_NAME})
#  add_clang_tooling(
#      TARGETS
#      ${PROJECT_NAME}
#      SOURCE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/src ${CMAKE_CURRENT_SOURCE_DIR}/include
#      CT_HEADER_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include
#      CF_WERROR
#  )
#endif (cmake_clang_tools_FOUND)

## install and export configuration
install(TARGETS hpipm ${PROJECT_NAME}
    EXPORT export_${PROJECT_NAME}
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin)

install(DIRECTORY include/
    DESTINATION include/${PROJECT_NAME})

## testing
if (BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
  find_package(ament_cmake_gtest REQUIRED)

  ament_add_gtest(test_${PROJECT_NAME}
      test/testHpipmInterface.cpp)
  target_link_libraries(test_${PROJECT_NAME} ${PROJECT_NAME})
endif ()

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(hpipm ${ros_dependencies})

ament_package()
