cmake_minimum_required(VERSION 3.18)
project(ocs2_mpc)

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
)
foreach (dep IN ITEMS ${ros_dependencies})
  find_package(${dep} REQUIRED)
endforeach ()

## add targets
add_library(${PROJECT_NAME}
    src/LoopshapingSystemObservation.cpp
    src/MPC_BASE.cpp
    src/MPC_Settings.cpp
    src/SystemObservation.cpp
    src/MRT_BASE.cpp
    src/MPC_MRT_Interface.cpp
    # src/MPC_OCS2.cpp
)
ament_target_dependencies(${PROJECT_NAME} PUBLIC ${ros_dependencies})
target_include_directories(${PROJECT_NAME} PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:include/${PROJECT_NAME}>"
    PRIVATE src)

add_executable(${PROJECT_NAME}_lintTarget
    src/lintTarget.cpp)
target_link_libraries(${PROJECT_NAME}_lintTarget PUBLIC
    ${PROJECT_NAME})

## TODO (rxdu): clang tooling
#find_package(cmake_clang_tools QUIET)
#if (cmake_clang_tools_FOUND)
#  message(STATUS "Run clang tooling for target " ${PROJECT_NAME}_lintTarget)
#  add_clang_tooling(
#      TARGETS ${PROJECT_NAME}_lintTarget
#      SOURCE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/src ${CMAKE_CURRENT_SOURCE_DIR}/include
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

## testing
#catkin_add_gtest(testMPC_OCS2
#  test/testMPC_OCS2.cpp
#)
#target_link_libraries(testMPC_OCS2
#  ${catkin_LIBRARIES}
#  ${Boost_LIBRARIES}
#)
#target_compile_options(testMPC_OCS2 PRIVATE ${OCS2_CXX_FLAGS})

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(${ros_dependencies})

ament_package()
