cmake_minimum_required(VERSION 3.5)
project(unitree_ros2_to_real)

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(ros2_unitree_legged_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(std_srvs REQUIRED)

message("-- CMAKE_SYSTEM_PROCESSOR: ${CMAKE_SYSTEM_PROCESSOR}")
if("${CMAKE_SYSTEM_PROCESSOR}" MATCHES "x86_64.*")
  set(ARCH amd64)
else()
  set(ARCH arm64)
endif()
set(EXTRA_LIBS -pthread libunitree_legged_sdk_${ARCH}.so lcm)

set(CMAKE_CXX_FLAGS "-O3")

include_directories(
  include
  ${CMAKE_SOURCE_DIR}/unitree_legged_sdk/include
  )
  
install(
  DIRECTORY ${CMAKE_SOURCE_DIR}/unitree_legged_sdk/lib/
  DESTINATION lib
)

link_directories(${CMAKE_SOURCE_DIR}/unitree_legged_sdk/lib)


# LCM STUFF
include_directories($ENV{UNITREE_LEGGED_SDK_PATH}/include)
link_directories($ENV{UNITREE_LEGGED_SDK_PATH}/lib)
string(CONCAT LEGGED_SDK_NAME libunitree_legged_sdk_$ENV{UNITREE_PLATFORM}.so)
set(EXTRA_LIBS ${LEGGED_SDK_NAME} lcm)

add_definitions(-DSDK3_2)

add_executable(lcm_server_3_2 $ENV{UNITREE_LEGGED_SDK_PATH}/examples/lcm_server.cpp)
target_link_libraries(lcm_server_3_2 ${EXTRA_LIBS} ${catkin_LIBRARIES})
ament_target_dependencies(lcm_server_3_2 ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(twist_driver src/twist_driver.cpp)
target_link_libraries(twist_driver ${EXTRA_LIBS})
ament_target_dependencies(twist_driver rclcpp ros2_unitree_legged_msgs geometry_msgs std_srvs)

add_executable(joy_to_twist src/joy_to_twist.cpp)
target_link_libraries(joy_to_twist ${EXTRA_LIBS})
ament_target_dependencies(joy_to_twist rclcpp sensor_msgs geometry_msgs std_srvs)

install(DIRECTORY 
   launch 
   DESTINATION share/${PROJECT_NAME}/
)

install(TARGETS
    lcm_server_3_2
    twist_driver
    joy_to_twist
    DESTINATION lib/${PROJECT_NAME}
)


ament_package()




