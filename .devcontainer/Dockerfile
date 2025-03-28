# Phase 1: Base Image, Core Dependencies, System Tools
# Use a specific LTS version for stability. Plan to update manually every ~2 years.
FROM ubuntu:24.04

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
       # --- Use recent LTS JDK (e.g., 21) ---
       openjdk-21-jdk-headless \
       # --- Core Build/Dev Tools & Utilities ---
       # Note: apt-transport-https is deprecated in 24.04+
       wget curl zip unzip xz-utils ca-certificates lsb-release gnupg \
       # --- Parser for GitHub API (still needed for other potential dynamic fetches) ---
       jq \
       # --- Linux Desktop Build Dependencies (Optional but good for Flutter) ---
       clang cmake ninja-build pkg-config libgtk-3-dev liblzma-dev \
       # --- Prerequisites for Homebrew ---
       build-essential procps curl file \
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

# --- Install Homebrew (as root, non-interactive) ---
# Installs under /home/linuxbrew/.linuxbrew
# Needs user interaction simulation ('< /dev/null') and prerequisites installed above
RUN /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" < /dev/null

# --- Add Homebrew to sudoers path (needed for brew installs that require sudo) ---
# Although we install as user later, some formulae might need sudo during install
RUN echo 'Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin:/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin"' > /etc/sudoers.d/secure_path_brew

# Phase 2: Dynamic CLI Installations (Supabase moved to Phase 3/User setup)
# --- Install latest Firebase CLI ---
RUN curl -sL https://firebase.tools | bash

# Phase 3: User Environment Setup
# --- Switch to non-root user for user-specific setup ---
USER $USERNAME
WORKDIR /home/$USERNAME

# --- Install Oh My Zsh non-interactively ---
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# --- Add Homebrew setup, custom aliases, and environment settings to .zshrc ---
# Note: Adding to .zshrc ensures it's available in interactive shells.
# .zprofile is typically for login shells, but Zsh might source it depending on config. Adding to both is safer.
RUN echo '\n# Setup Homebrew\n\
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"\n\
\n# Custom Aliases & Settings\n\
alias fd="flutter doctor"\n\
alias frc="flutter run -d chrome"\n\
alias frd="flutter run"\n\
alias supas="supabase start"\n\
alias supastop="supabase stop"\n\
\n\
# Ensure Flutter SDK path is recognized by Zsh plugins (if any)\n\
export FLUTTER_HOME=/home/vscode/flutter\n\
export PATH=$FLUTTER_HOME/bin:$PATH\n\
' >> /home/$USERNAME/.zshrc \
    && echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/$USERNAME/.zprofile

# --- Install latest Supabase CLI via Homebrew (as user) ---
# Needs to run after Homebrew PATH is configured for the user shell
# Source the brew environment variables before running the install command
# Cache bust: 2025-03-28 15:41:00 MDT
RUN eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)" && \
    /home/linuxbrew/.linuxbrew/bin/brew install supabase/tap/supabase

# Phase 4: SDK Installations (as User)
# --- Android SDK Configuration ---
# !! IMPORTANT: Review and update ANDROID_PLATFORM_VERSION periodically !!
ENV ANDROID_PLATFORM_VERSION=35
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
    # Optional: Update all installed packages to latest patch versions (already done by installing latest above)
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
    #&& flutter update-packages
# Optional: Switch back to root if needed for final cleanup/commands (unlikely)
# USER root
# WORKDIR /