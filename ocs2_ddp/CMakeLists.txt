cmake_minimum_required(VERSION 3.18)
project(ocs2_ddp)

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
    ocs2_oc
    ocs2_mpc
    ocs2_qp_solver
)
foreach (dep IN ITEMS ${ros_dependencies})
  find_package(${dep} REQUIRED)
endforeach ()

## add targets
add_library(${PROJECT_NAME}
    src/riccati_equations/ContinuousTimeRiccatiEquations.cpp
    src/riccati_equations/DiscreteTimeRiccatiEquations.cpp
    src/riccati_equations/RiccatiModification.cpp
    src/search_strategy/LevenbergMarquardtStrategy.cpp
    src/search_strategy/LineSearchStrategy.cpp
    src/search_strategy/StrategySettings.cpp
    src/ContinuousTimeLqr.cpp
    src/GaussNewtonDDP.cpp
    src/HessianCorrection.cpp
    src/ILQR.cpp
    src/SLQ.cpp
    src/DDP_Settings.cpp
    src/DDP_HelperFunctions.cpp)
ament_target_dependencies(${PROJECT_NAME} PUBLIC ${ros_dependencies})
target_include_directories(${PROJECT_NAME} PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/test/include>"
    "$<INSTALL_INTERFACE:include/${PROJECT_NAME}>"
    PRIVATE src)

add_executable(${PROJECT_NAME}_lintTarget
    src/lintTarget.cpp)
target_link_libraries(${PROJECT_NAME}_lintTarget PUBLIC ${PROJECT_NAME})

## TODO (rxdu): clang tooling
#find_package(cmake_clang_tools QUIET)
#if (cmake_clang_tools_FOUND)
#  message(STATUS "Run clang tooling for target " ${PROJECT_NAME}_lintTarget)
#  add_clang_tooling(
#      TARGETS ${PROJECT_NAME}_lintTarget ${PROJECT_NAME}
#      SOURCE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/src ${CMAKE_CURRENT_SOURCE_DIR}/include ${CMAKE_CURRENT_SOURCE_DIR}/test
#      CT_HEADER_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include
#      CF_WERROR
#  )
#endif (cmake_clang_tools_FOUND)

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

  ament_add_gtest(exp0_ddp_test
      test/Exp0Test.cpp)
  target_link_libraries(exp0_ddp_test ${PROJECT_NAME})

  ament_add_gtest(exp1_ddp_test
      test/Exp1Test.cpp)
  target_link_libraries(exp1_ddp_test ${PROJECT_NAME})

  ament_add_gtest(correctness_test
      test/CorrectnessTest.cpp)
  target_link_libraries(correctness_test ${PROJECT_NAME})

  ament_add_gtest(riccati_ode_test
      test/RiccatiTest.cpp)
  target_link_libraries(riccati_ode_test ${PROJECT_NAME})

  ament_add_gtest(circular_kinematics_ddp_test
      test/CircularKinematicsTest.cpp)
  target_link_libraries(circular_kinematics_ddp_test ${PROJECT_NAME})

  ament_add_gtest(hybrid_slq_test
      test/HybridSlqTest.cpp)
  target_link_libraries(hybrid_slq_test ${PROJECT_NAME})

  ament_add_gtest(bouncing_mass_test
      test/bouncingmass/BouncingMassTest.cpp
      test/bouncingmass/OverallReference.cpp
      test/bouncingmass/Reference.cpp)
  target_link_libraries(bouncing_mass_test ${PROJECT_NAME})

  ament_add_gtest(testContinuousTimeLqr
      test/testContinuousTimeLqr.cpp)
  target_link_libraries(testContinuousTimeLqr ${PROJECT_NAME})

  ament_add_gtest(testDdpHelperFunction
      test/testDdpHelperFunction.cpp)
  target_link_libraries(testDdpHelperFunction ${PROJECT_NAME})

  ament_add_gtest(testReachingTask
      test/testReachingTask.cpp)
  target_link_libraries(testReachingTask ${PROJECT_NAME})
endif ()

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(${sys_dependencies} ${ros_dependencies})

ament_package()