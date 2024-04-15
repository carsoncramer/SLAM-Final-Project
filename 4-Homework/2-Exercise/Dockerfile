# Use ROS2 Humble official image as base
FROM ros:humble-ros-core-jammy

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    lsb-release \
    python3-pip \
    software-properties-common \
    && rm -rf /var/lib/apt/lists/*

# Install rosdep and colcon
RUN apt-get update && \
    apt-get install -y python3-rosdep python3-colcon-common-extensions && \
    rosdep init && \
    rosdep update

RUN \
  apt-get update && \
  apt-get -y install libgl1-mesa-glx libgl1-mesa-dri && \
  rm -rf /var/lib/apt/lists/*

# Add GTSAM PPA and install GTSAM
RUN add-apt-repository ppa:borglab/gtsam-release-4.1 -y \
    && apt-get update \
    && apt-get install -y libgtsam-dev libgtsam-unstable-dev

# Install rviz2 and its dependencies including OGRE
RUN apt-get update && apt-get install -y \
    ros-humble-rviz2 \
    ros-humble-rviz-ogre-vendor \
    ros-humble-rosbag2 \
    && rm -rf /var/lib/apt/lists/*

# Set up a workspace
WORKDIR /root/ros2_ws
RUN mkdir -p src

# Clone the SLAM package
RUN cd src \
    && git clone --recursive https://github.com/rsasaki0109/li_slam_ros2

# Resolve ROS dependencies
RUN apt-get update && rosdep update \
    && rosdep install --from-paths src --ignore-src -yr

# Build each ROS2 package individually to reduce peak memory usage
RUN /bin/bash -c "source /opt/ros/humble/setup.bash && \
    colcon build --packages-select lidarslam_msgs --cmake-args -DCMAKE_BUILD_TYPE=Release && \
    colcon build --packages-select ndt_omp_ros2 --cmake-args -DCMAKE_BUILD_TYPE=Release && \
    colcon build --packages-select scanmatcher --cmake-args -DCMAKE_BUILD_TYPE=Release && \
    colcon build --packages-select graph_based_slam --cmake-args -DCMAKE_BUILD_TYPE=Release"

# Copy ROS 2 bag directory into the container
COPY park_dataset_ros2_bag/ /root/ros2_ws/park_dataset_ros2_bag/

# Setup entrypoint
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["bash"]
