cmake_minimum_required(VERSION 2.8.3)
project(calibration_camera_lidar)
execute_process(
  COMMAND rosversion -d
  OUTPUT_VARIABLE ROS_VERSION
  OUTPUT_STRIP_TRAILING_WHITESPACE
)

include(FindPkgConfig)
include(FindLibXml2)
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rosconsole
  std_msgs
  sensor_msgs
  glviewer
  rosinterface
  cv_bridge
  pcl_ros
  image_transport
  autoware_msgs
)

find_package(OpenCV REQUIRED)

set(CMAKE_AUTOMOC ON)
#set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

find_package(Qt5Core REQUIRED)
find_package(Qt5Widgets REQUIRED)
find_package(Qt5OpenGL REQUIRED)

catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES camera
   CATKIN_DEPENDS glviewer rosinterface std_msgs message_runtime autoware_msgs
#  DEPENDS system_lib
)

set(CMAKE_CXX_FLAGS "-std=c++11 -O2 -Wall -g ${CMAKE_CXX_FLAGS}")

EXECUTE_PROCESS(
  COMMAND pkg-config --variable=host_bins Qt5Core
  OUTPUT_VARIABLE Qt5BIN
  OUTPUT_STRIP_TRAILING_WHITESPACE
)

## Specify additional locations of header files
## Your package locations should be listed before other locations
# include_directories(include)
include_directories(
  CalibrationToolkit
  nodes/calibration_toolkit
  ${catkin_INCLUDE_DIRS}
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}
)

## Declare a cpp library
if ("${ROS_VERSION}" MATCHES "(indigo|jade|kinetic)")
add_library(calibrationtoolkit
  CalibrationToolkit/calibrationtoolkit.cpp
  CalibrationToolkit/calibrationtoolkit.h
  CalibrationToolkit/selectionwidget.cpp
  CalibrationToolkit/selectionwidget.h
)

target_link_libraries(calibrationtoolkit
  ${catkin_LIBRARIES}
  ${OpenCV_LIBS}
  Qt5::Core
  Qt5::Widgets
  Qt5::OpenGL
)

include_directories(
  CalibrationToolkit
  nodes/calibration_toolkit
  ${catkin_INCLUDE_DIRS}
  ${LIBXML2_INCLUDE_DIR}
  ${OpenCV_INCLUDE_DIRS}
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}
)
endif()

## 2D
if ("${ROS_VERSION}" MATCHES "(indigo|jade|kinetic)")
add_executable(calibration_test
  nodes/calibration_test/chessboard.cpp
  nodes/calibration_test/trans.cpp
  nodes/calibration_test/scan_window.cpp
  nodes/calibration_test/common_2d_calib.cpp
  nodes/calibration_test/image_window.cpp
  nodes/calibration_test/camera_lidar2d_offline_calib.cpp)

set(PARAM_YAML "${CMAKE_CURRENT_SOURCE_DIR}/param.yaml")
set(CAMERA_YAML "$ENV{HOME}/.ros/autoware/camera_lidar_2d.yaml")

set_target_properties(calibration_test
  PROPERTIES COMPILE_FLAGS
  "-DPARAM_YAML=${PARAM_YAML} -DCAMERA_YAML=${CAMERA_YAML}")

target_link_libraries(calibration_test
   ${catkin_LIBRARIES} ${OpenCV_LIBS} xml2)
endif()
## 3D
if ("${ROS_VERSION}" MATCHES "(indigo|jade|kinetic)")

qt5_wrap_ui(calibration_toolkit_ui_mainwindow nodes/calibration_toolkit/mainwindow.ui)


add_executable(calibration_toolkit
  nodes/calibration_toolkit/main.cpp
  nodes/calibration_toolkit/mainwindow.cpp
  nodes/calibration_toolkit/mainwindow.h
  ${calibration_toolkit_ui_mainwindow}
)

set_target_properties(calibration_toolkit
  PROPERTIES COMPILE_FLAGS "-fPIC"
)

## Specify libraries to link a library or executable target against
target_link_libraries(calibration_toolkit
  ${catkin_LIBRARIES}
  ${OpenCV_LIBS}
  calibrationtoolkit
  glut
  GLU
  nlopt
  Qt5::Core
  Qt5::Widgets
  Qt5::OpenGL
)
endif()
# calibration_publisher
add_executable(calibration_publisher
  nodes/calibration_publisher/calibration_publisher.cpp
  )

add_dependencies(calibration_publisher
  autoware_msgs_generate_messages_cpp
)

target_link_libraries(calibration_publisher
  ${catkin_LIBRARIES}
  )
