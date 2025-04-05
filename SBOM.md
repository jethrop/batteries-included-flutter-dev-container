# Software Bill of Materials (SBOM) for Flutter Dev Container

This document lists the primary software components included in the Dev Container environment defined by `.devcontainer/devcontainer.json` and `.devcontainer/Dockerfile`.

**Note:** This environment heavily utilizes Dev Container Features. Default configurations often use `latest` versions, meaning the specific version installed depends on when the container image is built or rebuilt. Users can customize versions (especially for Java and CLIs) by modifying the `features` block in `.devcontainer/devcontainer.json` as described in the `README.md`.

## 1. Base Operating System

*   **OS:** Ubuntu
*   **Version:** 24.04 (LTS) - Upgraded from 22.04

## 2. Dev Container Features

*   **Installation Method:** Dev Container Features (via `.devcontainer/devcontainer.json`)
*   **Components & Default Configuration:** (Users can override defaults as noted in `README.md`)
    *   **`ghcr.io/devcontainers/features/common-utils:2`:**
        *   **Provides:** `git`, `sudo`, `curl`, `zsh`, Oh My Zsh, user setup (`vscode`), package upgrades.
        *   **Default:** Installs Zsh, OhMyZsh, upgrades packages.
    *   **`ghcr.io/devcontainers/features/git-lfs:1`:**
        *   **Provides:** Git Large File Storage client.
        *   **Default:** `latest` version, `autoPull=true`.
    *   **`ghcr.io/devcontainers/features/java:1`:**
        *   **Provides:** Java Development Kit.
        *   **Default:** `latest` version, `jdkDistro=ms` (Microsoft Build of OpenJDK).
        *   **Alternative:** Pinned JDK 21 commented out.
    *   **`ghcr.io/meaningful-ooo/devcontainer-features/homebrew:2`:**
        *   **Provides:** Homebrew package manager (primarily for Linux).
        *   **Default:** Installs latest Homebrew.
    *   **`ghcr.io/nordcominc/devcontainer-features/android-sdk:1`:**
        *   **Provides:** Android SDK Platform, Build Tools, Platform Tools, optionally Emulator.
        *   **Default:** Pinned `platforms=android-35`, `buildTools=35.0.0`, `installEmulator=true`, `installPlatformTools=true`.
        *   **Alternative:** Pinned Platform 34 commented out.
    *   **`ghcr.io/devcontainers-extra/features/firebase-cli:2`:**
        *   **Provides:** Firebase Command Line Interface.
        *   **Default:** `latest` version.
        *   **Alternative:** Pinned version commented out.
    *   **`ghcr.io/devcontainers-extra/features/supabase-cli:1`:**
        *   **Provides:** Supabase Command Line Interface.
        *   **Default:** `latest` version.
        *   **Alternative:** Pinned version commented out.

## 3. Manual Installations (via Dockerfile)

*   **Flutter SDK:**
    *   **Installation Method:** `git clone` from GitHub repository.
    *   **Channel:** `stable` (defined by `ENV FLUTTER_CHANNEL`).
    *   **Version:** Latest commit on the `stable` branch at the time of cloning (image build time).
    *   **Configuration:** Configured with Android SDK path, Linux desktop enabled, analytics disabled.
*   **System Packages (via APT):**
    *   **Purpose:** Essential dependencies for Flutter Linux builds or useful utilities not covered by features.
    *   **Components:** (Versions reflect those available in Ubuntu 24.04 repos at build time)
        *   `wget`
        *   `zip`
        *   `unzip`
        *   `xz-utils`
        *   `ca-certificates`
        *   `jq`
        *   `clang`
        *   `cmake`
        *   `ninja-build`
        *   `pkg-config`
        *   `libgtk-3-dev`
        *   `liblzma-dev`
        *   `scrcpy`

## 4. VS Code Extensions

*   **Installation Method:** VS Code Marketplace (via `devcontainer.json`)
*   **Components:** (Latest compatible versions installed by VS Code at container startup/build)
    *   `dart-code.dart-code`
    *   `dart-code.flutter`
    *   `googlecloudtools.cloudcode`
    *   `supabase.supabase`
