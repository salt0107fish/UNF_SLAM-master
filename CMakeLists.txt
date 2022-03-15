cmake_minimum_required(VERSION 2.8)
project(ULNC_SLAM-master)

SET(CMAKE_BUILD_TYPE Debug)
IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

#set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall  -O3 -march=native ")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall   -O3 -march=native")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -O3 -march=native ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}   -O3 -march=native")

add_definitions(-w)
# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++14" COMPILER_SUPPORTS_CXX14)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX14)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
   add_definitions(-DCOMPILEDWITHC14)
   message(STATUS "Using flag -std=c++14.")
elseif(COMPILER_SUPPORTS_CXX0X)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
   add_definitions(-DCOMPILEDWITHC0X)
   message(STATUS "Using flag -std=c++0x.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++14 support. Please use a different C++ compiler.")
endif()

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

find_package(OpenCV 4.5 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)

set(TORCH_PATH /usr/local/libtorch1.8.2/libtorch/share/cmake/Torch)
if( TORCH_PATH ) 
   message("TORCH_PATH set to: ${TORCH_PATH}")
   set(Torch_DIR ${TORCH_PATH})
else()
   message(FATAL_ERROR "Need to specify Torch path, e.g., pytorch/torch/share/cmake/Torch ")
endif()

find_package(Torch REQUIRED)
message(STATUS "Torch version is: ${Torch_VERSION}")
if(Torch_VERSION GREATER 1.0.1)
   message(STATUS "Torch version is newer than v1.0.1, will use new api")
   add_definitions(-DTORCH_NEW_API)
endif()

include_directories(
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/include/orbslam
${PROJECT_SOURCE_DIR}/include/unlc_slam
${EIGEN3_INCLUDE_DIR}
${Pangolin_INCLUDE_DIRS}
)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

add_library(${PROJECT_NAME} SHARED
src/orbslam/System.cc
src/orbslam/Tracking.cc
src/orbslam/LocalMapping.cc
src/orbslam/LoopClosing.cc
src/orbslam/ORBmatcher.cc
src/orbslam/FrameDrawer.cc
src/orbslam/Converter.cc
src/orbslam/MapPoint.cc
src/orbslam/KeyFrame.cc
src/orbslam/Map.cc
src/orbslam/MapDrawer.cc
src/orbslam/Optimizer.cc
src/orbslam/PnPsolver.cc
src/orbslam/Frame.cc
src/orbslam/KeyFrameDatabase.cc
src/orbslam/Sim3Solver.cc
src/orbslam/Initializer.cc
src/orbslam/Viewer.cc
src/ulnc_slam/ULNCextractor.cc
)

target_link_libraries(${PROJECT_NAME}
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${TORCH_LIBRARIES}
${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so
${PROJECT_SOURCE_DIR}/Thirdparty/g2o/lib/libg2o.so
)

# Build examples

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/test/RGB-D)

add_executable(rgbd_kitti
test/RGB-D/rgbd_kitti.cc)
target_link_libraries(rgbd_kitti ${PROJECT_NAME})

#add_executable(stereo_kitti
#Examples/Stereo/stereo_kitti.cc)
#target_link_libraries(stereo_kitti ${PROJECT_NAME})

#add_executable(stereo_euroc
#Examples/Stereo/stereo_euroc.cc)
#target_link_libraries(stereo_euroc ${PROJECT_NAME})


set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/test/Monocular)

#add_executable(mono_tum
#Examples/Monocular/mono_tum.cc)
#target_link_libraries(mono_tum ${PROJECT_NAME})

add_executable(ulnc_kitti
test/Monocular/ulnc_kitti.cc)
target_link_libraries(ulnc_kitti ${PROJECT_NAME} ${TORCH_LIBRARIES})

#add_executable(mono_euroc
#Examples/Monocular/mono_euroc.cc)
#target_link_libraries(mono_euroc ${PROJECT_NAME})

