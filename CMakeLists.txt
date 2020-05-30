cmake_minimum_required(VERSION 2.8.3)
project(ouster_ros)

add_compile_options(-std=c++11 -Wall)

find_package(catkin REQUIRED COMPONENTS
  message_generation
  std_msgs
  sensor_msgs
  pcl_ros
  pcl_conversions
  roscpp
  tf2
  tf2_ros
  tf2_geometry_msgs
)

add_message_files(DIRECTORY msg FILES PacketMsg.msg)
add_service_files(FILES OS1ConfigSrv.srv)
generate_messages(DEPENDENCIES std_msgs sensor_msgs)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ouster_ros
  CATKIN_DEPENDS pcl_ros pcl_conversions roscpp message_runtime std_msgs sensor_msgs
)

# ==== Libraries ====
add_library(ouster_ros STATIC src/os1_ros.cpp src/os1.cpp src/compat.cpp src/os1_util.cpp)
target_link_libraries(ouster_ros PUBLIC ${catkin_LIBRARIES})
target_include_directories(ouster_ros PUBLIC include ${catkin_INCLUDE_DIRS})
target_compile_options(ouster_ros PRIVATE "$<$<CONFIG:DEBUG>:${FLAGS_DEBUG}>")
target_compile_options(ouster_ros PRIVATE "$<$<CONFIG:RELEASE>:${FLAGS_RELEASE}>")
target_compile_options(ouster_ros PRIVATE "${FLAGS}")
add_dependencies(ouster_ros ${PROJECT_NAME}_gencpp)

# ==== Executables ====
add_executable(os1_node src/os1_node.cpp)
target_link_libraries(os1_node ouster_ros ${catkin_LIBRARIES} ${VTK_LIBRARIES})
target_include_directories(os1_node PRIVATE include ${catkin_INCLUDE_DIRS})
add_dependencies(os1_node ${PROJECT_NAME}_gencpp)

add_executable(os1_cloud_node src/os1_cloud_node.cpp)
target_link_libraries(os1_cloud_node ouster_ros ${catkin_LIBRARIES} ${VTK_LIBRARIES})
target_include_directories(os1_cloud_node PRIVATE include ${catkin_INCLUDE_DIRS})
add_dependencies(os1_cloud_node ${PROJECT_NAME}_gencpp)


# ==== Install ====
install(TARGETS os1_node os1_cloud_node 
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
install(DIRECTORY src/launch/
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
        PATTERN ".svn" EXCLUDE
        )

install(DIRECTORY src/rviz/
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/rviz
        PATTERN ".svn" EXCLUDE
        )
