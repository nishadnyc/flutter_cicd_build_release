# Flutter Automated CI/CD Release Pipeline

A production-ready **GitHub Actions workflow** for Flutter applications that automatically compiles release APKs and publishes tagged GitHub Releases on every push to the `main` branch.

---

## 🚀 Key Features

- **Automated Triggers:** Listens to `push` events on the `main` branch.
- **Java Setup:** Configures Java 17 (Azul Zulu distribution) required by Android Gradle build tools.
- **Flutter Setup:** Utilizes `subosito/flutter-action` to fetch the latest `stable` Flutter SDK.
- **Dependency Resolution:** Installs all Flutter package dependencies defined in `pubspec.yaml`.
- **Release Build:** Compiles a standalone, optimized release APK (`app-release.apk`).
- **Automated Publishing:** Creates a GitHub Release (tagged sequentially by build run number `v<run_number>`) and attaches the generated APK artifact automatically.

---

## 🛠️ Workflow Architecture

```yaml
name: Flutter Build APK

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # 1. Check out repository code
      - uses: actions/checkout@v5

      # 2. Set up Java (Required by Android Gradle compiler tools)
      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: "zulu"
          java-version: "17"

      # 3. Set up Flutter SDK environment
      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: "stable"

      # 4. Resolve package dependencies
      - name: Install dependencies
        run: flutter pub get

      # 5. Compile standalone release APK
      - name: Build production APK
        run: flutter build apk --release

      # 6. Generate GitHub Release and attach release APK
      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: v${{ github.run_number }}
          name: Release Build v${{ github.run_number }}
          draft: false
          prerelease: false
          files: build/app/outputs/flutter-apk/app-release.apk
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 📋 Prerequisites & Setup

### 1. Workflow Placement
Place the workflow YAML file inside your Flutter project at:
```text
.github/workflows/main.yml
```

### 2. GitHub Token Permissions
The workflow uses the automatically generated `GITHUB_TOKEN` to publish releases. Ensure repository workflow permissions are set correctly:

1. Navigate to **Settings** > **Actions** > **General** in your GitHub repository.
2. Under **Workflow permissions**, select **Read and write permissions**.
3. Click **Save**.

---

## 📦 How It Works

1. **Trigger:** Push code to `main`.
2. **Build Execution:** GitHub runner provisions `ubuntu-latest`, checks out code, loads Java 17 and Flutter, installs dependencies via `flutter pub get`, and runs `flutter build apk --release`.
3. **Release Artifact:** The generated APK (`build/app/outputs/flutter-apk/app-release.apk`) is published under **Releases** tagged as `v<run_number>` (e.g., `v1`, `v2`).

---

## 📄 License
This workflow design is available under the [MIT License](LICENSE).
