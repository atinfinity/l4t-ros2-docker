FROM docker.io/arm64v8/ubuntu:26.04

ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES all

ARG UID=1000
ARG GID=1000

# add new sudo user
ENV USERNAME jetson
ENV HOME /home/$USERNAME

# https://askubuntu.com/questions/1513927/ubuntu-24-04-docker-images-now-includes-user-ubuntu-with-uid-gid-1000
RUN if getent passwd $UID; then \
      userdel -f $(getent passwd $UID | cut -d: -f1); \
    fi \
    && if getent group $GID; then \
      groupdel $(getent group $GID | cut -d: -f1); \
    fi

RUN useradd -m $USERNAME && \
        echo "$USERNAME:$USERNAME" | chpasswd && \
        usermod --shell /bin/bash $USERNAME && \
        usermod -aG sudo $USERNAME && \
        mkdir /etc/sudoers.d && \
        echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/$USERNAME && \
        chmod 0440 /etc/sudoers.d/$USERNAME && \
        usermod  --uid $UID $USERNAME && \
        groupmod --gid $GID $USERNAME
RUN gpasswd -a $USERNAME video

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
    apt-get install -qq -y --no-install-recommends \
        bash-completion \
        bc \
        build-essential \
        bzip2 \
        can-utils \
        ca-certificates \
        cmake \
        command-not-found \
        curl \
        emacs \
        freeglut3-dev \
        git \
        gnupg2 \
        gstreamer1.0-alsa \
        gstreamer1.0-libav \
        gstreamer1.0-plugins-bad \
        gstreamer1.0-plugins-base \
        gstreamer1.0-plugins-good \
        gstreamer1.0-plugins-ugly \
        gstreamer1.0-tools \
        i2c-tools \
        iproute2 \
        iputils-ping \
        iw \
        kbd \
        kmod \
        language-pack-en-base \
        libapt-pkg-dev \
        libcanberra-gtk3-module \
        libgles2 \
        libglu1-mesa-dev \
        libglvnd-dev \
        libgtk-3-0t64 \
        libudev1 \
        libvulkan1 \
        libzmq5 \
        mesa-utils \
        mtd-utils \
        parted \
        pciutils \
        python3 \
        python3-numpy \
        python3-pexpect \
        python3-pip \
        sox \
        sudo \
        tmux \
        udev \
        vulkan-tools \
        wget \
        wpasupplicant \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# EGL
RUN echo "/usr/lib/aarch64-linux-gnu/nvidia" >> /etc/ld.so.conf.d/nvidia-tegra.conf && \
    echo "/usr/lib/aarch64-linux-gnu/tegra-egl" >> /etc/ld.so.conf.d/nvidia-tegra.conf
RUN rm -rf /usr/share/glvnd/egl_vendor.d && \
    mkdir -p /usr/share/glvnd/egl_vendor.d/ && echo '\
{\
    "file_format_version" : "1.0.0",\
    "ICD" : {\
        "library_path" : "libEGL_nvidia.so.0"\
    }\
}' > /usr/share/glvnd/egl_vendor.d/10_nvidia.json
RUN mkdir -p /usr/share/egl/egl_external_platform.d/ && echo '\
{\
    "file_format_version" : "1.0.0",\
    "ICD" : {\
        "library_path" : "libnvidia-egl-wayland.so.1"\
    }\
}' > /usr/share/egl/egl_external_platform.d/nvidia_wayland.json

RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/jetson-ota-public.gpg] https://repo.download.nvidia.com/jetson/common r39.2 main" >> /etc/apt/sources.list.d/nvidia-l4t-apt-source.list && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/jetson-ota-public.gpg] https://repo.download.nvidia.com/jetson/som r39.2 main" >> /etc/apt/sources.list.d/nvidia-l4t-apt-source.list && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/jetson-ota-public.gpg] https://repo.download.nvidia.com/jetson/ffmpeg r39.2 main" >> /etc/apt/sources.list.d/nvidia-l4t-apt-source.list
RUN wget -O /etc/jetson-ota-public.key https://gitlab.com/nvidia/container-images/l4t-base/-/raw/master/jetson-ota-public.key && \
    cat /etc/jetson-ota-public.key | gpg --dearmor -o /usr/share/keyrings/jetson-ota-public.gpg

# CUDA, cuDNN
# NOTE: use cuda-toolkit instead of the `cuda` metapackage, which pulls in
#       nvidia-driver-open. The driver is mounted from the host by the NVIDIA runtime.
RUN apt-get update && \
    apt-get install -qq -y --no-install-recommends \
        cuda-toolkit \
        libcudnn9-cuda-13 \
        libcudnn9-dev-cuda-13 \
        libcudnn9-samples \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN ldconfig

# install ROS 2 Lyrical
ARG ROS_APT_SOURCE_VERSION=1.2.0
RUN curl -fsSL -o /tmp/ros2-apt-source.deb \
        "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo $UBUNTU_CODENAME)_all.deb" && \
    dpkg -i /tmp/ros2-apt-source.deb && \
    rm -f /tmp/ros2-apt-source.deb
RUN apt-get update && apt-get install -y --no-install-recommends \
        ros-lyrical-desktop \
        ros-lyrical-rmw-cyclonedds-cpp \
        ros-dev-tools \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# install colcon and rosdep
RUN apt-get update && apt-get install -y --no-install-recommends \
        python3-colcon-common-extensions \
        python3-rosdep \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER $USERNAME
WORKDIR /home/$USERNAME
SHELL ["/bin/bash", "-l", "-c"]

RUN echo "export PATH=/usr/local/cuda/bin:$PATH" >> ~/.bashrc && \
    echo "export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH" >> ~/.bashrc

# initialize rosdep
RUN sudo rosdep init && \
    rosdep update

RUN echo "source /opt/ros/lyrical/setup.bash" >> ~/.bashrc
