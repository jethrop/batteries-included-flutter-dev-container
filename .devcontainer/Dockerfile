# Phase 1: Base Image, Core Dependencies, System Tools
# Use a specific LTS version for stability. Plan to update manually every ~2 years.
FROM ubuntu:22.04

# Locale
ENV LANG C.UTF-8

# Args for non-root user (used later)
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Install base packages, JDK, tools, and prerequisites
# Install git first as it's needed for Flutter install later
RUN apt update && apt install -y --no-install-recommends git && \
    apt install -y --no-install-recommends \
       sudo \
       # --- Use recent LTS JDK (e.g., 17) ---
       openjdk-17-jdk-headless \
       # --- Core Build/Dev Tools & Utilities ---
       wget curl zip unzip xz-utils ca-certificates apt-transport-https lsb-release gnupg \
       # --- Parser for GitHub API ---
       jq \
       # --- Linux Desktop Build Dependencies (Optional but good for Flutter) ---
       clang cmake ninja-build pkg-config libgtk-3-dev liblzma-dev \
       # --- Added Tooling ---
       scrcpy \
       git-lfs \
       zsh \
    # --- Install Git LFS system hooks ---
    && git lfs install --system \
    # --- Clean Up ---
    && apt-get purge --auto-remove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# --- Create non-root user and set Zsh as default shell ---
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/zsh --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Phase 2: Dynamic CLI Installations
# --- Install latest Supabase CLI dynamically ---
RUN LATEST_RELEASE_URL=$(curl -sL "https://api.github.com/repos/supabase/cli/releases/latest" | jq -r ".assets[] | select(.name | test(\"linux_amd64.deb$\")) | .browser_download_url") \
    && echo "Downloading Supabase CLI from: $LATEST_RELEASE_URL" \
    && wget --quiet "$LATEST_RELEASE_URL" -O supabase_latest_linux_amd64.deb \
    && apt update && apt install -y ./supabase_latest_linux_amd64.deb \
    && rm supabase_latest_linux_amd64.deb \
    && apt-get clean && rm -rf /var/lib/apt/lists/* # Clean up apt cache again

# --- Install latest Firebase CLI ---
RUN curl -sL https://firebase.tools | bash

# Phase 3: User Environment Setup
# --- Switch to non-root user for user-specific setup ---
USER $USERNAME
WORKDIR /home/$USERNAME

# --- Install Oh My Zsh non-interactively ---
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# --- Add custom aliases and environment settings to .zshrc ---
RUN echo '\n# Custom Aliases & Settings\n\
alias fd="flutter doctor"\n\
alias frc="flutter run -d chrome"\n\
alias frd="flutter run"\n\
alias supas="supabase start"\n\
alias supastop="supabase stop"\n\
\n\
# Ensure Flutter SDK path is recognized by Zsh plugins (if any)\n\
export FLUTTER_HOME=/home/vscode/flutter\n\
export PATH=$FLUTTER_HOME/bin:$PATH\n\
' >> /home/$USERNAME/.zshrc

# Phase 4: SDK Installations (as User)
# --- Android SDK Configuration ---
# !! IMPORTANT: Review and update ANDROID_PLATFORM_VERSION periodically !!
ENV ANDROID_PLATFORM_VERSION=34
# Calculate Build Tools version based on Platform version for consistency
ENV ANDROID_BUILD_TOOLS_VERSION=${ANDROID_PLATFORM_VERSION}.0.0
# Install SDK in user's home directory
ENV ANDROID_HOME=/home/$USERNAME/android-sdk-linux
ENV ANDROID_SDK_ROOT="$ANDROID_HOME"
# Add SDK tools to PATH
ENV PATH=${PATH}:${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/emulator

# --- Install Android SDK (as user) ---
# USER $USERNAME # Already running as user
# WORKDIR /home/$USERNAME # Already in user home
RUN mkdir -p ${ANDROID_HOME} \
    # Accept licenses FIRST
    && yes | sdkmanager --licenses --sdk_root=${ANDROID_HOME} \
    # Install latest tools, emulator, and specific platform/build-tools
    && sdkmanager --install "cmdline-tools;latest" "platform-tools" "emulator" \
       "platforms;android-${ANDROID_PLATFORM_VERSION}" "build-tools;${ANDROID_BUILD_TOOLS_VERSION}" \
       --sdk_root=${ANDROID_HOME} \
    # Optional: Update all installed packages to latest patch versions
    && sdkmanager --update --sdk_root=${ANDROID_HOME}

# --- Flutter SDK Configuration ---
# Set desired channel: stable, beta, master
ENV FLUTTER_CHANNEL="stable"
ENV FLUTTER_HOME=/home/$USERNAME/flutter
# Note: PATH for Flutter was added to .zshrc earlier, but setting ENV var is good practice too
ENV PATH=${PATH}:${FLUTTER_HOME}/bin

# --- Install Flutter SDK via Git clone (as user) ---
# USER $USERNAME # Already user
# WORKDIR /home/$USERNAME # Already in user home
RUN git clone --depth 1 --branch ${FLUTTER_CHANNEL} https://github.com/flutter/flutter.git ${FLUTTER_HOME} \
    && flutter config --android-sdk "${ANDROID_SDK_ROOT}" \
    # Accept licenses AFTER configuring android SDK path
    && yes | flutter doctor --android-licenses \
    && flutter config --no-analytics \
    # Enable Linux desktop builds (if dependencies installed earlier)
    && flutter config --enable-linux-desktop \
    # Download development binaries for host platform and configured targets
    && flutter precache \
    # Ensure project dependencies are up-to-date (less critical here, more for project)
    && flutter update-packages

# Optional: Switch back to root if needed for final cleanup/commands (unlikely)
# USER root
# WORKDIR /
