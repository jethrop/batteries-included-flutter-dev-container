**Goal:** Create a streamlined, maintainable Flutter dev container using Dev Container Features. Default to `latest` versions for future-proofing where practical (Java, CLIs), while allowing users to easily switch to pinned versions via comments in `devcontainer.json`. Android SDK remains pinned due to Flutter requirements, but allows switching between different pinned target sets. Flutter SDK installation remains manual (`git clone stable`).

**Core Principles:**

*   **Features First:** Delegate installation and configuration of common tools to features.
*   **Flexibility via Comments:** Use `devcontainer.json` comments to enable/disable pre-configured blocks for `latest` vs. `pinned` versions.
*   **`latest` Default:** Prioritize `latest` versions for non-critical components (Java, CLIs) as the default uncommented state.
*   **Android SDK Pinned:** Acknowledge Flutter's need for specific Android Platform/Build Tools; provide commented options for different *pinned* sets.
*   **Minimal Dockerfile:** Simplify the Dockerfile to handle only the base OS, essential `apt` packages, manual Flutter installation, and integration points (like the Android SDK path).

---

**Migration Plan:**

**Phase 1: Configure Features and Flexibility in `devcontainer.json`**

1.  **Structure `devcontainer.json`:** Open `.devcontainer/devcontainer.json`. Replace the existing `features` block and add comments to guide users on switching configurations. Keep other sections like `name`, `dockerFile`, `forwardPorts`, `remoteUser`, `customizations` (with `settings` and `extensions`), and `postCreateCommand`.

    ```json
    {
        "name": "Flutter (Batteries Included - Latest Default)",
        "dockerFile": "Dockerfile",

        // --- Ports, Remote User, VS Code Settings & Extensions ---
        "forwardPorts": [
            // 54323 // Default Supabase Studio port
        ],
        "remoteUser": "vscode",
        "customizations": {
            "vscode": {
                "settings": {
                    "terminal.integrated.shell.linux": "/bin/zsh",
                    // Ensure this path matches the manual Flutter install in Dockerfile
                    "dart.flutterSdkPath": "/home/vscode/flutter"
                },
                "extensions": [
                    "dart-code.dart-code",
                    "dart-code.flutter",
                    "googlecloudtools.cloudcode",
                    "supabase.supabase"
                ]
            }
        },

        // --- Dev Container Features Configuration ---
        // Instructions: To switch between 'Latest' and 'Pinned' versions for a tool:
        // 1. Find the tool's section below (e.g., Java, Firebase CLI).
        // 2. Comment out the entire block for the currently active configuration (e.g., /* ... */ or // per line).
        // 3. Uncomment the entire block for the desired configuration.
        // 4. Modify the 'version' in the Pinned block if needed.
        // 5. Save this file and Rebuild the container (Cmd/Ctrl+Shift+P -> Rebuild Container).
        "features": {
            // --- Git LFS: Usually safe with latest ---
            "ghcr.io/devcontainers/features/git-lfs:1": {
                "version": "latest", // Default
                "autoPull": true
            },

            // --- Java: Default to Latest, provide pinned JDK 21 as alternative ---
            // --- Latest Java (Default) ---
            "ghcr.io/devcontainers/features/java:1": {
                // "version": "latest", // Or omit version entirely for latest
                "jdkDistro": "ms"
            },
            /* --- Pinned Java (Example: JDK 21) ---
            "ghcr.io/devcontainers/features/java:1": {
                "version": "21",
                "jdkDistro": "ms"
            },
            */ // End Pinned Java

            // --- Homebrew: Always installs latest ---
            "ghcr.io/meaningful-ooo/devcontainer-features/homebrew:2": {},

            // --- Android SDK: Always Pinned (Required by Flutter). Default: Platform 35 ---
            //       Switch between different *pinned* targets using comments below.
            // --- Default Pinned Android SDK (Platform 35) ---
            "ghcr.io/nordcominc/devcontainer-features/android-sdk:1": {
                "platforms": "android-35",
                "buildTools": "35.0.0",
                "installEmulator": true, // Set true/false as needed
                "installPlatformTools": true
                // !! IMPORTANT: Verify the install path from this feature's documentation
                //    and ensure it matches the ARG in the Dockerfile !!
            },
            /* --- Alternative Pinned Android SDK (Example: Platform 34) ---
            "ghcr.io/nordcominc/devcontainer-features/android-sdk:1": {
                "platforms": "android-34",
                "buildTools": "34.0.0", // Check if build tools need adjustment for platform 34
                "installEmulator": true,
                "installPlatformTools": true
                // Verify the install path (likely the same as above)
            },
            */ // End Alternative Pinned Android

            // --- Firebase CLI: Default to Latest, provide pinned as alternative ---
            // --- Latest Firebase CLI (Default) ---
            "ghcr.io/devcontainers-extra/features/firebase-cli:2": {
                "version": "latest"
            },
            /* --- Pinned Firebase CLI (Example: Specific Version) ---
            "ghcr.io/devcontainers-extra/features/firebase-cli:2": {
                "version": "13.0.0" // Replace with desired pinned version
            },
            */ // End Pinned Firebase CLI

            // --- Supabase CLI: Default to Latest, provide pinned as alternative ---
            // --- Latest Supabase CLI (Default) ---
            "ghcr.io/devcontainers-extra/features/supabase-cli:1": {
                "version": "latest"
            },
            /* --- Pinned Supabase CLI (Example: Specific Version) ---
            "ghcr.io/devcontainers-extra/features/supabase-cli:1": {
                "version": "1.100.0" // Replace with desired pinned version
            },
            */ // End Pinned Supabase CLI

            // --- Common Utils: Handles Zsh, OhMyZsh, User Creation ---
            // Usually safe with latest major version tag.
            "ghcr.io/devcontainers/features/common-utils:2": {
                "installZsh": true,
                "installOhMyZsh": true,
                "installOhMyZshConfig": true,
                "upgradePackages": true, // Upgrades apt packages during feature install
                "username": "vscode",
                "userUid": "1000",
                "userGid": "1000"
            }
        }, // End of features block

        // --- Post Create Command: Runs after container creation and feature setup ---
        // Used here to add shell aliases reliably after OhMyZsh is configured.
        "postCreateCommand": "echo '\n# Custom Aliases (added via postCreateCommand)\nalias fd=\"flutter doctor\"\nalias frc=\"flutter run -d chrome\"\nalias frd=\"flutter run\"\nalias supas=\"supabase start\"\nalias supastop=\"supabase stop\"\n' >> ~/.zshrc"

    } // End of JSON
    ```

