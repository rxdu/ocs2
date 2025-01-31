cmake_minimum_required(VERSION 3.18)
project(ocs2_ipm)

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
    ocs2_core
    ocs2_mpc
    ocs2_oc
    blasfeo_colcon
    hpipm_colcon)
foreach (dep IN ITEMS ${ros_dependencies})
  find_package(${dep} REQUIRED)
endforeach ()

## add targets
# Multiple shooting solver library
add_library(${PROJECT_NAME}
    src/IpmHelpers.cpp
    src/IpmInitialization.cpp
    src/IpmPerformanceIndexComputation.cpp
    src/IpmSettings.cpp
    src/IpmSolver.cpp)
target_link_libraries(${PROJECT_NAME} PUBLIC ${Boost_LIBRARIES})
ament_target_dependencies(${PROJECT_NAME} PUBLIC ${ros_dependencies})
target_include_directories(${PROJECT_NAME} PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:include/${PROJECT_NAME}>"
    PRIVATE src)

## TODO (rxdu): clang tooling
#find_package(cmake_clang_tools QUIET)
#if(cmake_clang_tools_FOUND)
#  message(STATUS "Run clang tooling for target " ${PROJECT_NAME})
#  add_clang_tooling(
#    TARGETS ${PROJECT_NAME}
#    SOURCE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/src ${CMAKE_CURRENT_SOURCE_DIR}/include ${CMAKE_CURRENT_SOURCE_DIR}/test
#    CT_HEADER_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include
#    CF_WERROR
#  )
#endif(cmake_clang_tools_FOUND)

## install and export configuration
install(TARGETS ${PROJECT_NAME}
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
      test/Exp0Test.cpp
      test/Exp1Test.cpp
      test/testCircularKinematics.cpp
      test/testSwitchedProblem.cpp
      test/testUnconstrained.cpp
      test/testValuefunction.cpp  )
  target_link_libraries(test_${PROJECT_NAME} ${PROJECT_NAME})
endif ()

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(${sys_dependencies} ${ros_dependencies})

ament_package()