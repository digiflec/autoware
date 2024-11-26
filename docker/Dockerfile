# Use ARG for base image
ARG BASE_IMAGE

# Base stage
FROM ${BASE_IMAGE} AS base
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Arguments for customization
ARG ROS_DISTRO
ARG SETUP_ARGS
ARG LIB_DIR

# Install apt packages and configure SSH for GitHub
RUN apt-get update && \
  DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  gosu ssh && \
  mkdir -p ~/.ssh && \
  ssh-keyscan github.com >> ~/.ssh/known_hosts && \
  apt-get clean && rm -rf /var/lib/apt/lists/* "$HOME/.cache"

# Copy necessary files
COPY setup-dev-env.sh ansible-galaxy-requirements.yaml amd64.env arm64.env /autoware/
COPY ansible/ /autoware/ansible/
WORKDIR /autoware

# Setup base environment
RUN --mount=type=ssh \
  ./setup-dev-env.sh -y --module all --no-cuda-drivers openadkit \
  && pip uninstall -y ansible ansible-core \
  && apt-get autoremove -y && rm -rf "$HOME"/.cache && \
  rm -rf /var/lib/apt/lists/*
# Copy application source code
COPY src/ /autoware/src

# Install ROS dependencies
RUN apt-get update && \
  rosdep update && \
  rosdep install --from-paths src --ignore-src -y && \
  apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Ouster dependencies 
RUN apt-get update && \
  apt-get install -y --no-install-recommends \
  vim libflatbuffers-dev libtins-dev libpcap-dev libglfw3-dev libglew-dev \
  libspdlog-dev htop && \
  apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Python packages
RUN pip install boto3 botocore

# Downgrade Numpy for MIM Packages
RUN pip install numpy==1.26.4

# MiM Dependencies
RUN pip install torch==2.1.0 torchvision==0.16.0 --index-url https://download.pytorch.org/whl/cu121
RUN pip install -U openmim

RUN mim install mmengine && \
    mim install 'mmcv<2.2.0' && \
    mim install 'mmdet>=3.0.0'

# MMDetection3d
RUN cd / && \
    git clone https://github.com/open-mmlab/mmdetection3d.git -b dev-1.x && \
    cd mmdetection3d && \
    pip install -v -e .

# Cleanup unnecessary files and optimize the image
# RUN --mount=type=ssh \
#   --mount=type=cache,target=/var/cache/apt,sharing=locked \
#   pip uninstall -y ansible ansible-core && \
#   apt-get update && \
#   # apt-get autoremove -y && \
#   rm -rf "$HOME/.cache" && \
#   rm -rf /autoware/ansible /autoware/ansible-galaxy-requirements.yaml \
#          /autoware/setup-dev-env.sh /autoware/*.env \
#          /root/.local/pipx \
#          /etc/apt/sources.list.d/docker.list \

RUN mkdir /workspace

WORKDIR /workspace

RUN rm -r /autoware

# Set up entrypoint
COPY docker/etc/ros_entrypoint.sh /ros_entrypoint.sh
RUN chmod +x /ros_entrypoint.sh

# Copy bash aliases
COPY docker/etc/.bash_aliases /root/.bash_aliases
# RUN echo "source /opt/autoware/setup.bash" >> /etc/bash.bashrc

# Define entrypoint and default command
ENTRYPOINT ["/ros_entrypoint.sh"]
CMD ["/bin/bash"]
