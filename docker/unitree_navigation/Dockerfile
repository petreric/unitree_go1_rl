FROM osrf/ros:humble-desktop

ARG DEBIAN_FRONTEND=noninteractive
ARG WORKSPACE_DIR=/unitree_ws

RUN apt-get update && apt-get install -y \
    git \
    build-essential \
    cmake \
    openjdk-8-jdk \
    python3-colcon-common-extensions \
    ros-humble-xacro \
    ros-humble-robot-state-publisher \
    ros-humble-joint-state-publisher \
    ros-humble-nav2-msgs \
    && rm -rf /var/lib/apt/lists/*

ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

RUN rosdep update

WORKDIR ${WORKSPACE_DIR}
RUN mkdir -p src

RUN cd src && \
    git clone https://github.com/katie-hughes/unitree_ros2.git && \
    git clone https://github.com/ngmor/unitree_nav.git

RUN cd /tmp && \
    git clone https://github.com/lcm-proj/lcm.git && \
    cd lcm && \
    mkdir build && cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release .. && \
    make -j$(nproc) && \
    make install && \
    ldconfig && \
    rm -rf /tmp/lcm

RUN cd ${WORKSPACE_DIR} && \
    . /opt/ros/humble/setup.sh && \
    rosdep install --from-paths src --ignore-src -r -y \
        --skip-keys="rslidar_sdk roscpp rtabmap rtabmap-ros" || true

RUN cd ${WORKSPACE_DIR} && \
    . /opt/ros/humble/setup.sh && \
    colcon build \
        --packages-select \
            ros2_unitree_legged_msgs \
            unitree_legged_real \
            unitree_nav \
            unitree_nav_interfaces \
            go1_description \
        --cmake-args -DCMAKE_BUILD_TYPE=Release

RUN echo '#!/bin/bash\n\
source /opt/ros/humble/setup.bash\n\
source '${WORKSPACE_DIR}'/install/setup.bash\n\
exec "$@"\n' > /entrypoint.sh && \
    chmod +x /entrypoint.sh

WORKDIR ${WORKSPACE_DIR}

ENTRYPOINT ["/entrypoint.sh"]
CMD ["ros2", "launch", "unitree_nav", "control.launch.py", "use_rviz:=false"]