
name: Build NASRALLAH TV APK

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Get source code
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Extract source ZIP
        run: |
          mkdir -p extracted
          unzip -o NASRALLAH_TV_VLC_source.zip -d extracted

      - name: Find Android project
        id: project
        run: |
          PROJECT_DIR=$(find extracted -type f \( -name settings.gradle -o -name settings.gradle.kts \) -print -quit | xargs -r dirname)
          if [ -z "$PROJECT_DIR" ]; then
            echo "Android Gradle project not found in ZIP."
            exit 1
          fi
          echo "dir=$PROJECT_DIR" >> "$GITHUB_OUTPUT"

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.9'

      - name: Build debug APK
        working-directory: ${{ steps.project.outputs.dir }}
        run: |
          if [ -f gradlew ]; then
            chmod +x gradlew
            ./gradlew --no-daemon assembleDebug
          else
            gradle --no-daemon assembleDebug
          fi

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: NASRALLAH-TV-APK
          path: extracted/**/build/outputs/apk/debug/*.apk
          if-no-files-found: error
