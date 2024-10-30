# File-Hub
## Multiple apps. One Place.

**This is a project aimed at creating connections** on the Telegram topic, where each application is sent via workflow. Join and access on Telegram @fi_lehub. Discover amazing applications!

> [!warning]
> Files smaller than 50 MB

```js
name: Android CI Debug

on:
  push:
    branches: ["**"]
    paths-ignore:
      - '**/*.md'
  pull_request:
    branches: ["**"]
    paths-ignore:
      - '**/*.md'
  workflow_dispatch:

jobs:
  init:
    name: Init
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

  build_debug_apk:
    name: Build App Debug APK
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up JDK 20
        uses: actions/setup-java@v4
        with:
          java-version: '20' //default
          distribution: 'oracle' /default
          cache: gradle

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Validate Gradle wrapper
        uses: gradle/actions/wrapper-validation@v4

      - name: Build with Gradle
        id: gradle_build_debug
        run: ./gradlew assembleDebug

      - name: Upload debug APK
        uses: actions/upload-artifact@v4
        with:
          name: app-debug
          path: app/build/outputs/apk/debug/app-debug.apk

  send_debug_apk:
    name: Send Debug APK
    runs-on: ubuntu-latest
    needs: build_debug_apk

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Download debug APK
        uses: actions/download-artifact@v4
        with:
          name: app-debug

      - name: Rename APK file
        run: |
          mv app-debug.apk custom-app-debug.apk /custom
          ls 

      - name: Send APK to Telegram
        if: success()
        continue-on-error: true
        run: |
          curl -X POST "https://api.telegram.org/bot${{ secrets.TELEGRAM_BOT_TOKEN }}/sendDocument" \
            -F chat_id="-1002327390277" \
            -F message_thread_id="30" \ // custom
            -F document=@"custom-app-debug.apk" \
            -F caption="App by @robokinc - ${{ github.event.head_commit.message }} by ${{ github.actor }}" // custom
```

#

> [!caution]
> Files larger than 50 MB

```js
name: Android CI Debug

on:
  push:
    branches: ["**"]
    paths-ignore:
      - '**/*.md'
  pull_request:
    branches: ["**"]
    paths-ignore:
      - '**/*.md'
  workflow_dispatch:

jobs:
  # Etapa 1: Build do APK
  build_debug_apk:
    name: Build App Debug APK
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up JDK 20
        uses: actions/setup-java@v4
        with:
          java-version: '20'
          distribution: 'oracle'
          cache: gradle

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Validate Gradle wrapper
        uses: gradle/actions/wrapper-validation@v4

      - name: Build with Gradle
        id: gradle_build_debug
        run: ./gradlew assembleDebug

      - name: Upload debug APK
        uses: actions/upload-artifact@v4
        with:
          name: app-debug
          path: app/build/outputs/apk/debug/app-debug.apk

  # Etapa 2: Envio do APK para o Telegram em partes
  send_debug_apk:
    name: Send Debug APK
    runs-on: ubuntu-latest
    needs: build_debug_apk

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Download debug APK
        uses: actions/download-artifact@v4
        with:
          name: app-debug

      - name: Rename APK file
        run: |
          mv app-debug.apk custom-app-debug.apk
          
      - name: Split APK into smaller parts
        run: |
          split -b 45M custom-app-debug.apk "custom-app-debug-part-"
          
      - name: Send APK parts to Telegram
        if: success()
        continue-on-error: true
        run: |
          for part in custom-app-debug-part-*; do
            curl -X POST "https://api.telegram.org/bot${{ secrets.TELEGRAM_BOT_TOKEN }}/sendDocument" \
              -F chat_id="-1002327390277" \
              -F message_thread_id="30" \
              -F document=@"$part"
          done

      - name: Send final recommendation message
        if: success()
        run: |
          curl -X POST "https://api.telegram.org/bot${{ secrets.TELEGRAM_BOT_TOKEN }}/sendMessage" \
            -F chat_id="-1002327390277" \
            -F message_thread_id="30" \
            -F text="All APK parts have been sent by @robokinc. To reassemble, use the command: ```bash\ncat custom-app-debug-part-* > custom-app-debug.apk```."
```
