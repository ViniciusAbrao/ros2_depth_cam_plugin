cmake_minimum_required(VERSION 3.5)
project(my_plugins)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  # we dont use add_compile_options with pedantic in message packages
  # because the Python C extensions dont comply with it
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wpedantic -Wno-inconsistent-missing-override")
endif()

if(WIN32)
  add_compile_definitions(
    # For math constants
    _USE_MATH_DEFINES
    # Minimize Windows namespace collision
    NOMINMAX
    WIN32_LEAN_AND_MEAN
  )
endif()

find_package(ament_cmake REQUIRED)
find_package(camera_info_manager REQUIRED)
find_package(gazebo_dev REQUIRED)
find_package(gazebo_msgs REQUIRED)
find_package(gazebo_ros REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(image_transport REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(std_srvs REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(trajectory_msgs REQUIRED)

link_directories(${gazebo_dev_LIBRARY_DIRS})

option(ENABLE_PROFILER "Enable Ignition Profiler" FALSE)
if(ENABLE_PROFILER)
  find_package(ignition-common3 QUIET COMPONENTS profiler)
  if(ignition-common3_FOUND)
    add_definitions("-DIGN_PROFILER_ENABLE=1" "-DIGN_PROFILER_REMOTERY=1")
    message(STATUS "Profiler is active")
  else()
    message(WARNING "Can't find Ignition common3. Profiler will not be actived")
    add_definitions("-DIGN_PROFILER_ENABLE=0" "-DIGN_PROFILER_REMOTERY=0")
  endif()
endif()

# multi_camera_plugin
add_library(multi_camera_plugin SHARED
  src/multi_camera_plugin.cpp
)
target_include_directories(multi_camera_plugin PUBLIC include)
ament_target_dependencies(multi_camera_plugin
  "gazebo_dev"
)
target_link_libraries(multi_camera_plugin
  gazebo_sensors
)
target_compile_definitions(multi_camera_plugin
  PRIVATE BUILDING_DLL
)

# gazebo_ros_camera
add_library(gazebo_ros_camera SHARED
  src/gazebo_ros_camera.cpp
)
target_include_directories(gazebo_ros_camera PUBLIC include)
ament_target_dependencies(gazebo_ros_camera
  "camera_info_manager"
  "gazebo_dev"
  "gazebo_ros"
  "image_transport"
  "rclcpp"
  "sensor_msgs"
  "std_msgs"
)
target_link_libraries(gazebo_ros_camera
  CameraPlugin
  DepthCameraPlugin
  multi_camera_plugin
)
ament_export_libraries(gazebo_ros_camera)
if(ENABLE_PROFILER)
  target_include_directories(gazebo_ros_camera PUBLIC ${ignition-common3_INCLUDE_DIRS})
  target_link_libraries(gazebo_ros_camera ${ignition-common3_LIBRARIES})
endif()


ament_package()

install(DIRECTORY include/
        DESTINATION include)

install(TARGETS
    gazebo_ros_camera
    multi_camera_plugin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin)
