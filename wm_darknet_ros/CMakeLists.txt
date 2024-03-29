cmake_minimum_required(VERSION 2.8.3)
project(wm_darknet_ros)


set(CMAKE_CXX_FLAGS "-std=c++11")

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  actionlib
  cv_bridge
  image_transport
  message_generation
  roscpp
  rospy
  sensor_msgs
  std_msgs
  wm_darknet_msgs
)


# Define path of darknet folder here.
find_path( DARKNET_PATH
    NAMES "README.md"
    HINTS "${CMAKE_CURRENT_SOURCE_DIR}/../darknet/")

message("Darknet path dir = ${DARKNET_PATH}")
add_definitions(-DDARKNET_FILE_PATH="${DARKNET_PATH}")

# Find CUDA
FIND_PACKAGE(CUDA)
if (CUDA_FOUND)
    find_package(CUDA REQUIRED)
    message(STATUS "CUDA Version: ${CUDA_VERSION_STRINGS}")
    message(STATUS"CUDA Library: ${CUDA_LIBRARIES}")
    set(
            CUDA_NVCC_FLAGS
            ${CUDA_NVCC_FLAGS};
            -O3
            -gencode arch=compute_20,code=[sm_20,sm_21]
            -gencode arch=compute_30,code=sm_30
            -gencode arch=compute_35,code=sm_35
            -gencode arch=compute_50,code=[sm_50,compute_50]
            -gencode arch=compute_52,code=[sm_52,compute_52]
            -gencode arch=compute_61,code=sm_61
            -gencode arch=compute_62,code=sm_62
        )
    add_definitions(-DGPU)
else()
    list(APPEND LIBRARIES "m")
endif()

# Find X11
MESSAGE ( STATUS "Searching for X11...")
FIND_PACKAGE( X11 REQUIRED)
IF (X11_FOUND)
    INCLUDE_DIRECTORIES(${X11_INCLUDE_DIR})
    LINK_LIBRARIES(${X11_LIBRARIES})
    MESSAGE(STATUS "X11_INCLUDE_DIR: " ${X11_INCLUDE_DIR})
    MESSAGE(STATUS "X11_LIBRARIES: " ${X11_LIBRARIES})
ENDIF(X11_FOUND)

# Find Required packages
find_package(Boost REQUIRED COMPONENTS thread)
find_package(OpenCV REQUIRED)
find_package(catkin REQUIRED
    cv_bridge
    roscpp
    rospy
    std_msgs
    actionlib
    wm_darknet_msgs
    image_transport
)


# Enable OPENCV in darknet
add_definitions(-DOPENCV)
add_definitions(-O4 -g)
add_definitions(-DDEBUG)


catkin_package(
  INCLUDE_DIRS
    include
  LIBRARIES
    ${PROJECT_NAME}_lib
  CATKIN_DEPENDS
    cv_bridge
    roscpp
    actionlib
    rospy
    std_msgs
    wm_darknet_msgs
    image_transport
  DEPENDS
    Boost
)

