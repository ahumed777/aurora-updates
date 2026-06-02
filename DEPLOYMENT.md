# Aurora Deployment Guide

Use this workflow for each public Aurora update distributed outside the Mac
App Store.

## One-Time Apple Setup

1. Join the Apple Developer Program.
2. In Xcode, open **Settings > Accounts**, select the team, open **Manage
   Certificates**, and create a **Developer ID Application** certificate.
3. Enable **Hardened Runtime** for the Aurora target's Release configuration.
4. Create an app-specific password for the Apple ID used for notarization.
5. Store the notarization credentials:

   ```bash
   xcrun notarytool store-credentials "aurora-notary" \
     --apple-id "YOUR_APPLE_ID" \
     --team-id "YOUR_TEAM_ID" \
     --password "YOUR_APP_SPECIFIC_PASSWORD"
   ```

6. Keep the Sparkle private signing key available in Keychain. Do not commit
   the private key to GitHub.

## Build, Sign, and Notarize

1. Update `MARKETING_VERSION` and increment `CURRENT_PROJECT_VERSION` in Xcode.
2. From the Aurora source repository root, run:

   ```bash
   TEAM_ID="YOUR_TEAM_ID" bash scripts/release_notarize.sh
   ```

3. The script creates a Release archive, exports the Developer ID-signed app,
   submits it to Apple with `notarytool`, staples the ticket, verifies the app,
   and creates:

   ```text
   dist/<timestamp>/Aurora-<version>.zip
   ```

4. Do not publish an Xcode Preview or Debug app. The public archive must not
   contain `__preview.dylib` or `Aurora.debug.dylib`.

## Publish the Sparkle Update

1. Clone or update this repository:

   ```bash
   git clone https://github.com/ahumed777/aurora-updates.git \
     "$HOME/Desktop/aurora-updates"
   cd "$HOME/Desktop/aurora-updates"
   git pull
   ```

2. Locate the Sparkle tools:

   ```bash
   SPARKLE_BIN=$(find "$HOME/Library/Developer/Xcode/DerivedData" \
     -path "*SourcePackages/artifacts/sparkle/Sparkle/bin" \
     -type d 2>/dev/null | head -n 1)
   echo "$SPARKLE_BIN"
   ```

3. Copy the notarized archive into the update repository:

   ```bash
   cp "/PATH/TO/Aurora-<version>.zip" .
   ```

4. Generate the signed appcast entry:

   ```bash
   "$SPARKLE_BIN/generate_appcast" .
   ```

5. Confirm that `appcast.xml` references the new archive, the correct version,
   and a Sparkle signature.
6. Update `RELEASE_NOTES.md` and `CHANGELOG.md`.
7. Commit and push:

   ```bash
   git add appcast.xml "Aurora-<version>.zip" README.md RELEASE_NOTES.md CHANGELOG.md
   git commit -m "Release <version>"
   git push origin main
   ```

## Verify

1. Wait for GitHub Pages to update.
2. Open:

   ```text
   https://ahumed777.github.io/aurora-updates/appcast.xml
   https://ahumed777.github.io/aurora-updates/Aurora-<version>.zip
   ```

3. Test **Settings > Check for Updates** from an older Aurora build.
