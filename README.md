# Sukoon — public update feed

Public distribution channel for **Sukoon** (formerly *Adhan Widget*), the native
macOS prayer-times menu bar app. Source lives in the private repos
`MindMints/Sukoon` (Xcode project) and `MindMints/Sukoon-source` (mirror).

## Feeds

| File | Consumed by | Purpose |
|------|-------------|---------|
| `sukoon/appcast.xml` | Sukoon 1.3+ (bundle id `ai.mindmints.Sukoon`) via Sparkle 2 | Silent auto-updates, EdDSA-signed |
| `appcast.xml` | legacy *Adhan Widget* 1.2 (bundle id `ai.mindmints.AdhanWidgetHost`) | **Frozen at 1.2 forever** — Sparkle refuses cross-bundle-id updates, so this population migrates manually |
| `latest.json` | legacy *Adhan Widget* 1.0/1.1 (JSON checker) | Migration beacon pointing at the latest GitHub Release |

## Release checklist (per release)

1. In the Sukoon Xcode project: bump `MARKETING_VERSION` + `CURRENT_PROJECT_VERSION` (Sparkle compares the **build** number).
2. `xcodebuild -scheme Sukoon -configuration Release archive -archivePath /tmp/Sukoon.xcarchive`
3. `ditto -c -k --sequesterRsrc --keepParent <archive>/Products/Applications/Sukoon.app Sukoon-<ver>.zip`
4. `sign_update Sukoon-<ver>.zip` (Sparkle 2.9.x tools; the EdDSA private key is in the login keychain).
5. Append an `<item>` to `sukoon/appcast.xml`: `sparkle:version` = build, `sparkle:shortVersionString` = marketing version, `sparkle:minimumSystemVersion` = the archive's `LSMinimumSystemVersion`, the `enclosure` url/length/`sparkle:edSignature` from step 4. Omit `sparkle:hardwareRequirements` while builds are universal (`lipo -archs` to confirm).
6. Bump `latest.json` (build + notes + url) for legacy JSON-checker installs.
7. Commit here, create the GitHub Release `v<ver>` with the zip attached — **owner publishes; never publish without explicit approval.**
8. Smoke-test: serve this repo with `python3 -m http.server`, `defaults write ai.mindmints.Sukoon SUFeedURL http://localhost:8000/sukoon/appcast.xml`, run an older build, Check for Updates, then `defaults delete ai.mindmints.Sukoon SUFeedURL`.

Signing note: releases are currently Apple Development-signed (no Developer ID
cert on the build machine). Sparkle updates still verify via EdDSA, but enroll
team `45QCZ72475` for a Developer ID Application certificate + notarization
before wider distribution.
