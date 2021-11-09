This repo presents a ROS2 version of the plugin: libgazebo_ros_camera.so 
It fix the issue of pointcloud2 rotated in the rviz.
The gazebo_ros_camera.cpp file is modified to use x-forward and z-up frame for pointcloud2.
The plugin is compiled togheter multi_camera plugin, as required in the CMakeLists.txt.
A simple demo ROS2 package has been included, named my_package, that takes the URDF of a robot model and spawn it in Gazebo with the depth camera. 
In the Rviz2 the pointcloud2 can be in the sequence displayed.

To run the demo:
ros2 launch my_package spawn_camera.launch.py

Remender to change the plugin path, in the simple_camera.urdf file, since the plugin version is not included in the path of 
gazebo_ros_pkgs. Find the .so file path at "/dev_ws/build/my_plugins/libgazebo_ros_camera.so".
