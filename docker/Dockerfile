ARG ROS_DISTRO=jazzy
FROM ubuntu:24.04

ARG ROS_DISTRO

SHELL ["/bin/bash", "-c"]

ARG USERNAME=ros
ARG USER_UID=1000
ARG USER_GID=1000

ENV DEBIAN_FRONTEND=noninteractive
ENV ROS_DISTRO=${ROS_DISTRO}
ENV WORKSPACE=/workspaces/kuka_lbr_control

RUN apt-get update && apt-get install -y --no-install-recommends \
    bash-completion \
    build-essential \
    cmake \
    curl \
    git \
    locales \
    lsb-release \
    gnupg \
    python3-pip \
    sudo \
    tzdata \
 && rm -rf /var/lib/apt/lists/*

RUN python3 -m pip install --break-system-packages --no-cache-dir --upgrade \
    colcon-common-extensions \
    rosdep \
    vcstool

RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
    -o /usr/share/keyrings/ros-archive-keyring.gpg \
 && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo ${UBUNTU_CODENAME}) main" \
    > /etc/apt/sources.list.d/ros2.list

RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-${ROS_DISTRO}-desktop \
 && rm -rf /var/lib/apt/lists/*

RUN locale-gen en_US en_US.UTF-8 && update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8

RUN actual_uid="${USER_UID}"; \
    while getent passwd "${actual_uid}" >/dev/null; do actual_uid=$((actual_uid + 1)); done; \
    actual_gid="${USER_GID}"; \
    while getent group "${actual_gid}" >/dev/null; do actual_gid=$((actual_gid + 1)); done; \
    groupadd --gid "${actual_gid}" "${USERNAME}"; \
    useradd --uid "${actual_uid}" --gid "${actual_gid}" --create-home --shell /bin/bash "${USERNAME}"; \
    usermod -aG sudo "${USERNAME}"; \
    echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" >/etc/sudoers.d/"${USERNAME}"; \
    chmod 0440 /etc/sudoers.d/"${USERNAME}"

RUN rosdep init 2>/dev/null || true && rosdep update

COPY . /tmp/kuka_lbr_control
RUN rosdep install --from-paths /tmp/kuka_lbr_control --ignore-src -r -y --rosdistro "${ROS_DISTRO}" \
 && rm -rf /tmp/kuka_lbr_control

COPY docker/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

WORKDIR ${WORKSPACE}

USER ${USERNAME}

ENTRYPOINT ["/entrypoint.sh"]
CMD ["bash"]