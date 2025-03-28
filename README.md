# Flutter (Future-Proofed Dev Container)

## Summary

*Develop Flutter based applications. Includes the Flutter SDK (latest stable), Android SDK (Platform 34), JDK 17, Zsh, Firebase CLI, Supabase CLI, scrcpy, Git LFS, Linux Desktop dependencies, and needed VS Code extensions.*

This definition provides a comprehensive environment for Flutter development, aiming for maintainability by using latest stable versions where feasible.

| Metadata | Value |
|----------|-------|
| *Contributors* | Lucas Hilleshein dos Santos, Cline |
| *Definition type* | Dockerfile |
| *Works in Codespaces* | Not tested |
| *Container host OS support* | Windows (using WSL2), macOS, Linux |
| *Languages, platforms* | Dart, Flutter |

### Topics
- [Flutter (Future-Proofed Dev Container)](#flutter-future-proofed-dev-container)
  - [Summary](#summary)
    - [Topics](#topics)
  - [Using this definition with an existing folder](#using-this-definition-with-an-existing-folder)
  - [Key Features \& Included Tooling](#key-features--included-tooling)
  - [Maintainability \& Updates](#maintainability--updates)
  - [Verifying the Setup](#verifying-the-setup)
  - [Android Development](#android-development)
    - [Physical Devices (All Host OS - Windows/Mac/Linux)](#physical-devices-all-host-os---windowsmaclinux)
    - [Connecting to Host Emulator (Windows/Mac/Linux Host)](#connecting-to-host-emulator-windowsmaclinux-host)
  - [Shell \& Aliases](#shell--aliases)
  - [License](#license)


## Using this definition with an existing folder

This definition does not require any special steps to use beyond the standard Dev Container setup.

1. If this is your first time using a development container, please follow the [getting started steps](https://aka.ms/vscode-remote/containers/getting-started) to set up your machine (Docker Desktop, VS Code, Remote-Containers extension).
2. Clone this repository or your fork locally.
3. Ensure you have a project folder where you want to use this definition.
4. Copy the `.devcontainer` folder from this repository into the root of your project folder.
5. Open your project folder in VS Code.
6. VS Code should prompt you to "Reopen in Container". Click it. (Alternatively, press <kbd>F1</kbd> and run **Remote-Containers: Reopen Folder in Container**).

## Key Features & Included Tooling

This container comes pre-configured with:

*   **Flutter SDK:** Latest commit from the `stable` channel (cloned via `git`).
*   **Android SDK:** Platform `34`, Build Tools `34.0.0`, latest `cmdline-tools` & `platform-tools`.
*   **Java Development Kit:** OpenJDK `17` (LTS).
*   **Firebase CLI:** Latest version (installed via official script).
*   **Supabase CLI:** Latest version (downloaded from GitHub releases).
*   **Zsh Shell:** Enhanced shell with Oh My Zsh installed.
*   **scrcpy:** Display and control Android devices connected via ADB.
*   **Git LFS:** For handling large files in Git repositories (system hooks installed).
*   **Linux Desktop Dependencies:** Pre-installed for building Flutter Linux apps (`clang`, `cmake`, `ninja`, `pkg-config`, `libgtk-3-dev`).
*   **VS Code Extensions:** Dart, Flutter, Cloud Code (Firebase/GCP), Supabase.
*   **Common Utilities:** `curl`, `wget`, `unzip`, `jq`, etc.

## Maintainability & Updates

The goal is to stay reasonably up-to-date with minimal manual intervention.

*   **Automatic Updates (on Container Rebuild):**
    *   **Flutter SDK:** Fetches the latest commit from the configured `FLUTTER_CHANNEL` (`stable` by default) in the Dockerfile.
    *   **Firebase CLI:** The install script fetches the latest version.
    *   **Supabase CLI:** The Dockerfile fetches the latest `.deb` release from GitHub.
    *   **Android `cmdline-tools`, `platform-tools`, `emulator`:** `sdkmanager --update` attempts to get the latest patch versions for installed components.
    *   **System Packages:** `apt update` fetches the latest versions available in the Ubuntu 22.04 repositories at build time.

*   **Manual Review Recommended Periodically:**
    *   **`.devcontainer/Dockerfile`:**
        *   `FROM ubuntu:22.04`: Consider updating the base image LTS version (e.g., to `24.04`) every ~2 years for newer system libraries and security updates.
        *   `openjdk-17-jdk-headless`: Update the JDK LTS version if required by future Android or Flutter versions.
        *   `ENV ANDROID_PLATFORM_VERSION=34`: Update the target Android platform version based on your project needs or new Android releases. The `ANDROID_BUILD_TOOLS_VERSION` will update automatically based on this.

## Verifying the Setup

Once the container is built and VS Code has connected (you should see a Zsh prompt like `~ %` in the integrated terminal):

1.  **Verify User:** Run `whoami`. Expected output: `vscode`.
2.  **Verify Shell:** Run `echo $SHELL`. Expected output: `/bin/zsh`.
3.  **Run Flutter Doctor:** Run `flutter doctor -v` (or use the alias `fd`).
    *   Check that `Flutter`, `Android toolchain`, `Chrome`, `Linux toolchain`, and `Android Studio` (even if not installed, it checks for the SDK) sections show reasonable paths and versions (e.g., `/home/vscode/flutter`, `/home/vscode/android-sdk-linux`).
    *   Check `Connected device` section (will likely show none initially).
4.  **Check CLI Versions:**
    *   `firebase --version`
    *   `supabase --version`
    *   `scrcpy --version`
    *   `git lfs version`
5.  **Test Aliases:** Type `fd` and press Enter (should run `flutter doctor`). Try `frc`, `supas`, etc.

## Android Development

Connecting Android devices or emulators requires using ADB (Android Debug Bridge) over the network, as direct USB passthrough is limited in Docker Desktop.

### Physical Devices (All Host OS - Windows/Mac/Linux)

The recommended method is ADB over TCP/IP. This requires `adb` installed on your **host** machine.

1.  **Install ADB on Host:** Download the [Android SDK Platform Tools](https://developer.android.com/studio/releases/platform-tools#downloads) for your host OS and add `adb` to your host's PATH.
2.  **Enable Debugging:** On your Android device, enable Developer Options and USB Debugging.
3.  **Connect via USB (Temporarily):** Connect your device to your host machine via USB.
4.  **Switch to TCP/IP Mode (Host Terminal):** Open a terminal *on your host machine* and run:
    ```bash
    # Verify host ADB sees the device via USB
    adb devices

    # Tell the device to listen for TCP/IP connections on port 5555
    adb tcpip 5555
    ```
5.  **Find Device IP:** Find your device's local network IP address (usually in Settings > Wi-Fi > Connected Network details, or Settings > About Phone > Status).
6.  **Connect via Wi-Fi (Host Terminal):**
    ```bash
    # Replace YOUR_DEVICE_IP with the actual IP found in the previous step
    adb connect YOUR_DEVICE_IP:5555

    # Verify the host ADB now sees the device via Wi-Fi (it might list both USB and Wi-Fi initially)
    adb devices
    ```
7.  **Disconnect USB:** You can now safely disconnect the USB cable. The Wi-Fi ADB connection should persist.
8.  **Connect from Container (VS Code Terminal):** Open the integrated terminal *inside the Dev Container* and run:
    ```bash
    # Ensure no old container ADB server is running
    adb kill-server

    # Connect to the device using the same IP address
    adb connect YOUR_DEVICE_IP:5555

    # Verify the container ADB sees the device
    adb devices

    # Check if Flutter recognizes the device
    flutter doctor
    ```
9.  You should now be able to select the device and run your app (`flutter run` or `frd`) and use tools like `scrcpy` from within the container.

### Connecting to Host Emulator (Windows/Mac/Linux Host)

Using an Android Emulator running directly on your host machine is often simpler.

1.  **Start Emulator on Host:** Launch your desired Android Emulator using Android Studio or the `emulator` command *on your host machine*. Wait for it to fully boot.
2.  **Connect from Container (VS Code Terminal):** Open the integrated terminal *inside the Dev Container* and run:
    ```bash
    # Ensure no old container ADB server is running
    adb kill-server

    # Connect to the host machine's ADB server. Docker Desktop provides the special DNS name 'host.docker.internal'
    # which resolves to the host's internal IP address. Port 5037 is the default ADB server port.
    adb connect host.docker.internal:5037

    # Verify the container ADB sees the emulator running on the host
    adb devices

    # Check if Flutter recognizes the emulator
    flutter doctor
    ```
3.  You should now be able to select the host emulator as a target device for `flutter run` (or `frd`).

## Shell & Aliases

*   The default shell in this container is **Zsh**, enhanced with **Oh My Zsh**.
*   The following convenient aliases are pre-configured in `~/.zshrc`:
    *   `fd="flutter doctor"`
    *   `frc="flutter run -d chrome"` (Run for Chrome)
    *   `frd="flutter run"` (Run for default device)
    *   `supas="supabase start"`
    *   `supastop="supabase stop"`

## License

Copyright (c) 2020-2025 Lucas Hilleshein dos Santos

Licensed under the MIT License. See [LICENSE](LICENSE) file for details.
