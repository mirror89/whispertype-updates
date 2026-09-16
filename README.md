# WhisperType Updates

Öffentliches Feed-Repo: [github.com/mirror89/whispertype-updates](https://github.com/mirror89/whispertype-updates)

Zwei getrennte Dinge — nicht verwechseln:

| Was | Für wen | Wo |
| --- | --- | --- |
| **DMG** in `macos/` | Mensch lädt Installer herunter (Website, Lemon Squeezy, Link) | z. B. `macos/WhisperType_1.0.1_aarch64.dmg` |
| **`macos/latest.json`** | Schon installierte App prüft auf Update | Die App liest nur diese Datei |

Eine neuere DMG allein ändert **nichts** an bestehenden Installationen. Die App vergleicht ihre eigene Version (z. B. `1.0.0` in der gebauten App) mit dem Feld `"version"` in `latest.json`. Nur wenn die JSON-Version **höher** ist, erscheint **Nach Updates suchen** / der Dialog nach dem Start.

Die Mac-App ruft auf:

`https://raw.githubusercontent.com/mirror89/whispertype-updates/main/macos/latest.json`

Windows und Linux später: `windows/latest.json` bzw. `linux/latest.json`.

---

## Release 1.0.1 (Beispiel)

### 1. Version in der App hochsetzen

Im WhisperType-Projekt **dieselbe** Nummer an drei Stellen:

- `package.json` → `"version"`
- `src-tauri/Cargo.toml` → `version`
- `src-tauri/tauri.conf.json` → `"version"`

Ohne diesen Schritt denkt eine neu gebaute App noch, sie sei `1.0.0`, und würde sich nicht als Update anbieten.

### 2. DMG bauen

Im WhisperType-Ordner:

```bash
export TAURI_SIGNING_PRIVATE_KEY_PATH="/Pfad/zu/Whispertype/src-tauri/updater.key"
export TAURI_SIGNING_PRIVATE_KEY_PASSWORD=""
npm run tauri -- build --bundles dmg
```

Die Datei `updater.key` ist **privat** (gitignored). Ohne sie kann das In-App-Update nicht signiert werden. Passwort ist leer, solange der Key so erzeugt wurde.

Ergebnisse typischerweise:

- Installer: `src-tauri/target/release/bundle/dmg/WhisperType_1.0.1_aarch64.dmg`
- In-App-Archiv: `src-tauri/target/release/bundle/macos/WhisperType.app.tar.gz`
- Signatur: dieselbe Datei plus `.sig` (Inhalt der `.sig` kommt in `latest.json`)

Falls ein altes DMG-Volume hängt: `hdiutil detach /Volumes/WhisperType -force`

### 3. Dateien ins Update-Repo

In **dieses** Repo (`whispertype-updates`):

1. DMG nach `macos/` kopieren (Download für Menschen).
2. `WhisperType.app.tar.gz` als GitHub-Release `v1.0.1` hochladen **oder** ebenfalls versioniert ablegen und die URL in der JSON darauf zeigen.
3. `macos/latest.json` anpassen:

```json
{
  "version": "1.0.1",
  "notes": "Kurz was neu ist.",
  "pub_date": "2026-09-16T12:00:00Z",
  "platforms": {
    "darwin-aarch64": {
      "signature": "<kompletter Inhalt der .sig-Datei>",
      "url": "https://github.com/mirror89/whispertype-updates/releases/download/v1.0.1/WhisperType.app.tar.gz"
    },
    "darwin-x86_64": {
      "signature": "<gleiche Signatur, falls ein Universal-/Intel-Build>",
      "url": "https://github.com/mirror89/whispertype-updates/releases/download/v1.0.1/WhisperType.app.tar.gz"
    }
  }
}
```

`"version"` muss **höher** sein als die Version, die Nutzer gerade installiert haben. Die `"url"` muss die **.app.tar.gz** sein, nicht die DMG — die laufende App kann eine DMG nicht als In-App-Update einspielen.

4. Committen und `git push`.

### 4. Optional: Lemon Squeezy

Neue DMG dort als Produktdatei tauschen, damit **neue** Käufe die aktuelle Version bekommen. Das steuert nicht das In-App-Update.

---

## Prüfen

- App `1.0.0` installiert lassen.
- `latest.json` auf `1.0.1` + gültige `url` + `signature` pushen.
- WhisperType → **Nach Updates suchen…** → Dialog → Installieren → Neustart.
- Danach sollte **Über WhisperType** `1.0.1` zeigen.

Wenn der Dialog ausbleibt: JSON noch `1.0.0`, oder die App wurde schon mit derselben Nummer gebaut.

---

## Ordner

```text
macos/    Mac (Apple Silicon + später Intel)
windows/  später
linux/    später
```
