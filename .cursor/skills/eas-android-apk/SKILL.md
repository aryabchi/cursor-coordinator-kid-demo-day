---
name: eas-android-apk
description: >-
  Build an installable Android APK for an Expo app with EAS cloud build.
  Use when the user asks for an Android APK, a phone install, or an EAS
  Android build. Reads the Expo token from the EXPO_TOKEN environment secret.
environments:
  - cloud
---

# EAS Android APK

Build a direct-install `.apk` for the Expo app in the current checkout. Do not publish to Google Play.

Use the git branch that is already checked out, or the branch the user names. If that branch already has a pull request, push there. Do not open a second PR.

## Do not

- Install Android Studio, the Android SDK, a JDK, or a local Gradle setup.
- Run `eas submit` or upload to Google Play.
- Commit a keystore or any other secret.
- Print, log, or paste `EXPO_TOKEN`. Never ask the user to put a token in chat.
- Change UI or other app behavior. Touch only APK config (`eas.json`, the app config’s Android package and EAS project link, lockfile if a required CLI pin needs it).
- Leave a dev server running after the pre-build check.

## Config

Add or update an `eas.json` profile named **`apk`**:

- `distribution`: `internal`
- `android.buildType`: `apk` (not `app-bundle`)
- `credentialsSource`: `remote` (EAS-managed keystore)

Android package: keep the existing application id. If it is missing, set a stable reverse-DNS id from the Expo owner and slug (lowercase, `[a-z0-9_]` only), report the id you chose, and do not change it later.

## Before the build

1. `npx tsc --noEmit`
2. `npx expo export --platform web`

Both must pass. Stop any dev server you started for the check.

## Token

1. Check EXPO_TOKEN environment secret:
2. If it is **unset**, stop. Ask the user, in their language, to create an Expo account and add a personal access token. Do not ask them to paste the token. When they say the secret is in place, continue this skill from the token check. If they already added it and this machine still shows unset, start a **new** cloud VM (secrets are injected at boot) and continue there.
3. If it is **set**: `npx eas-cli whoami`. Use the username only.

## Build

Start the build:

```bash
npx eas-cli build --platform android --profile apk --non-interactive
```

The build takes a long time. As soon as EAS prints the build page URL, report that URL to the Project Coordinator in that turn. Do not wait until the build finishes. Say that the build is in progress and name the profile (`apk`). The Coordinator posts the link in the user chat immediately so the user can open the page and watch the build.

Then wait until EAS finishes.

- If the Expo project is not linked, link or create it non-interactively (`npx eas-cli init --non-interactive`). Let EAS generate the Android keystore. Commit only the `projectId` and `owner` that this command writes.
- If the CLI still needs a human credentials click, stop and give the exact Expo dashboard steps. Do not ask for a password or token in chat. Continue after the user says that step is done.

## Report in chat

Answer in the user’s language. Include:

1. Build profile (`apk`)
2. Android package identifier
3. EAS build status
4. Build page URL and the direct `.apk` URL
5. Short install steps: open the APK link in the phone’s browser, allow the download, open the file, allow install from that source if the system asks, tap Install, then open the app by the name in its Expo config.

Mention the artifact expiry date when EAS prints one.
