FROM osrf/ros:jazzy-desktop

ARG USER_NAME=rosuser
ARG USER_UID=1000
ARG USER_GID=1000

RUN set -eux; \
	existing_group="$(getent group "${USER_GID}" | cut -d: -f1 || true)"; \
	if [ -n "${existing_group}" ]; then \
		[ "${existing_group}" = "${USER_NAME}" ] || groupmod --new-name "${USER_NAME}" "${existing_group}"; \
	elif getent group "${USER_NAME}" >/dev/null; then \
		groupmod --gid "${USER_GID}" "${USER_NAME}"; \
	else \
		groupadd --gid "${USER_GID}" "${USER_NAME}"; \
	fi; \
	existing_user="$(getent passwd "${USER_UID}" | cut -d: -f1 || true)"; \
	if [ -n "${existing_user}" ]; then \
		if [ "${existing_user}" != "${USER_NAME}" ]; then \
			usermod --login "${USER_NAME}" --home "/home/${USER_NAME}" --move-home "${existing_user}"; \
		fi; \
		usermod --gid "${USER_GID}" --shell /bin/bash "${USER_NAME}"; \
	elif getent passwd "${USER_NAME}" >/dev/null; then \
		usermod --uid "${USER_UID}" --gid "${USER_GID}" --home "/home/${USER_NAME}" --shell /bin/bash "${USER_NAME}"; \
	else \
		useradd --no-log-init --create-home --uid "${USER_UID}" --gid "${USER_GID}" --shell /bin/bash "${USER_NAME}"; \
	fi; \
	usermod --append --groups sudo "${USER_NAME}"; \
	echo "${USER_NAME} ALL=(ALL) NOPASSWD:ALL" > "/etc/sudoers.d/${USER_NAME}"; \
	chmod 0440 "/etc/sudoers.d/${USER_NAME}"

RUN apt update && apt install -y ros-jazzy-ros2-control

USER $USER_NAME
RUN mkdir -p /home/${USER_NAME}/ros2_ws/src
WORKDIR /home/${USER_NAME}/ros2_ws
RUN git clone --recursive https://github.com/idra-lab/kuka_lbr_control.git src

RUN cd src/controllers

RUN cd /home/${USER_NAME}/ros2_ws
RUN rosdep update && rosdep install --from-paths src -i -r -y

RUN . "/opt/ros/${ROS_DISTRO}/setup.sh" && \
	colcon build --symlink-install
