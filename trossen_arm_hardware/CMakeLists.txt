cmake_minimum_required(VERSION 3.10.0)
project(trossen_arm_hardware)

find_package(ament_cmake REQUIRED)
find_package(hardware_interface REQUIRED)
find_package(libtrossen_arm REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_lifecycle REQUIRED)

include_directories(
  include
)

add_library(${PROJECT_NAME} SHARED
  src/interface.cpp
)

target_link_libraries(${PROJECT_NAME} PUBLIC
  libtrossen_arm
  hardware_interface::hardware_interface
  pluginlib::pluginlib
  rclcpp::rclcpp
  rclcpp_lifecycle::rclcpp_lifecycle)

# libtrossen_arm.a bundles its own urdfdom, tinyxml2 and console_bridge. Hide those symbols so the
# bundled code binds to its own copies rather than the (ABI-incompatible) system libraries already
# loaded by ros2_control_node, which otherwise segfaults in urdf::parseURDF during configure.
target_link_options(${PROJECT_NAME} PRIVATE "LINKER:--exclude-libs,libtrossen_arm.a")

pluginlib_export_plugin_description_file(hardware_interface ${PROJECT_NAME}.xml)

install(
  DIRECTORY
    include/
  DESTINATION
    include
)

install(
  TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()


ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(
  hardware_interface
  pluginlib
  rclcpp
  rclcpp_lifecycle)
ament_export_dependencies(libtrossen_arm)
ament_package()