2.  **Verify Android SDK Path:** Determine the exact installation path used by the `ghcr.io/nordcominc/devcontainer-features/android-sdk:1` feature (check its documentation or inspect the container filesystem after a build). Common paths are `/opt/android-sdk` or `/usr/local/lib/android/sdk`. Note this path for the next phase.

**Phase 2: Simplify and Integrate the `Dockerfile`**

1.  **Modify Dockerfile:** Open `.devcontainer/Dockerfile`. Replace its content with the following simplified version. **Crucially, update the default value for `ARG ANDROID_SDK_ROOT_PATH` to the path you verified in Phase 1.**

    ```dockerfile
    # Base Image: Use a stable LTS release
    FROM ubuntu:24.04

    # Set non-interactive frontend for apt installs
    ENV DEBIAN_FRONTEND=noninteractive
    # Set Locale
    ENV LANG C.UTF-8

    # --- Android SDK Path ---
    # Set this ARG to the *actual* installation path used by the Android SDK Dev Container Feature.
    # Verify this path from the feature's documentation!
    ARG ANDROID_SDK_ROOT_PATH=/opt/android-sdk # <-- EXAMPLE PATH, CHANGE IF NEEDED!

    # Install essential packages NOT handled by features + Flutter Linux dependencies
    # Common Utils feature often handles git, sudo, curl, build-essential, etc.
    # Keep packages needed by Flutter or useful utilities like scrcpy.
    RUN apt-get update && apt-get install -y --no-install-recommends \
        wget \
        zip \
        unzip \
        xz-utils \
        ca-certificates \
        # Tools potentially needed for Flutter builds or diagnostics
        jq \
        # Flutter Linux Desktop build dependencies
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
        # Configure Flutter to use the Android SDK installed by the feature via the ARG
        && ${FLUTTER_HOME}/bin/flutter config --android-sdk "${ANDROID_SDK_ROOT_PATH}" \
        # Accept Android licenses (needs Android SDK configured first)
        && yes | ${FLUTTER_HOME}/bin/flutter doctor --android-licenses \
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
    # Switch to the non-root 'vscode' user created by the common-utils feature
    USER vscode

    # Optional: Set default working directory (often workspaceFolder is mounted here anyway)
    # WORKDIR /home/vscode
    ```

