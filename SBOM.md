# Software Bill of Materials (SBOM) for Flutter Dev Container

This document lists the software components included in the Dev Container environment defined by `.devcontainer/devcontainer.json` and `.devcontainer/Dockerfile`.

**Note:** Many components are installed using package managers or scripts that fetch the latest available version at the time the container image is built. Specific versions may vary depending on when the image was created.

## 1. Base Operating System

*   **OS:** Ubuntu
*   **Version:** 24.04 (LTS) - Upgraded from 22.04

## 2. System Packages (Installed via APT)

*   **Package Manager:** APT
*   **Components:** (Versions reflect those available in Ubuntu 24.04 repos at build time)
    *   `git`
    *   `sudo`
    *   `openjdk-17-jdk-headless`
    *   `wget`
    *   `curl`
    *   `zip`
    *   `unzip`
    *   `xz-utils`
    *   `ca-certificates`
    *   `lsb-release`
    *   `gnupg`
    *   `jq`
    *   `clang`
    *   `cmake`
    *   `ninja-build`
    *   `pkg-config`
    *   `libgtk-3-dev` (Note: Ubuntu 24.04 defaults towards GTK 4)
    *   `liblzma-dev`
    *   `build-essential`
    *   `procps`
    *   `file`
    *   `scrcpy`
    *   `git-lfs`
    *   `zsh`
    *   *Note: `apt-transport-https` removed as it's deprecated in Ubuntu 24.04.*

## 3. Tools Installed via Scripts/Direct Download

*   **Homebrew:** Latest version from `install.sh` script at build time.
*   **Firebase CLI:** Latest version from `firebase.tools` script at build time.
*   **Oh My Zsh:** Latest version from `install.sh` script at build time.

## 4. Tools Installed via Homebrew

*   **Package Manager:** Homebrew
*   **Components:**
    *   `supabase/tap/supabase`: Latest version available in the tap at build time.

## 5. Software Development Kits (SDKs)

*   **Android SDK:**
    *   **Installation Method:** Android SDK Manager (`sdkmanager`)
    *   **Components:** (Versions reflect targets for Android 15/API 35 as of March 2025)
        *   `cmdline-tools`: `latest` version available at build time (targeting >= 12.0).
        *   `platform-tools`: Latest version available at build time (targeting >= 35.0.2).
        *   `emulator`: Latest version available at build time (targeting >= 34.2.0).
        *   `platforms;android-35`: Platform API Level 35 (Android 15).
        *   `build-tools;35.0.0`: Specific version tied to platform version.
*   **Flutter SDK:**
    *   **Installation Method:** Git clone from GitHub.
    *   **Channel:** `stable`
    *   **Version:** Latest commit on the `stable` branch at the time of cloning (build time).

## 6. VS Code Extensions

*   **Installation Method:** VS Code Marketplace (via `devcontainer.json`)
*   **Components:** (Latest compatible versions installed by VS Code at container startup/build)
    *   `dart-code.dart-code`
    *   `dart-code.flutter`
    *   `googlecloudtools.cloudcode`
    *   `supabase.supabase`
