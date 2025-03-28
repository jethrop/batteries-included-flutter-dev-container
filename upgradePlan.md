**Goal:** Create a secure, up-to-date, feature-rich, consistent, and *maintainable* Flutter development container environment that automatically uses recent stable versions of key tools where feasible.

**Key "Future-Proofing" Strategies:**
*   **Base Image:** Use a specific Ubuntu LTS version (e.g., `22.04`) for stability. *Requires manual update every ~2 years.*
*   **JDK:** Use a specific OpenJDK LTS version (e.g., `17`). *Requires manual update periodically for major Android compatibility shifts.*
*   **Android SDK:**
    *   Pin `ANDROID_PLATFORM_VERSION`. *Requires periodic user review/update.*
    *   Derive `ANDROID_BUILD_TOOLS_VERSION` from the platform version for consistency.
    *   Install `cmdline-tools;latest` using `sdkmanager` to get the newest tools.
*   **Flutter SDK:** Clone the desired channel (`stable`, `beta`) directly using `git` to get the latest commit on that branch at build time.
*   **Firebase CLI:** Use the official install script (`curl ... | bash`), which fetches the latest version.
*   **Supabase CLI:** Fetch the latest release asset URL from the GitHub API dynamically using `curl` and `jq`.
*   **Other Tools (scrcpy, git-lfs, zsh):** Install via `apt` from the Ubuntu LTS repository. `apt update` ensures the latest available *within that repo* are used.

**Prerequisites:**
*   Git installed.
*   VS Code installed with the "Remote - Containers" extension.
*   Docker Desktop for Windows installed and running, using the WSL 2 backend.
*   Your project (`lucashilles-flutter-dev-container.git` or your fork) cloned locally.

---

**Unified Step-by-Step Plan (Future-Proofed):**

**Phase 1: Update Dockerfile - Base Image, Core Dependencies, System Tools [COMPLETED]**

1.  **Set Base Image, Install Dependencies & System Tools: [COMPLETED]**
    *   **File:** `.devcontainer/Dockerfile`
    *   **Action:** Specify Ubuntu LTS base. Install LTS JDK. Install essential build tools, `git`, enhanced shell (`zsh`), helpful utilities (`scrcpy`, `git-lfs`), and prerequisites for dynamic CLI installs (`jq`). Install Git LFS system-wide hooks immediately after package install.
    *   **Changes:**
        ```dockerfile
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
        ```
    *   **Why:** Sets a stable base, installs necessary system-wide tools and libraries. `git lfs install --system` ensures LFS is active. Creates user with Zsh default.

**Phase 2: Update Dockerfile - Dynamic CLI Installations [COMPLETED]**

2.  **Install Latest Supabase & Firebase CLIs: [COMPLETED]**
    *   **File:** `.devcontainer/Dockerfile`
    *   **Action:** Add `RUN` commands (still as `root`) to dynamically fetch and install the latest Supabase CLI from GitHub releases and the latest Firebase CLI using its official script.
    *   **Additions (Insert after user creation block):**
        ```dockerfile
        # --- Install latest Supabase CLI dynamically ---
        RUN LATEST_RELEASE_URL=$(curl -sL "https://api.github.com/repos/supabase/cli/releases/latest" | jq -r ".assets[] | select(.name | test(\"linux_amd64.deb$\")) | .browser_download_url") \
            && echo "Downloading Supabase CLI from: $LATEST_RELEASE_URL" \
            && wget --quiet "$LATEST_RELEASE_URL" -O supabase_latest_linux_amd64.deb \
            && apt update && apt install -y ./supabase_latest_linux_amd64.deb \
            && rm supabase_latest_linux_amd64.deb \
            && apt-get clean && rm -rf /var/lib/apt/lists/* # Clean up apt cache again

        # --- Install latest Firebase CLI ---
        RUN curl -sL https://firebase.tools | bash
        ```
    *   **Why:** Installs the most recent versions of these CLIs available at build time without hardcoding version numbers.

**Phase 3: Update Dockerfile - User Environment Setup [COMPLETED]**

3.  **Install Oh My Zsh & Configure User Environment: [COMPLETED]**
    *   **File:** `.devcontainer/Dockerfile`
    *   **Action:** Switch to the `vscode` user. Install Oh My Zsh non-interactively. Add custom aliases to `.zshrc`.
    *   **Additions (Insert after CLI installs):**
        ```dockerfile
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

        # Note: Android SDK paths will be set via ENV vars below, Zsh will pick them up
        ```
    *   **Why:** Sets up the user's enhanced shell environment with helpful aliases and ensures the Flutter path is explicitly available if needed by shell plugins.

**Phase 4: Update Dockerfile - SDK Installations (as User) [COMPLETED]**

4.  **Configure & Install Android SDK: [COMPLETED]**
    *   **File:** `.devcontainer/Dockerfile`
    *   **Action:** Define Android `ENV` vars (pinning only the platform version). Install latest tools and specific platform/build-tools using `sdkmanager` *as the `vscode` user* into their home directory.
    *   **Changes:**
        ```dockerfile
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
            # AVD creation removed for container simplicity/reliability
        ```
    *   **Why:** Gets latest tooling, requires minimal user intervention (just platform API level). Installs with correct user permissions.

5.  **Install Latest Flutter SDK from Channel Branch: [COMPLETED]**
    *   **File:** `.devcontainer/Dockerfile`
    *   **Action:** Define Flutter channel `ENV` var. Clone the Flutter repo directly from the specified channel branch using `git` *as the `vscode` user*. Configure Flutter.
    *   **Changes:**
        ```dockerfile
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
        ```
    *   **Why:** Automatically uses the latest commit from the chosen Flutter channel at build time. Installs with correct user permissions.

