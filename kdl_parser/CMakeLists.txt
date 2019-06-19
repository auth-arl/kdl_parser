cmake_minimum_required(VERSION 2.8.3)

project(kdl_parser)

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../../../core/cmake/ ${CMAKE_MODULE_PATH} )
set(EIGEN3_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../eigen3/)

find_package(Boost REQUIRED)
include_directories(${Boost_INCLUDE_DIR})

find_package(catkin REQUIRED
  COMPONENTS rosconsole urdf cmake_modules
)

find_package(orocos-kdl REQUIRED)
find_package(TinyXML REQUIRED)

include_directories(include ${EIGEN3_INCLUDE_DIR} ${orocos-kdl_INCLUDE_DIR} ${TinyXML_INCLUDE_DIRS} ${catkin_INCLUDE_DIRS})

link_directories(${catkin_LIBRARY_DIRS})
link_directories(${Boost_LIBRARY_DIRS})

add_compile_options(-std=c++11)

catkin_package(
  LIBRARIES ${PROJECT_NAME} ${orocos-kdl_LIBRARIES}
  INCLUDE_DIRS include
  CATKIN_DEPENDS rosconsole urdf
  DEPENDS orocos-kdl TinyXML
)

add_library(${PROJECT_NAME} src/kdl_parser.cpp)
target_link_libraries(${PROJECT_NAME}
  ${TinyXML_LIBRARIES} ${orocos-kdl_LIBRARIES} ${catkin_LIBRARIES}
)

if(WIN32)
  target_compile_definitions(${PROJECT_NAME} PRIVATE "KDL_PARSER_BUILDING_DLL")
endif()

add_executable(check_kdl_parser src/check_kdl_parser.cpp )
target_link_libraries(check_kdl_parser ${PROJECT_NAME})

if(CATKIN_ENABLE_TESTING)
  find_package(catkin REQUIRED COMPONENTS roscpp rostest)
  add_rostest_gtest(test_kdl_parser test/test_kdl_parser.launch test/test_kdl_parser.cpp )
  target_link_libraries(test_kdl_parser ${PROJECT_NAME})

  add_rostest_gtest(test_inertia_rpy test/test_inertia_rpy.launch test/test_inertia_rpy.cpp )
  target_link_libraries(test_inertia_rpy ${PROJECT_NAME})
endif()

# How does CATKIN do this?
#rosbuild_add_rostest(${PROJECT_SOURCE_DIR}/test/test_kdl_parser.launch)
install(TARGETS ${PROJECT_NAME}
  DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})
