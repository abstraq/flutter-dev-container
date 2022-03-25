FROM debian:bullseye

# Non-Root User
ARG USER_UID=1000
ARG USER_GID=1000
ARG USER_NAME="codespaces"

# Android SDK
ARG ANDROID_CMD_TOOLS_VERSION=8092744
ARG ANDROID_PLATFORM_VERSION=30
ARG ANDROID_BUILD_TOOLS_VERSION=30.0.3
ENV ANDROID_HOME="/home/${USER_NAME}/android/sdk"
ENV PATH="$PATH:${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/emulator:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/platforms"

# Flutter SDK
ARG FLUTTER_CHANNEL="stable"
ARG FLUTTER_VERSION="2.10.3"
ENV FLUTTER_HOME="/home/${USER_NAME}/flutter"
ENV PATH="$PATH:${FLUTTER_HOME}/bin"


# Upgrade packages and install dependencies.
ARG DEBIAN_FRONTEND "noninteractive"
RUN  apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends openjdk-11-jdk curl unzip sed git bash xz-utils libglvnd0 ssh xauth x11-xserver-utils libpulse0 libxcomposite1 libgl1-mesa-glx sudo \
	&& apt-get autoremove -y \
	&& apt-get clean -y \
	&& rm -rf /var/lib/{apt,dpkg,cache,log}

# Create non-root user with sudo access.
RUN groupadd --gid ${USER_GID} ${USER_NAME} \
  && useradd -s /bin/bash --uid ${USER_UID} --gid ${USER_GID} -m ${USER_NAME} \
  && echo ${USER_NAME} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USER_NAME} \
  && chmod 0440 /etc/sudoers.d/${USER_NAME}

# Switch to non-root user
USER $USER_NAME

# Install Android SDK
RUN mkdir -p ${ANDROID_HOME} \
    && cd ${ANDROID_HOME} \
    && curl -C - --output android-cmd-tools.zip https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_CMD_TOOLS_VERSION}_latest.zip \
    && unzip -qq android-cmd-tools.zip \
    && rm android-cmd-tools.zip \
    && mkdir ${ANDROID_HOME}/cmdline-tools/latest \
    && mv ${ANDROID_HOME}/cmdline-tools/bin ${ANDROID_HOME}/cmdline-tools/latest/bin \
    && mv ${ANDROID_HOME}/cmdline-tools/lib ${ANDROID_HOME}/cmdline-tools/latest/lib \
    && yes | sdkmanager --licenses \
    && touch $HOME/.android/repositories.cfg \
    && sdkmanager "platform-tools" "platforms;android-${ANDROID_PLATFORM_VERSION}" "build-tools;${ANDROID_BUILD_TOOLS_VERSION}" "emulator"

# Install Flutter SDK
RUN cd $HOME \
	&& curl -C - --output flutter.tar.xz https://storage.googleapis.com/flutter_infra_release/releases/${FLUTTER_CHANNEL}/linux/flutter_linux_${FLUTTER_VERSION}-${FLUTTER_CHANNEL}.tar.xz \
	&& tar xf flutter.tar.xz -C . \
	&& rm flutter.tar.xz \
	&& yes | flutter doctor --android-licenses \
	&& flutter config --no-analytics \
	&& flutter update-packages
