#
# Set minimum required version
#
cmake_minimum_required(VERSION 2.8.12)

#
# Options
#
option(ARGOS_SIMULATION "Build for ARGoS Simulation" ON)
option(IRL_ROBOT "Build for IRL robot" ON)
option(ROS_IRL_ROBOT "Build for IRL robot using ROS" ON)

#
# Find pkg-config
#
find_package(PkgConfig REQUIRED)

if(ARGOS_SIMULATION)
#
# Look for the optional ARGoS package
#
pkg_check_modules(ARGOS argos3_simulator)
if(ARGOS_FOUND)
  include_directories(${ARGOS_INCLUDE_DIRS})
  link_directories(${ARGOS_LIBRARY_DIRS})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${ARGOS_PREFIX}/share/argos3/cmake)
  include(ARGoSCheckQTOpenGL)
  # Look for Lua52
  find_package(Lua52 REQUIRED)
  include_directories(${LUA_INCLUDE_DIR})
  link_directories(${ARGOS_PREFIX}/lib/argos3/)
endif(ARGOS_FOUND)
endif(ARGOS_SIMULATION)

#
# Find dependancies
#
# Find GTSAM
find_package(GTSAM REQUIRED)
include_directories(${GTSAM_INCLUDE_DIR})
link_directories(${GTSAM_LIBRARY_DIRS})
message("GTSAM Libraries: ${GTSAM_LIBRARIES}")
# for unittest examples
set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH}" "${GTSAM_DIR}/../GTSAMCMakeTools")
include(GtsamTesting)

# Find Boost
find_package(Boost COMPONENTS serialization filesystem system program_options thread REQUIRED)
include_directories(${Boost_INCLUDE_DIRS})
link_libraries(${Boost_LIBRARIES})
message("Boost Libraries: ${Boost_LIBRARIES}")

#
# Compile buzz slam library for ARGoS
#
if(ARGOS_SIMULATION)
set(ARGOS_BUZZ_SOURCES
  slam/utils/buzz_slam_utils.h
  slam/buzz_slam.h
  slam/buzz_slam.cpp
  slam/buzz_slam_singleton.h
  slam/buzz_slam_closures.cpp
  slam/specialized/no_sensing/buzz_slam_no_sensing_closures.cpp
  slam/specialized/no_sensing/buzz_slam_no_sensing.cpp
  slam/specialized/no_sensing/buzz_slam_no_sensing.h
  slam/specialized/dataset/buzz_slam_dataset_closures.cpp
  slam/specialized/dataset/buzz_slam_dataset.cpp
  slam/specialized/dataset/buzz_slam_dataset.h
  argos/buzz_controller_quadmapper.h      
  argos/buzz_controller_quadmapper.cpp     
  argos/simulation_random_walk/buzz_controller_quadmapper_no_sensing.h      
  argos/simulation_random_walk/buzz_controller_quadmapper_no_sensing.cpp
  argos/simulation_random_walk/buzz_closures_quadmapper_no_sensing.cpp     
  argos/simulation_dataset/buzz_controller_quadmapper_with_dataset.h      
  argos/simulation_dataset/buzz_controller_quadmapper_with_dataset.cpp
  argos/simulation_dataset/buzz_closures_quadmapper_with_dataset.cpp)

add_library(argos3plugin_simulator_buzz_slam SHARED ${ARGOS_BUZZ_SOURCES})
target_link_libraries(argos3plugin_simulator_buzz_slam
  argos3core_simulator
  argos3plugin_simulator_genericrobot
  argos3plugin_simulator_footbot
  argos3plugin_simulator_spiri
  argos3plugin_simulator_buzz
  buzz
  gtsam
  distributed_mapper
  distributed_pcm
  global_map_solver
  fast_max-clique_finder
  pairwise_consistency
  robot_measurements
  graph_utils
  )

add_compile_options ( -g -rdynamic )
set_target_properties(argos3plugin_simulator_buzz_slam PROPERTIES LINKER_LANGUAGE CXX)

install(TARGETS argos3plugin_simulator_buzz_slam LIBRARY DESTINATION ${ARGOS_PREFIX}/lib/argos3)
install(DIRECTORY . DESTINATION include/buzz/argos FILES_MATCHING PATTERN "*.h")
endif(ARGOS_SIMULATION)

if(IRL_ROBOT)
find_package(Buzz REQUIRED)
include_directories(${CMAKE_SOURCE_DIR}/src ${BUZZ_C_INCLUDE_DIR})
add_executable(generic_robot_buzz_slam
  slam/utils/buzz_slam_utils.h
  slam/buzz_slam.h
  slam/buzz_slam.cpp
  slam/buzz_slam_singleton.h
  slam/buzz_slam_closures.cpp
  slam/specialized/no_sensing/buzz_slam_no_sensing_closures.cpp
  slam/specialized/no_sensing/buzz_slam_no_sensing.cpp
  slam/specialized/no_sensing/buzz_slam_no_sensing.h
  slam/specialized/dataset/buzz_slam_dataset_closures.cpp
  slam/specialized/dataset/buzz_slam_dataset.cpp
  slam/specialized/dataset/buzz_slam_dataset.h
  generic_robot/utils/buzz_utils.h
  generic_robot/buzz_generic_robot_slam_datasets.cpp)
target_link_libraries(generic_robot_buzz_slam 
  buzz
  buzzdbg
  gtsam
  distributed_mapper
  distributed_pcm
  global_map_solver
  fast_max-clique_finder
  pairwise_consistency
  robot_measurements
  graph_utils
  -lpthread
  )
install(TARGETS generic_robot_buzz_slam RUNTIME DESTINATION bin)
endif(IRL_ROBOT)

if(ROS_IRL_ROBOT)
find_package(Buzz REQUIRED)
include_directories(${CMAKE_SOURCE_DIR}/src ${BUZZ_C_INCLUDE_DIR})
include_directories(${catkin_INCLUDE_DIRS})
add_executable(generic_robot_buzz_slam_node
  slam/utils/buzz_slam_utils.h
  slam/buzz_slam.h
  slam/buzz_slam.cpp
  slam/buzz_slam_singleton.h
  slam/buzz_slam_closures.cpp
  slam/specialized/ros/buzz_slam_ros_closures.cpp
  slam/specialized/ros/buzz_slam_ros.cpp
  slam/specialized/ros/buzz_slam_ros.h
  generic_robot/utils/buzz_utils.h
  generic_robot/message_handler_utils/message_handler_utils.h
  generic_robot/buzz_generic_robot_slam_ros.cpp)
find_library(MULTIROBOT_SEPARATORS_LIBRARY multi_robot_separators HINTS /root/multi_robot_SLAM_separators/ros_ws/devel/lib)
target_link_libraries(generic_robot_buzz_slam_node 
  buzz
  buzzdbg
  gtsam
  distributed_mapper
  distributed_pcm
  global_map_solver
  fast_max-clique_finder
  pairwise_consistency
  robot_measurements
  graph_utils
  -lpthread
  ${catkin_LIBRARIES}
  ${MULTIROBOT_SEPARATORS_LIBRARY}
  )
install(TARGETS generic_robot_buzz_slam_node RUNTIME DESTINATION bin)
endif(ROS_IRL_ROBOT)