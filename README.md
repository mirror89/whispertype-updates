# WhisperType updates

Public feed for in-app updates. The Mac app reads `macos/latest.json`. Windows and Linux will use the other folders later.

## Layout

```text
macos/latest.json
windows/latest.json
linux/latest.json
```

Create a **public** GitHub repo named `whispertype-updates` (the app currently expects `mirror89/whispertype-updates`). Push this folder as `main`.

## Publish a new Mac version

1. Bump `version` in WhisperType (`tauri.conf.json`, `package.json`, `Cargo.toml`).
2. Build with the updater signing key:

```bash
export TAURI_SIGNING_PRIVATE_KEY_PATH="/absolute/path/to/src-tauri/updater.key"
export TAURI_SIGNING_PRIVATE_KEY_PASSWORD=""
npm run tauri -- build --bundles app
```

3. Copy `src-tauri/target/release/bundle/macos/WhisperType.app.tar.gz` and the matching `.sig` file.
4. GitHub Release `vX.Y.Z` with those artifacts.
5. Set `macos/latest.json` `version`, `url`, and `signature` (contents of the `.sig` file) and push.

The installed app only prompts when this `version` is **higher** than the running build.