**Phase 5: Update devcontainer.json Configuration [COMPLETED]**

6.  **Configure User, Shell, Extensions, Ports, Settings: [COMPLETED]**
    *   **File:** `.devcontainer/devcontainer.json`
    *   **Action:** Ensure `remoteUser` is `vscode`. Set default shell to `zsh`. Remove `postCreateCommand`. Add necessary extensions. Suggest relevant port forwards. Add setting for Flutter SDK path.
    *   **Changes:**
        ```json
        {
        	"name": "Flutter (Future-Proofed)",
        	"dockerFile": "Dockerfile",

        	// Configure container settings
        	"settings": {
        		// Set Zsh as the default integrated shell
        		"terminal.integrated.shell.linux": "/bin/zsh",
                // Help VS Code find the Flutter SDK path inside container
        		"dart.flutterSdkPath": "/home/vscode/flutter"
        	},

        	// VS Code extensions to install inside the container
        	"extensions": [
        		"dart-code.dart-code",          // Dart language support
        		"dart-code.flutter",            // Flutter specific features
                "googlecloudtools.cloudcode",   // Firebase/GCP integration
                "supabase.supabase"             // Supabase official extension
        	],

            // Ports to forward from container to host (uncomment/adjust as needed)
            "forwardPorts": [
                // 54323, // Default Supabase Studio port
                // Common Flutter web dev ports can be added if known
            ],
            // Optional: Open forwarded ports in browser automatically
            // "portsAttributes": {
            //     "54323": { "label": "Supabase Studio" }
            // },
            // "onAutoForwardPort": "openBrowser",

        	// Ensure VS Code connects as the non-root user 'vscode'
        	"remoteUser": "vscode",

            // --- postCreateCommand REMOVED ---

            // Optional: Run commands after container is created (e.g., final checks)
            // "postStartCommand": "flutter doctor -v"
        }
        ```
    *   **Why:** Configures VS Code for the container environment, installs necessary extensions, uses the secure non-root user, and removes potentially disruptive automatic commands. Explicitly setting `dart.flutterSdkPath` improves reliability.

**Phase 6: Update Documentation (README & LICENSE) [COMPLETED]**

7.  **Update LICENSE: [COMPLETED]**
    *   **File:** `LICENSE`
    *   **Action:** Update the copyright year (e.g., `2024` or `2020-2024`).

8.  **Revise README.md: [COMPLETED]**
    *   **File:** `README.md`
    *   **Action:**
        *   Update summary to mention key included tools (Zsh, Firebase CLI, Supabase CLI, scrcpy, Linux Desktop deps).
        *   **Remove** outdated/misleading sections ("Container settings" with ENV vars, "Testing the definition").
        *   **Add** "Key Features & Included Tooling" section listing the main components and their purpose (mentioning dynamic updates where applicable).
        *   **Add** "Maintainability & Updates" section explaining:
            *   What updates automatically (Flutter channel, CLIs via scripts/API).
            *   What requires periodic manual review in `Dockerfile` (`FROM ubuntu:...`, `openjdk-...`, `ANDROID_PLATFORM_VERSION`).
        *   **Add** "Verifying the Setup" section (run `flutter doctor -v`, `firebase --version`, `supabase --version`, `scrcpy --version`).
        *   **Revise** "Android Development" section:
            *   Clarify Win/Mac TCP/IP complexity.
            *   **Add** clear instructions for "Connecting to Host Emulator" (run on host, connect container ADB via `host.docker.internal:5037` or host IP).
        *   Mention Zsh is the default shell and list custom aliases.
        *   Update copyright year if present.
    *   **Why:** Provides accurate, current documentation focused on the enhanced features and maintainability aspects, guiding the user on verification and potential manual updates.

**Phase 7: Testing**

9.  **Rebuild and Test Thoroughly:**
    *   **Action:**
        *   Save all modified files.
        *   In VS Code, run **Remote-Containers: Rebuild and Reopen in Container**.
        *   Once connected (check for Zsh prompt):
            *   Verify user (`whoami` -> `vscode`). Verify shell (`echo $SHELL` -> `/usr/bin/zsh`).
            *   Verify tools: `flutter doctor -v` (check SDK paths), `firebase --version`, `supabase --version`, `scrcpy --version`, `git lfs version`. Test aliases (`fd`, `supas`).
            *   Manually create a test project: `flutter create my_test_app && cd my_test_app`.
            *   Test Web: `flutter run -d chrome` (or `frc`).
            *   Test Linux Desktop (if host supports): `flutter run -d linux`.
            *   Test Android (Physical Device): Follow updated README TCP/IP steps. Check `adb devices`, then `flutter run`. Try `scrcpy`.
            *   Test Android (Host Emulator): Follow new README steps. Check `adb devices`, `flutter run`.
            *   Test Supabase: `supabase init`, `supabase start` (check Studio access via forwarded port if configured), `supabase stop`.
    *   **Why:** Confirms all components are installed correctly, dynamic fetching works, tools integrate, SDKs function, permissions are correct, and various development workflows are operational.

---

This comprehensive guide should result in a robust, feature-rich, and significantly more maintainable Flutter dev container, minimizing version pinning while ensuring a consistent and up-to-date development experience. Remember to commit your changes regularly.
