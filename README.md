# Flutter Dev Container (Features-Based, Flexible Versions)

IN TESTING!!
## Summary

*Develop Flutter applications using a Dev Container powered by Features. Includes Flutter SDK (manual install, stable), Android SDK (pinned via feature), Java (latest/pinned via feature), Zsh, Firebase CLI (latest/pinned via feature), Supabase CLI (latest/pinned via feature), scrcpy, Git LFS, Linux Desktop dependencies, and needed VS Code extensions.*

This definition provides a flexible and maintainable environment for Flutter development by leveraging Dev Container Features for common tooling, while keeping the Flutter SDK installation manual for precise control. It defaults to `latest` versions for Java and CLIs but allows easy switching to pinned versions via `devcontainer.json`.

| Metadata | Value |
|----------|-------|
| *Contributors* | Lucas Hilleshein dos Santos, Cline |
| *Definition type* | Dockerfile |
| *Works in Codespaces* | Not tested |
| *Container host OS support* | Windows (using WSL2), macOS, Linux |
| *Languages, platforms* | Dart, Flutter |

### Topics
- [Flutter Dev Container (Features-Based, Flexible Versions)](#flutter-dev-container-features-based-flexible-versions)
  - [Summary](#summary)
    - [Topics](#topics)
  - [Using this definition with an existing folder](#using-this-definition-with-an-existing-folder)
  - [Key Features \& Included Tooling](#key-features--included-tooling)
  - [Configuration: Latest vs. Pinned Versions](#configuration-latest-vs-pinned-versions)
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

This container utilizes Dev Container Features for installing and configuring most tools, providing flexibility and easier updates.

*   **Installed via Dev Container Features (Configurable in `.devcontainer/devcontainer.json`):**
    *   **Java Development Kit:** Defaults to `latest` OpenJDK from Microsoft. Option to switch to pinned JDK 21 (or others) via comments.
    *   **Android SDK:** Pinned versions (required by Flutter). Defaults to Platform `35` / Build Tools `35.0.0`. Option to switch to other pinned sets (e.g., Platform 34) via comments. Includes platform-tools and optionally the emulator. *The SDK path (`ARG ANDROID_SDK_ROOT_PATH`) in the Dockerfile should match the feature's install location (e.g., /opt/android-sdk) and is used by the `postCreateCommand` to configure Flutter.*
    *   **Firebase CLI:** Defaults to `latest`. Option to switch to a pinned version via comments.
    *   **Supabase CLI:** Defaults to `latest`. Option to switch to a pinned version via comments.
    *   **Git LFS:** `latest` version, auto-pull enabled.
    *   **Homebrew:** `latest` version (used by some features like Supabase CLI).
    *   **Common Utilities:** Installs Zsh, Oh My Zsh, creates the `vscode` user, and handles basic package upgrades (`git`, `curl`, `sudo`, etc.).
*   **Installed Manually (in `.devcontainer/Dockerfile`):**
    *   **Flutter SDK:** Cloned from the `stable` channel via `git` during build time. Path: `/home/vscode/flutter`.
    *   **Essential Linux Packages:** `apt` packages needed for Flutter Linux builds (`clang`, `cmake`, `ninja-build`, `pkg-config`, `libgtk-3-dev`, `liblzma-dev`) and utilities like `scrcpy`, `wget`, `zip`, `unzip`, `jq`.
*   **VS Code Extensions:** Dart, Flutter, Cloud Code (Firebase/GCP), Supabase.

## Configuration: Latest vs. Pinned Versions

This Dev Container is designed for flexibility. You can easily switch between using the `latest` available versions of tools like Java, Firebase CLI, and Supabase CLI, or using specific `pinned` versions. This is controlled by commenting and uncommenting blocks within the `.devcontainer/devcontainer.json` file.

**How to Switch:**

1.  **Open:** `.devcontainer/devcontainer.json`
2.  **Locate:** Find the tool you want to configure (e.g., `"// --- Java: ..."`).
3.  **Comment Out:** Add `/*` before and `*/` after the entire JSON block for the currently *active* configuration (e.g., the "Latest Java" block if it's not commented).
4.  **Uncomment:** Remove the `/*` and `*/` surrounding the JSON block for the *desired* configuration (e.g., the "Pinned Java" block).
5.  **Adjust Pinned Version (Optional):** If you uncommented a "Pinned" block, you can change the `"version"` value within that block to the specific version you need.
6.  **Rebuild:** Save the `devcontainer.json` file. Use the Command Palette (<kbd>Cmd/Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>P</kbd>) and run **Remote-Containers: Rebuild Container**.

**Example: Switching Java from Latest to Pinned JDK 21**

*   **Find:** The Java section in `devcontainer.json`.
*   **Comment Latest:**
    ```json
    /* --- Latest Java (Default) ---
    "ghcr.io/devcontainers/features/java:1": {
        // "version": "latest",
        "jdkDistro": "ms"
    },
    */ // End Latest Java
    ```
*   **Uncomment Pinned:**
    ```json
    --- Pinned Java (Example: JDK 21) ---
    "ghcr.io/devcontainers/features/java:1": {
        "version": "21",
        "jdkDistro": "ms"
    },
    // End Pinned Java
    ```
*   **Rebuild:** Save and rebuild the container. `java --version` inside the container should now show JDK 21.

**Note on Android SDK:** The Android SDK is always pinned due to Flutter's requirements. However, you can switch between *different sets* of pinned Platform/Build Tools versions using the same commenting method described above.

## Maintainability & Updates

This features-based approach simplifies updates for most components.

*   **Dev Container Feature Updates (on Container Rebuild):**
    *   **Tools set to `latest` (Java, Firebase CLI, Supabase CLI, Git LFS, Homebrew, Common Utils):** These features will automatically pull their respective latest versions whenever the container is rebuilt (e.g., after changes to `devcontainer.json` or the Dockerfile, or via the **Remote-Containers: Rebuild Container** command).
    *   **Tools set to `pinned` versions (Android SDK, optionally Java/CLIs):** Updates require manually changing the version number within the corresponding commented-out or active block in `devcontainer.json` and then rebuilding the container. Check the feature's documentation or repository for available pinned versions.
*   **Manual Flutter SDK Updates:**
    *   **On Rebuild:** The Dockerfile clones the latest commit from the `stable` channel (`git clone --depth 1 --branch stable ...`) every time the container image is rebuilt from scratch. Flutter is configured to use the feature-installed Android SDK via the `postCreateCommand` after the container starts and features are installed.
    *   **Inside Container:** You can update Flutter manually within a running container by executing `flutter upgrade` in the terminal. This change will persist for the current container session but will be overwritten on the next full rebuild unless you commit the changes (not generally recommended for the SDK itself).
*   **Base Image Updates:**
    *   Updating the base Ubuntu OS (e.g., from `ubuntu:24.04` to `ubuntu:26.04`) requires editing the `FROM` line in the `.devcontainer/Dockerfile` and rebuilding.
*   **APT Package Updates:**
    *   The `common-utils` feature runs `apt-get upgrade` during its installation.
    *   Essential packages listed in the Dockerfile (`scrcpy`, Flutter Linux deps, etc.) are installed via `apt-get install` during the image build. Their versions depend on the Ubuntu repositories at build time. You can manually run `sudo apt update && sudo apt upgrade` inside the container for temporary updates.

## Verifying the Setup

Once the container is built and VS Code has connected (you should see a Zsh prompt like `~ %` in the integrated terminal):

1.  **Verify User:** Run `whoami`. Expected output: `vscode`.
2.  **Verify Shell:** Run `echo $SHELL`. Expected output: `/bin/zsh`.
3.  **Check Feature-Installed Tool Versions:**
    *   `java --version` (Should show the version corresponding to the active block in `devcontainer.json` - latest by default, or pinned e.g., 21)
    *   `firebase --version` (Should show latest or pinned version)
    *   `supabase --version` (Should show latest or pinned version)
    *   `git lfs version` (Should show latest version)
    *   `brew --version` (Should show latest Homebrew version)
4.  **Run Flutter Doctor:** Run `flutter doctor -v` (or use the alias `fd`).
    *   Check `Flutter` version (should be latest stable from build time).
    *   Check `Android toolchain` section:
        *   Verify the `Android SDK at` path matches the `ANDROID_SDK_ROOT_PATH` argument set in the `.devcontainer/Dockerfile` (e.g., `/opt/android-sdk`). **This is crucial!**
        *   Verify the correct Android Platform version is listed (e.g., `Platform android-35`, matching the default feature config).
        *   Ensure `Android SDK Command-line Tools`, `SDK Platform-Tools`, and `SDK Build-Tools` versions are listed.
        *   Check that Android licenses are accepted.
    *   Check `Linux toolchain` section (should show Clang, CMake, etc.).
    *   Check `Connected device` section (will likely show none initially).
5.  **Check Manually Installed Tools:**
    *   `scrcpy --version`
6.  **Test Aliases:** Type `fd` and press Enter (should run `flutter doctor`). Try `frc`, `supas`, etc.

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
