cmake_minimum_required(VERSION 3.8)
project(ethercat_interface)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(ament_cmake_ros REQUIRED)
find_package(rclcpp REQUIRED)

# EtherLab. Keep the system-wide installation as the default while allowing
# CI and development machines to use an unprivileged staging prefix.
set(ETHERLAB_DIR "/usr/local/etherlab" CACHE PATH "IgH EtherCAT installation prefix")
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

find_path(ETHERCAT_INCLUDE_DIR ecrt.h
  HINTS ${ETHERLAB_DIR}/include
  REQUIRED)
find_library(ETHERCAT_LIB ethercat
  HINTS ${ETHERLAB_DIR}/lib
  REQUIRED)

ament_export_include_directories(
  include
  ${ETHERCAT_INCLUDE_DIR}
)

add_library(
  ${PROJECT_NAME}
  SHARED
  src/ec_master.cpp)

target_include_directories(
  ${PROJECT_NAME}
  PRIVATE
  include
  ${ETHERCAT_INCLUDE_DIR}
)

target_link_libraries(${PROJECT_NAME} ${ETHERCAT_LIB})

ament_target_dependencies(
  ${PROJECT_NAME}
  rclcpp
)

# INSTALL
install(
  TARGETS ${PROJECT_NAME}
  DESTINATION lib
)
install(
  DIRECTORY include/
  DESTINATION include
)

if(BUILD_TESTING)
  # find_package(ament_cmake_gtest REQUIRED)
  find_package(ament_lint_auto REQUIRED)
  find_package(yaml_cpp_vendor REQUIRED)
  ament_lint_auto_find_test_dependencies()

  # Test PdoChannelManager
  ament_add_gmock(
    test_ec_pdo_channel_manager
    test/test_ec_pdo_channel_manager.cpp
  )
  target_include_directories(
    test_ec_pdo_channel_manager PRIVATE include ${ETHERCAT_INCLUDE_DIR})
  ament_target_dependencies(test_ec_pdo_channel_manager
    yaml_cpp_vendor
  )
endif()

## EXPORTS
ament_export_include_directories(
  include
)
ament_export_libraries(
  ${PROJECT_NAME}
  ${ETHERCAT_LIB}
)
ament_export_dependencies(
  rclcpp
)
ament_package()
