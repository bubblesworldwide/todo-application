# Mobile Setup Guide

This document explains how to install the todo app on a phone and how to package it as a native Android app. There are three paths, ordered from easiest to most involved.

---

## Path 1 — Install as a PWA (no packaging needed)

This is the simplest path and gives users a real app icon on their home screen. No app stores involved.

### Android (Chrome)
1. Open `https://todotets.netlify.app` in Chrome.
2. Tap the three-dot menu in the top right.
3. Tap **Install app** (or **Add to Home screen**).
4. Confirm. The app icon appears on the home screen and launches fullscreen.

### iOS (Safari)
iOS does not support PWA install prompts the same way. Users have to:
1. Open the site in Safari (not Chrome — Chrome on iOS can't install PWAs).
2. Tap the Share button (square with up arrow).
3. Scroll down and tap **Add to Home Screen**.
4. Confirm.

> iOS limitations: no background sync, limited storage (~50 MB), and the service worker is more restricted than on Android. Most of your todo functionality will work, but expect quirks.

---

## Path 2 — WebNative VSCode extension (recommended for quick native builds)

The WebNative extension wraps your web app into a native shell from inside VSCode, without needing to install the full Android SDK / JDK / Gradle toolchain manually.

> Heads up: the exact menu names in WebNative change between versions. If anything below looks different in your install, check the extension's marketplace page for the current docs.

### One-time setup
1. Open VSCode.
2. Open the Extensions panel (`Ctrl+Shift+X`).
3. Search for **WebNative** and install it.
4. Reload VSCode if prompted.

### Building the app
1. Open your project folder in VSCode (the folder with `index.html`).
2. Open the Command Palette (`Ctrl+Shift+P`).
3. Type `WebNative` to see the available commands (typically something like *Initialize*, *Build*, *Run*).
4. Run the **Initialize** (or equivalent) command and fill in:
   - **App name**: Todo
   - **Package ID / Bundle ID**: `com.bubblesworldwide.todo` (must be unique; reverse domain style)
   - **Start URL**: `https://todotets.netlify.app`
   - **Icon**: point to your 512×512 PNG icon
5. Run the **Build** command. Output usually lands in a `dist/`, `native/`, or `build/` folder.

### Result
You'll get an `.apk` (Android) and/or `.ipa` (iOS) file. Sideload the `.apk` onto a phone to test, or submit it to the Play Store.

---

## Path 3 — Bubblewrap (the full toolchain, what you tried before)

Use this if WebNative doesn't give you enough control over the Android build, or if you specifically need a TWA (Trusted Web Activity) for Play Store submission.

### Prerequisites
- **Node.js 18+** and npm
- **Java JDK 17** (Bubblewrap can download this automatically to `~/.bubblewrap/jdk/`)
- **Android SDK** (Bubblewrap also handles this, into `~/.bubblewrap/android_sdk/`)
- Git Bash on Windows, or any terminal on Mac/Linux

### Setup
```bash
npm install -g @bubblewrap/cli
```

### Initialize from your PWA's manifest
```bash
bubblewrap init --manifest=https://todotets.netlify.app/manifest.json
```

Answer the prompts (most defaults are fine). It will generate a keystore — **save the password somewhere safe**, you'll need it for every future update.

### Build
```bash
bubblewrap build
```

This produces `app-release-signed.apk` and `app-release-bundle.aab`. The `.aab` is what you upload to the Play Store; the `.apk` is for direct install.

### Install on a phone for testing
With the phone in USB debug mode and connected:
```bash
bubblewrap install
```

Or copy the `.apk` to the phone manually and tap it.

### Digital Asset Links (important for TWA)
For the app to open without showing the browser bar at the top, you must verify ownership of the website:

1. Bubblewrap generates an `assetlinks.json` file during build.
2. Host it at `https://todotets.netlify.app/.well-known/assetlinks.json`.
3. Verify with: `https://developers.google.com/digital-asset-links/tools/generator`

---

## Updating the app later

The huge advantage of all three paths is that your **app content lives at the URL** (`todotets.netlify.app`). To ship a new version of the todo logic:

1. Edit `index.html` / `sw.js` / `manifest.json` locally.
2. `git push` to GitHub.
3. Netlify auto-deploys.
4. Users get the update the next time they open the app.

You only need to rebuild and republish the native shell (Paths 2 or 3) if you change:
- The app name, icon, or splash screen
- Android permissions
- The package/bundle ID
- The start URL

---

## Common gotchas

- **Service worker not updating**: bump the cache version string in `sw.js` whenever you change cached files.
- **Icon looks fuzzy**: provide a 512×512 PNG with a transparent or solid background, and a separate "maskable" icon for Android adaptive icons.
- **App opens in browser, not fullscreen**: your manifest's `display` field needs to be `"standalone"` or `"fullscreen"`, and (for TWA) Digital Asset Links must be set up correctly.
- **HTTPS required**: PWAs only work over HTTPS. Netlify handles this automatically.
- **Keystore lost**: if you lose the `.keystore` file or its password, you **cannot update** your Play Store app under the same listing. Back it up to a password manager.
