#############################################################
# Copyright (c) 2022, University of California, Los Angeles #
#                                                           #
# Modifier: Julian Gaal                                     #
# Modifier: Eungchang Mason Lee                             #
# Authors: Kenny J. Chen, Brett T. Lopez                    #
# Contact: kennyjchen@ucla.edu, btlopez@ucla.edu            #
#############################################################

cmake_minimum_required(VERSION 3.10.0)
project(nano_gicp)

set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_THREAD_PREFER_PTHREAD TRUE)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(PCL REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(OpenMP REQUIRED)

include_directories(
  include
  ${PCL_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
)

add_library(nanoflann SHARED src/nanoflann.cc)
target_include_directories(nanoflann PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
target_link_libraries(nanoflann PUBLIC
  ${PCL_LIBRARIES}
  Eigen3::Eigen
  OpenMP::OpenMP_CXX
)
set_target_properties(nanoflann PROPERTIES EXPORT_NAME nanoflann)

add_library(${PROJECT_NAME} SHARED src/nano_gicp.cc)
target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
target_link_libraries(${PROJECT_NAME} PUBLIC
  nanoflann
  ${PCL_LIBRARIES}
  Eigen3::Eigen
  OpenMP::OpenMP_CXX
)
set_target_properties(${PROJECT_NAME} PROPERTIES EXPORT_NAME nano_gicp)

install(DIRECTORY include/
  DESTINATION include
)

install(
  TARGETS nanoflann
  EXPORT export_nanoflann
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

install(
  TARGETS ${PROJECT_NAME}
  EXPORT export_nano_gicp
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

ament_export_targets(export_nanoflann HAS_LIBRARY_TARGET)
ament_export_dependencies(PCL Eigen3 OpenMP)
ament_export_targets(export_nano_gicp HAS_LIBRARY_TARGET)
ament_export_dependencies(PCL Eigen3 OpenMP)
ament_package()