include_directories(
  ${DARKNET_PATH}/src
  ${DARKNET_PATH}/include
  include
  ${Boost_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
)

if (CUDA_FOUND)

  link_directories(
    /usr/local/cuda/lib64
  )

  cuda_add_library(${PROJECT_NAME}_lib
    src/YoloObjectDetector.cpp
    src/object_detector_demo.cpp
    src/image_interface.c

    ${DARKNET_PATH}/src/activation_layer.c        ${DARKNET_PATH}/src/im2col.c
    ${DARKNET_PATH}/src/activations.c             ${DARKNET_PATH}/src/image.c
    ${DARKNET_PATH}/src/avgpool_layer.c           ${DARKNET_PATH}/src/layer.c
    ${DARKNET_PATH}/src/batchnorm_layer.c         ${DARKNET_PATH}/src/list.c
    ${DARKNET_PATH}/src/blas.c                    ${DARKNET_PATH}/src/local_layer.c
    ${DARKNET_PATH}/src/box.c                     ${DARKNET_PATH}/src/lstm_layer.c
    ${DARKNET_PATH}/src/col2im.c                  ${DARKNET_PATH}/src/matrix.c
    ${DARKNET_PATH}/src/compare.c                 ${DARKNET_PATH}/src/maxpool_layer.c
    ${DARKNET_PATH}/src/connected_layer.c         ${DARKNET_PATH}/src/network.c
    ${DARKNET_PATH}/src/convolutional_layer.c     ${DARKNET_PATH}/src/normalization_layer.c
    ${DARKNET_PATH}/src/cost_layer.c              ${DARKNET_PATH}/src/option_list.c
    ${DARKNET_PATH}/src/crnn_layer.c              ${DARKNET_PATH}/src/parser.c
    ${DARKNET_PATH}/src/crop_layer.c              ${DARKNET_PATH}/src/region_layer.c
    ${DARKNET_PATH}/src/cuda.c                    ${DARKNET_PATH}/src/reorg_layer.c
    ${DARKNET_PATH}/src/data.c                    ${DARKNET_PATH}/src/rnn_layer.c
    ${DARKNET_PATH}/src/deconvolutional_layer.c   ${DARKNET_PATH}/src/route_layer.c
    ${DARKNET_PATH}/src/demo.c                    ${DARKNET_PATH}/src/shortcut_layer.c
    ${DARKNET_PATH}/src/detection_layer.c         ${DARKNET_PATH}/src/softmax_layer.c
    ${DARKNET_PATH}/src/dropout_layer.c           ${DARKNET_PATH}/src/tree.c
    ${DARKNET_PATH}/src/gemm.c                    ${DARKNET_PATH}/src/utils.c
    ${DARKNET_PATH}/src/gru_layer.c

    ${DARKNET_PATH}/examples/art.c                ${DARKNET_PATH}/examples/regressor.c
    ${DARKNET_PATH}/examples/captcha.c            ${DARKNET_PATH}/examples/rnn.c
    ${DARKNET_PATH}/examples/cifar.c              ${DARKNET_PATH}/examples/rnn_vid.c
    ${DARKNET_PATH}/examples/classifier.c         ${DARKNET_PATH}/examples/segmenter.c
    ${DARKNET_PATH}/examples/coco.c               ${DARKNET_PATH}/examples/super.c
    ${DARKNET_PATH}/examples/detector.c           ${DARKNET_PATH}/examples/swag.c
    ${DARKNET_PATH}/examples/dice.c               ${DARKNET_PATH}/examples/tag.c
    ${DARKNET_PATH}/examples/go.c                 ${DARKNET_PATH}/examples/voxel.c
    ${DARKNET_PATH}/examples/lsd.c                ${DARKNET_PATH}/examples/writing.c
    ${DARKNET_PATH}/examples/nightmare.c          ${DARKNET_PATH}/examples/yolo.c

    ${DARKNET_PATH}/src/activation_kernels.cu     ${DARKNET_PATH}/src/deconvolutional_kernels.cu
    ${DARKNET_PATH}/src/avgpool_layer_kernels.cu  ${DARKNET_PATH}/src/dropout_layer_kernels.cu
    ${DARKNET_PATH}/src/blas_kernels.cu           ${DARKNET_PATH}/src/im2col_kernels.cu
    ${DARKNET_PATH}/src/col2im_kernels.cu         ${DARKNET_PATH}/src/maxpool_layer_kernels.cu
    ${DARKNET_PATH}/src/convolutional_kernels.cu  ${DARKNET_PATH}/src/network_kernels.cu
    ${DARKNET_PATH}/src/crop_layer_kernels.cu
  )

  target_link_libraries(${PROJECT_NAME}_lib
    m
    pthread
    stdc++
    cuda
    cudart
    cublas
    curand
    ${Boost_LIBRARIES}
    ${catkin_LIBRARIES}
  )

  cuda_add_executable(${PROJECT_NAME}
    src/yolo_object_detector_node.cpp
  )

else()

  add_library(${PROJECT_NAME}_lib
    src/YoloObjectDetector.cpp
    src/object_detector_demo.cpp
    src/image_interface.c

    ${DARKNET_PATH}/src/activation_layer.c        ${DARKNET_PATH}/src/im2col.c
    ${DARKNET_PATH}/src/activations.c             ${DARKNET_PATH}/src/image.c
    ${DARKNET_PATH}/src/avgpool_layer.c           ${DARKNET_PATH}/src/layer.c
    ${DARKNET_PATH}/src/batchnorm_layer.c         ${DARKNET_PATH}/src/list.c
    ${DARKNET_PATH}/src/blas.c                    ${DARKNET_PATH}/src/local_layer.c
    ${DARKNET_PATH}/src/box.c                     ${DARKNET_PATH}/src/lstm_layer.c
    ${DARKNET_PATH}/src/col2im.c                  ${DARKNET_PATH}/src/matrix.c
    ${DARKNET_PATH}/src/compare.c                 ${DARKNET_PATH}/src/maxpool_layer.c
    ${DARKNET_PATH}/src/connected_layer.c         ${DARKNET_PATH}/src/network.c
    ${DARKNET_PATH}/src/convolutional_layer.c     ${DARKNET_PATH}/src/normalization_layer.c
    ${DARKNET_PATH}/src/cost_layer.c              ${DARKNET_PATH}/src/option_list.c
    ${DARKNET_PATH}/src/crnn_layer.c              ${DARKNET_PATH}/src/parser.c
    ${DARKNET_PATH}/src/crop_layer.c              ${DARKNET_PATH}/src/region_layer.c
    ${DARKNET_PATH}/src/cuda.c                    ${DARKNET_PATH}/src/reorg_layer.c
    ${DARKNET_PATH}/src/data.c                    ${DARKNET_PATH}/src/rnn_layer.c
    ${DARKNET_PATH}/src/deconvolutional_layer.c   ${DARKNET_PATH}/src/route_layer.c
    ${DARKNET_PATH}/src/demo.c                    ${DARKNET_PATH}/src/shortcut_layer.c
    ${DARKNET_PATH}/src/detection_layer.c         ${DARKNET_PATH}/src/softmax_layer.c
    ${DARKNET_PATH}/src/dropout_layer.c           ${DARKNET_PATH}/src/tree.c
    ${DARKNET_PATH}/src/gemm.c                    ${DARKNET_PATH}/src/utils.c
    ${DARKNET_PATH}/src/gru_layer.c

    ${DARKNET_PATH}/examples/art.c                ${DARKNET_PATH}/examples/regressor.c
    ${DARKNET_PATH}/examples/captcha.c            ${DARKNET_PATH}/examples/rnn.c
    ${DARKNET_PATH}/examples/cifar.c              ${DARKNET_PATH}/examples/rnn_vid.c
    ${DARKNET_PATH}/examples/classifier.c         ${DARKNET_PATH}/examples/segmenter.c
    ${DARKNET_PATH}/examples/coco.c               ${DARKNET_PATH}/examples/super.c
    ${DARKNET_PATH}/examples/detector.c           ${DARKNET_PATH}/examples/swag.c
    ${DARKNET_PATH}/examples/dice.c               ${DARKNET_PATH}/examples/tag.c
    ${DARKNET_PATH}/examples/go.c                 ${DARKNET_PATH}/examples/voxel.c
    ${DARKNET_PATH}/examples/lsd.c                ${DARKNET_PATH}/examples/writing.c
    ${DARKNET_PATH}/examples/nightmare.c          ${DARKNET_PATH}/examples/yolo.c
  )

  target_link_libraries(${PROJECT_NAME}_lib
    m
    pthread
    stdc++
    ${Boost_LIBRARIES}
    ${catkin_LIBRARIES}
  )

  add_executable(${PROJECT_NAME}
    src/yolo_object_detector_node.cpp
  )

endif()

target_link_libraries(${PROJECT_NAME}
  ${PROJECT_NAME}_lib
)

install(TARGETS ${PROJECT_NAME}_lib
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(TARGETS ${PROJECT_NAME}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(
  DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
)

# Download tiny-yolo-voc.weights
set(PATH "${CMAKE_CURRENT_SOURCE_DIR}/yolo_network_config/weights")
set(FILE "${PATH}/tiny-yolo-voc.weights")
message(STATUS "Checking and downloading tiny-yolo-voc.weights if needed ...")
if (NOT EXISTS "${FILE}")
  message(STATUS "... file does not exist. Downloading now ...")
  execute_process(COMMAND wget http://pjreddie.com/media/files/tiny-yolo-voc.weights -P ${PATH})
endif()

