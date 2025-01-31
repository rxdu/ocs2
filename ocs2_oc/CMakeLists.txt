cmake_minimum_required(VERSION 3.18)
project(ocs2_oc)

if (CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif ()

## generate symbols for IDE indexer
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

## find dependencies
find_package(Boost REQUIRED COMPONENTS system filesystem log_setup log)

# ros dependencies
find_package(ament_cmake REQUIRED)
set(ros_dependencies
    ocs2_core
)
foreach (dep IN ITEMS ${ros_dependencies})
  find_package(${dep} REQUIRED)
endforeach ()

## add targets
add_library(${PROJECT_NAME}
    src/approximate_model/ChangeOfInputVariables.cpp
    src/approximate_model/LinearQuadraticApproximator.cpp
    src/multiple_shooting/Helpers.cpp
    src/multiple_shooting/Initialization.cpp
    src/multiple_shooting/LagrangianEvaluation.cpp
    src/multiple_shooting/MetricsComputation.cpp
    src/multiple_shooting/PerformanceIndexComputation.cpp
    src/multiple_shooting/ProjectionMultiplierCoefficients.cpp
    src/multiple_shooting/Transcription.cpp
    src/oc_data/LoopshapingPrimalSolution.cpp
    src/oc_data/PerformanceIndex.cpp
    src/oc_data/TimeDiscretization.cpp
    src/oc_problem/OptimalControlProblem.cpp
    src/oc_problem/LoopshapingOptimalControlProblem.cpp
    src/oc_problem/OptimalControlProblemHelperFunction.cpp
    src/oc_problem/OcpSize.cpp
    src/oc_problem/OcpToKkt.cpp
    src/oc_solver/SolverBase.cpp
    src/precondition/Ruzi.cpp
    src/rollout/PerformanceIndicesRollout.cpp
    src/rollout/RolloutBase.cpp
    src/rollout/RootFinder.cpp
    src/rollout/InitializerRollout.cpp
    src/rollout/StateTriggeredRollout.cpp
    src/rollout/TimeTriggeredRollout.cpp
    src/rollout/RolloutSettings.cpp
    src/synchronized_module/ReferenceManager.cpp
    src/synchronized_module/LoopshapingReferenceManager.cpp
    src/synchronized_module/LoopshapingSynchronizedModule.cpp
    src/synchronized_module/SolverObserver.cpp
    src/search_strategy/FilterLinesearch.cpp
    src/trajectory_adjustment/TrajectorySpreading.cpp)
target_compile_options(${PROJECT_NAME} PUBLIC ${OCS2_CXX_FLAGS})
target_link_libraries(${PROJECT_NAME} PUBLIC ${Boost_LIBRARIES})
ament_target_dependencies(${PROJECT_NAME} PUBLIC ${ros_dependencies})
target_include_directories(${PROJECT_NAME} PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/test/include>"
    "$<INSTALL_INTERFACE:include/${PROJECT_NAME}>" PRIVATE src)

add_executable(${PROJECT_NAME}_lintTarget
    src/lintTarget.cpp)
target_link_libraries(${PROJECT_NAME}_lintTarget PUBLIC
    ${PROJECT_NAME})
target_include_directories(${PROJECT_NAME}_lintTarget PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:include/${PROJECT_NAME}>"
    PRIVATE src)

## TODO (rxdu): clang tooling
#find_package(cmake_clang_tools QUIET)
#if(cmake_clang_tools_FOUND)
#  message(STATUS "Run clang tooling for target " ${PROJECT_NAME}_lintTarget)
#  add_clang_tooling(
#    TARGETS ${PROJECT_NAME}_lintTarget
#    SOURCE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/src ${CMAKE_CURRENT_SOURCE_DIR}/include ${CMAKE_CURRENT_SOURCE_DIR}/test/include
#    CT_HEADER_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include
#    CF_WERROR
#)
#endif(cmake_clang_tools_FOUND)

## install and export configuration
install(TARGETS ${PROJECT_NAME}
    EXPORT export_${PROJECT_NAME}
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin)

install(DIRECTORY include/
    DESTINATION include/${PROJECT_NAME})

install(DIRECTORY test/include/
    DESTINATION include/${PROJECT_NAME})

## testing
if (BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
  find_package(ament_cmake_gtest REQUIRED)

  ament_add_gtest(test_${PROJECT_NAME}_multiple_shooting
      test/multiple_shooting/testProjectionMultiplierCoefficients.cpp
      test/multiple_shooting/testTranscriptionMetrics.cpp
      test/multiple_shooting/testTranscriptionPerformanceIndex.cpp)
  target_link_libraries(test_${PROJECT_NAME}_multiple_shooting ${PROJECT_NAME})

  ament_add_gtest(test_${PROJECT_NAME}_data
      test/oc_data/testTimeDiscretization.cpp)
  target_link_libraries(test_${PROJECT_NAME}_data ${PROJECT_NAME})

  ament_add_gtest(test_${PROJECT_NAME}_rollout
      test/rollout/testTimeTriggeredRollout.cpp
      test/rollout/testStateTriggeredRollout.cpp)
  target_link_libraries(test_${PROJECT_NAME}_rollout ${PROJECT_NAME})

  ament_add_gtest(test_change_of_variables
      test/testChangeOfInputVariables.cpp)
  target_link_libraries(test_change_of_variables ${PROJECT_NAME})

  ament_add_gtest(test_trajectory_spreading
      test/trajectory_adjustment/TrajectorySpreadingTest.cpp)
  target_link_libraries(test_trajectory_spreading ${PROJECT_NAME})

  ament_add_gtest(test_ocp_to_kkt
      test/oc_problem/testOcpToKkt.cpp)
  target_link_libraries(test_ocp_to_kkt ${PROJECT_NAME})

  ament_add_gtest(test_precondition
      test/precondition/testPrecondition.cpp)
  target_link_libraries(test_precondition ${PROJECT_NAME})
endif ()

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(Boost ${ros_dependencies})

ament_package()
