cmake_minimum_required(VERSION 2.8.3)

project(roscpp)

if(NOT WIN32)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Wextra")
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rmw_implementation_cmake REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rostime REQUIRED)
find_package(rosconsole REQUIRED)

include_directories(include)

macro(targets)
  if(NOT "${target_suffix} " STREQUAL " ")
    get_rclcpp_information("${rmw_implementation}" "rclcpp${target_suffix}")
  endif()

  add_library(roscpp${target_suffix} SHARED
    src/roscpp.cpp
    src/wall_timer.cpp)
  ament_target_dependencies(roscpp${target_suffix}
    "rclcpp${target_suffix}"
    "rostime"
    "rosconsole")

  add_executable(ros1_talker${target_suffix} ros1_tutorials/talker.cpp)
  ament_target_dependencies(ros1_talker${target_suffix}
    "rclcpp${target_suffix}"
    "std_msgs")
  target_link_libraries(ros1_talker${target_suffix} roscpp${target_suffix})

  add_executable(ros1_listener${target_suffix} ros1_tutorials/listener.cpp)
  ament_target_dependencies(ros1_listener${target_suffix}
    "rclcpp${target_suffix}"
    "std_msgs")
  target_link_libraries(ros1_listener${target_suffix} roscpp${target_suffix})

  install(TARGETS
    roscpp${target_suffix}
    ros1_talker${target_suffix}
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin)

  ament_export_libraries(roscpp${target_suffix})
endmacro()

install(
  DIRECTORY include/
  DESTINATION include
)
install(
  DIRECTORY cmake
  DESTINATION share/${PROJECT_NAME}
)

call_for_each_rmw_implementation(targets GENERATE_DEFAULT)

ament_export_dependencies(rclcpp)
ament_export_dependencies(std_msgs)
ament_export_dependencies(rostime)
ament_export_dependencies(rosconsole)
ament_export_include_directories(include)

if(AMENT_ENABLE_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package(
  CONFIG_EXTRAS "roscpp-extras.cmake"
)

