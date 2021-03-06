cmake_minimum_required(VERSION 2.8.3)
project(rtt-gazebo-lwr-integration)
set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
#list(APPEND CMAKE_CXX_FLAGS "-std=c++11")
## Find catkin macros and libraries
#find_package(catkin REQUIRED COMPONENTS 
#    rtt_ros 
#    cmake_modules 
#    sensor_msgs 
#    message_generation 
#    std_msgs
#)
#
#catkin_package()

find_package(OROCOS-RTT REQUIRED COMPONENTS rtt-scripting rtt-transport-corba rtt-typekit)
if (NOT OROCOS-RTT_FOUND)
  message (FATAL_ERROR "\nCould not find Orocos-RTT. Please use the shell command\n 'source orocos_toolchain/env.sh' and then run cmake again.")
endif()

#find_package(OROCOS-OCL REQUIRED COMPONENTS ocl-deployment ocl-taskbrowser ocl-logging)

include(${OROCOS-RTT_USE_FILE_PATH}/UseOROCOS-RTT.cmake )

# Depend on system install of Gazebo 
find_package(gazebo REQUIRED)
message(STATUS "GAZEBO version: ${GAZEBO_VERSION}") 
message(STATUS "GAZEBO_INCLUDE_DIRS: ${GAZEBO_INCLUDE_DIRS}")
message(STATUS "GAZEBO_LIBRARIES: ${GAZEBO_LIBRARIES}")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${GAZEBO_CXX_FLAGS}")

set ( CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/eigen/  )
find_package(Eigen REQUIRED)
find_package(Boost COMPONENTS thread REQUIRED)

find_package(RCI REQUIRED)
message(STATUS "RCI version: ${RCI_VERSION}")	
include_directories(BEFORE SYSTEM ${RCI_INCLUDE_DIRS})	
add_definitions(${RCI_CFLAGS})
link_directories(${RCI_LIBRARY_DIR})

find_package(RSC REQUIRED)
message(STATUS "RSC version: ${RSC_VERSION}")	
include_directories(BEFORE SYSTEM ${RSC_INCLUDE_DIRS})	
add_definitions(${RSC_CFLAGS})
link_directories(${RSC_LIBRARY_DIR})

find_package(NemoMath REQUIRED)
message(STATUS "NemoMath version: ${NemoMath_VERSION}")	
include_directories(BEFORE SYSTEM ${NemoMath_INCLUDE_DIRS})	
add_definitions(${NemoMath_CFLAGS})
link_directories(${NemoMath_LIBRARY_DIR})

#find_package(orocos_kdl REQUIRED)

link_directories(
  ${GAZEBO_LIBRARY_DIRS}
  ${orocos_kdl_LIBRARY_DIRS}
  )

include_directories(include
  ${Boost_INCLUDE_DIR}
  ${GAZEBO_INCLUDE_DIRS}
  ${orocos_kdl_INCLUDE_DIRS}
  ${USE_OROCOS_INCLUDE_DIRS}
  ${Eigen_INCLUDE_DIR}
  )

orocos_use_package( ocl-taskbrowser )
orocos_use_package( ocl-deployment )
orocos_use_package( ocl-logging )
orocos_use_package( orocos-rtt-corba )
#orocos_use_package( rtt-typekit )

#add_definitions(${OROCOS-RTT_DEFINITIONS})

if(${OROCOS_TARGET} STREQUAL "xenomai" )
  find_package(Xenomai REQUIRED)
  set_property(DIRECTORY ${PROJECT_SOURCE_DIR} APPEND PROPERTY COMPILE_DEFINITIONS HAVE_RTNET XENOMAI)
endif()

## Default component
orocos_component(lwr_gazebo src/lwr_gazebo_component.cpp)
set_target_properties(lwr_gazebo PROPERTIES COMPILE_DEFINITIONS RTT_COMPONENT)
target_link_libraries(lwr_gazebo 
  ${orocos_kdl_LIBRARIES} 
  ${GAZEBO_LIBRARIES} 
  ${OROCOS-RTT_LIBRARIES}
  ${OROCOS-RTT_LIBRARIES} 
  ${OROCOS-RTT_RTT-SCRIPTING_LIBRARY} 
  ${OROCOS-RTT_RTT-TRANSPORT-CORBA_LIBRARY}
  ${OROCOS-RTT_TYPEKIT_LIBRARIES}
  ${USE_OROCOS_LIBRARIES}
  ${Boost_LIBRARIES}
  ${NemoMath_LIBRARIES}
  ${RSC_LIBRARIES}
  ${RCI_LIBRARIES})

orocos_generate_package()
