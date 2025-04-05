# Base Image: Use a stable LTS release
FROM ubuntu:24.04

# Set non-interactive frontend for apt installs
ENV DEBIAN_FRONTEND=noninteractive
# Set Locale
ENV LANG C.UTF-8

# --- Android SDK Path ---
# Set this ARG to the *actual* installation path used by the Android SDK Dev Container Feature.
# Verify this path from the feature's documentation or by inspecting the built container!
ARG ANDROID_SDK_ROOT_PATH=/opt/android-sdk # <-- EXAMPLE PATH, CHANGE IF NEEDED!

# Install essential packages NOT handled by features + Flutter Linux dependencies
# Common Utils feature often handles git, sudo, curl, build-essential, etc.
# Keep packages needed by Flutter or useful utilities like scrcpy.
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    wget \
    zip \
    unzip \
    xz-utils \
    ca-certificates \
    # Tools potentially needed for Flutter builds or diagnostics
    jq \
    # Flutter Linux Desktop build 
    curl \
    clang \
    cmake \
    ninja-build \
    pkg-config \
    libgtk-3-dev \
    liblzma-dev \
    # Other useful tools
    scrcpy \
# Clean up APT caches
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/*

# --- Manual Flutter SDK Installation ---
# Features handle user creation (vscode UID 1000 GID 1000 via common-utils)
# Install Flutter into the vscode user's home directory.

# Set Flutter ENV variables
ENV FLUTTER_CHANNEL="stable"
ENV FLUTTER_HOME=/home/vscode/flutter
# Add Flutter bin to the system PATH (useful even though .zshrc also adds it)
ENV PATH=${PATH}:${FLUTTER_HOME}/bin

# Clone Flutter, configure it, precache, and set permissions
# This RUN command executes before the final USER switch, potentially as root or buildkit user
RUN git clone --depth 1 --branch ${FLUTTER_CHANNEL} https://github.com/flutter/flutter.git ${FLUTTER_HOME} \
    # Disable Flutter analytics
    && ${FLUTTER_HOME}/bin/flutter config --no-analytics \
    # Enable Linux desktop builds (requires apt packages installed above)
    && ${FLUTTER_HOME}/bin/flutter config --enable-linux-desktop \
    # Download Flutter engine binaries
    && ${FLUTTER_HOME}/bin/flutter precache \
    # Ensure the 'vscode' user owns the Flutter installation
    # Create home dir structure if needed and set ownership BEFORE final USER switch
    && mkdir -p /home/vscode \
    && chown -R 1000:1000 ${FLUTTER_HOME} /home/vscode

# --- Final Container User ---
# The 'vscode' user is created by the common-utils feature.
# The 'remoteUser' setting in devcontainer.json handles the final switch.

# Optional: Set default working directory (often workspaceFolder is mounted here anyway)
# WORKDIR /home/vscode
