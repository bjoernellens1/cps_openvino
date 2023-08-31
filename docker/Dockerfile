ARG ROS_DISTRO=humble
ARG UNDERLAY_WS=/rmp_ws
ARG TARGETPLATFORM


# This file should work for both amd64 and arm64 builds. Because segway did not release some source code we must copy some binaries for the specific architectures.
# That is why we must use some conditional statements to copy over the correct files during build process.
# Variable TARGETPLATFORM should be auomatically created during build process by docker.

########################################
# Base Image for Bot Mini Control #
########################################
FROM ros:${ROS_DISTRO} as base
ENV ROS_DISTRO=${ROS_DISTRO}
#ENV TARGETPLATFORM=$TARGETPLATFORM

SHELL ["/bin/bash", "-c"]

COPY docker/entrypoint.sh /entrypoint.sh

# Install some basic dependencies
RUN apt-get update && apt-get install -y \
  curl \
  wget \
  gnupg2 \
  lsb-release \
  && rm -rf /var/lib/apt/lists/*

# Install OpenVINO Toolkit
RUN wget https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
RUN apt-key add GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
RUN echo "deb https://apt.repos.intel.com/openvino/2022 focal main" | sudo tee /etc/apt/sources.list.d/intel-openvino-2022.list
RUN apt-get update && apt-get install -y \
  openvino

# Install OpenVINO GPU Dependencies
RUN apt-get update && apt-get install -y \ 
    ocl-icd-libopencl1 \ 
    intel-opencl-icd
    #intel-level-zero-gpu \ 
    #level-zero

# Test OpenVINO Installation
RUN /usr/share/openvino/samples/cpp/build_samples.sh

# # Create Colcon workspace with external dependencies
# RUN mkdir -p /rmp_ws/src
# WORKDIR /rmp_ws/src
# COPY dependencies.arm64.repos .
# COPY dependencies.amd64.repos .

# # Choose correct sources for architecture:
# # Copy platform-specific files
# # Use if condition to copy different files for different platforms
# #RUN echo "TARGETPLATFORM is $TARGETPLATFORM"
# RUN echo "Hardware platform is: $(uname -m)"
# RUN uname -a

# RUN echo "update base 29.08.2023"

# RUN if [ "$(uname -m)" = "x86_64" ]; then \
#   echo "Copying files for linux/amd64"; \
#   mv dependencies.amd64.repos dependencies.repos; \
#   else \
#   echo "Copying files for linux/arm64"; \
#   mv dependencies.arm64.repos dependencies.repos; \
#   fi

# #RUN vcs import < dependencies.repos
# RUN vcs import < dependencies.repos;

# # Build the base Colcon workspace, installing dependencies first.
# WORKDIR /rmp_ws
# RUN source /opt/ros/${ROS_DISTRO}/setup.bash \
#   && apt-get update -y \
#   && apt-get install -y --no-install-recommends \
#    ros-${ROS_DISTRO}-foxglove-bridge \
#   && rosdep install --from-paths src --ignore-src --rosdistro ${ROS_DISTRO} -y

# RUN source /opt/ros/${ROS_DISTRO}/setup.bash \
#   && colcon build --symlink-install

# # Use Cyclone DDS as middleware and install xacro
# RUN apt-get install -y --no-install-recommends \
#   ros-${ROS_DISTRO}-rmw-cyclonedds-cpp \
#   ros-${ROS_DISTRO}-xacro \
#   && rm -rf /var/lib/apt/lists/*

# ENV RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

# ENV UNDERLAY_WS=${UNDERLAY_WS}

# # Set up the entrypoint
# COPY ./docker/entrypoint.sh /
# RUN chmod +x /entrypoint.sh

# ENTRYPOINT [ "/entrypoint.sh" ]

# ###########################################
# # Overlay Image for RMP Control #
# ###########################################
# FROM base AS overlay

# # Create an overlay Colcon workspace
# RUN mkdir -p /overlay_ws/src
# WORKDIR /overlay_ws/src
# COPY overlay.repos .
# RUN vcs import < overlay.repos

# WORKDIR /overlay_ws

# RUN echo "update overlay 22.08.2023"

# RUN source /rmp_ws/install/setup.bash \
#   && apt-get update \
#   && rosdep install --from-paths src --ignore-src --rosdistro ${ROS_DISTRO} -y \
#   && rm -rf /var/lib/apt/lists/*

# RUN source /rmp_ws/install/setup.bash \
#   && colcon build --symlink-install

# # Set up the entrypoint
# COPY ./docker/entrypoint.sh /
# RUN chmod +x /entrypoint.sh

# ENTRYPOINT [ "/entrypoint.sh" ]

# ###########################################
# # GUI Additions for Ros2 #
# ###########################################
# FROM overlay AS guis

# # Install additional GUI tools
# RUN source /rmp_ws/install/setup.bash \
#   && apt-get update \
#   && apt-get install -y --no-install-recommends ros-${ROS_DISTRO}-rviz2 \
#   && rm -rf /var/lib/apt/lists/*

# Install python3 and openvino python api and tensorflow
RUN apt-get update && apt-get install -y \
  python3 \
  python3-pip \
  && rm -rf /var/lib/apt/lists/*
RUN pip3 install tensorflow openvino openvino-dev opencv-python-headless requests tqdm onnx torch

#RUN RUN apt-get update && apt-get install ffmpeg libsm6 libxext6  -y

#RUN apt-get update && apt-get install libgl1

RUN mkdir /workspace
WORKDIR /workspace
RUN git clone https://github.com/openvinotoolkit/open_model_zoo.git

# Set up the entrypoint
RUN chmod +x /entrypoint.sh
ENTRYPOINT [ "/entrypoint.sh" ]

LABEL org.opencontainers.image.source=https://github.com/bjoernellens1/ros2_rmp