**Phase 3: Update Documentation**

1.  **Update `README.md`:**
    *   Change the title/summary to reflect the new approach (e.g., "Flutter Dev Container (Features-Based, Flexible Versions)").
    *   Rewrite the "Key Features & Included Tooling" section to list components installed via features (mentioning `latest` default for Java/CLIs, pinned Android) and the manually installed Flutter SDK.
    *   Add the new "Configuration: Latest vs. Pinned Versions" section (copy the markdown block from the previous response explaining how to use comments in `devcontainer.json`).
    *   Rewrite the "Maintainability & Updates" section:
        *   Explain updates now mainly happen via feature updates (on rebuild for `latest`, or by changing pins in `devcontainer.json`).
        *   Mention Flutter updates via `git clone` (rebuild) or `flutter upgrade` (inside container).
        *   Note base image updates require Dockerfile edit.
    *   Remove any outdated setup/verification steps specific to the old manual process. Add verification steps for checking feature-installed tools (`java --version`, `firebase --version`, etc.).

2.  **Update `SBOM.md`:**
    *   List the Dev Container Features used as the primary installation method for Java, Android SDK, CLIs, Git LFS, Zsh, Homebrew.
    *   Specify the *default* configuration (e.g., Java `latest`, Android Platform 35 pinned, Firebase `latest`). Add a note that users can override versions via `devcontainer.json`.
    *   List components still installed via `apt` (`scrcpy`, Linux deps, etc.).
    *   List Flutter SDK (installed via `git clone stable`, version reflects build time).

3.  **Update `upgradePlan.md`:** Mark it as **Archival**. Add a note at the top stating the project now uses a features-based approach configured via `.devcontainer/devcontainer.json` and the plan below is obsolete.

**Phase 4: Testing**

1.  **Save** all modified files (`.devcontainer/devcontainer.json`, `.devcontainer/Dockerfile`, `README.md`, `SBOM.md`, `upgradePlan.md`).
2.  **Test Default Configuration (Latest):**
    *   Run **Remote-Containers: Rebuild and Reopen in Container**. Check build logs for feature installation success.
    *   Verify: `whoami` (vscode), `echo $SHELL` (zsh), `java --version` (latest JDK from feature), `firebase --version` (latest), `supabase --version` (latest), `git lfs version`.
    *   Verify: `flutter doctor -v`. Check Flutter version (latest stable), **Android SDK path matches the ARG**, default Android Platform (e.g., 35) is recognized, Linux toolchain is OK.
    *   Test aliases (`fd`, `frc`, `supas`).
    *   Test core functionality (create/run Flutter app for web/linux/android).
3.  **Test Switching Mechanism:**
    *   **Pin Java:** Edit `devcontainer.json`, comment the "Latest Java" block, uncomment the "Pinned Java (Example: JDK 21)" block. Rebuild. Verify `java --version` shows JDK 21.
    *   **Switch Android Target:** Edit `devcontainer.json`, comment the "Default Pinned Android" block, uncomment the "Alternative Pinned Android" block. Rebuild. Verify `flutter doctor -v` recognizes the alternative Android Platform (e.g., 34).
    *   **(Optional)** Test pinning a CLI (e.g., Firebase) similarly.
    *   **Switch Back:** Revert `devcontainer.json` to the default (latest Java, default Android) and rebuild one last time to ensure it returns to the expected default state.

---

This comprehensive plan implements the desired features-based approach with built-in flexibility for version management, leveraging the strengths of Dev Container Features while retaining manual control over the Flutter SDK installation. Remember to carefully verify the Android SDK path provided by the specific feature you are using.