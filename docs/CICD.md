# CI/CD Pipeline

## Overview

Automated build, test, and deployment pipeline using GitHub Actions.

## Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        CI/CD Pipeline                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌────────┐ │
│  │  Push/PR │───►│  Build   │───►│   Test   │───►│ Deploy │ │
│  └──────────┘    └──────────┘    └──────────┘    └────────┘ │
│                       │               │               │      │
│                       ▼               ▼               ▼      │
│                  ┌──────────┐    ┌──────────┐    ┌────────┐  │
│                  │  Lint    │    │  Speed   │    │Archive │  │
│                  └──────────┘    └──────────┘    └────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## GitHub Actions Workflow

### .github/workflows/ci.yml
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  DEVELOPER_DIR: /Applications/Xcode.app/Contents/Developer

jobs:
  lint:
    name: Lint
    runs-on: macos-14
    steps:
      - uses: actions/checkout@v4
      - name: SwiftLint
        run: |
          brew install swiftlint
          swiftlint lint --strict

  build-macos:
    name: Build macOS
    runs-on: macos-14
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - name: Build macOS
        run: |
          xcodebuild -project ClipboardManager.xcodeproj \
            -scheme ClipboardManager \
            -destination 'platform=macOS' \
            build

  build-ios:
    name: Build iOS
    runs-on: macos-14
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - name: Build iOS
        run: |
          xcodebuild -project ClipboardManager.xcodeproj \
            -scheme ClipboardManager-iOS \
            -destination 'platform=iOS Simulator,name=iPhone 15' \
            build

  test:
    name: Tests
    runs-on: macos-14
    needs: [build-macos, build-ios]
    steps:
      - uses: actions/checkout@v4
      - name: Run Tests
        run: |
          xcodebuild test \
            -project ClipboardManager.xcodeproj \
            -scheme ClipboardManager-iOS \
            -destination 'platform=iOS Simulator,name=iPhone 15' \
            -resultBundlePath TestResults.xcresult
      - name: Upload Test Results
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: TestResults.xcresult

  archive-ios:
    name: Archive iOS
    runs-on: macos-14
    needs: test
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Archive iOS
        env:
          CERTIFICATE: ${{ secrets.IOS_CERTIFICATE }}
          CERTIFICATE_PASSWORD: ${{ secrets.IOS_CERTIFICATE_PASSWORD }}
          KEYCHAIN_PASSWORD: ${{ secrets.KEYCHAIN_PASSWORD }}
        run: |
          # Create temporary keychain
          security create-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
          security default-keychain -s build.keychain
          security unlock-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
          
          # Import certificate
          echo "$CERTIFICATE" | base64 -d > certificate.p12
          security import certificate.p12 \
            -k build.keychain \
            -P "$CERTIFICATE_PASSWORD" \
            -T /usr/bin/codesign
          
          # Archive
          xcodebuild -project ClipboardManager.xcodeproj \
            -scheme ClipboardManager-iOS \
            -sdk iphoneos \
            -archivePath build/ClipboardManager-iOS.xcarchive \
            archive

  deploy-testflight:
    name: Deploy to TestFlight
    runs-on: macos-14
    needs: archive-ios
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Upload to TestFlight
        env:
          API_KEY_ID: ${{ secrets.APP_STORE_API_KEY_ID }}
          API_ISSUER_ID: ${{ secrets.APP_STORE_API_ISSUER_ID }}
        run: |
          xcrun altool --upload-app \
            --type ios \
            --file build/ClipboardManager-iOS.xcarchive \
            --apiKey "$API_KEY_ID" \
            --apiIssuer "$API_ISSUER_ID"
```

## Required Secrets

### GitHub Secrets
| Secret | Purpose |
|--------|---------|
| `IOS_CERTIFICATE` | Base64 encoded .p12 certificate |
| `IOS_CERTIFICATE_PASSWORD` | Certificate password |
| `KEYCHAIN_PASSWORD` | Temporary keychain password |
| `APP_STORE_API_KEY_ID` | App Store Connect API key |
| `APP_STORE_API_ISSUER_ID` | App Store Connect issuer ID |

### Setup Commands
```bash
# Export certificate as base64
base64 -i cert.p12 | pbcopy

# Set secrets in GitHub repo settings
gh secret set IOS_CERTIFICATE < cert_base64.txt
gh secret set IOS_CERTIFICATE_PASSWORD
gh secret set KEYCHAIN_PASSWORD
gh secret set APP_STORE_API_KEY_ID
gh secret set APP_STORE_API_ISSUER_ID
```

## Local Development

### Pre-commit Hooks
```bash
#!/bin/bash
# .git/hooks/pre-commit

# Run SwiftLint
swiftlint lint --strict --quiet

# Run tests
xcodebuild test \
  -project ClipboardManager.xcodeproj \
  -scheme ClipboardManager-iOS \
  -destination 'platform=iOS Simulator,name=iPhone 15' \
  -quiet

# Check for TODO/FIXME
if git diff --cached --name-only | xargs grep -l "TODO\|FIXME"; then
  echo "⚠️  Found TODO/FIXME comments"
  exit 1
fi
```

### Fastlane Integration
```ruby
# Fastfile
default_platform(:ios)

platform :ios do
  desc "Run tests"
  lane :test do
    scan(
      project: "ClipboardManager.xcodeproj",
      scheme: "ClipboardManager-iOS",
      destination: "platform=iOS Simulator,name=iPhone 15"
    )
  end

  desc "Build for App Store"
  lane :beta do
    build_app(
      project: "ClipboardManager.xcodeproj",
      scheme: "ClipboardManager-iOS",
      export_method: "app-store"
    )
    upload_to_testflight
  end

  desc "Release to App Store"
  lane :release do
    build_app(
      project: "ClipboardManager.xcodeproj",
      scheme: "ClipboardManager-iOS",
      export_method: "app-store"
    )
    upload_to_app_store(
      force: true,
      skip_screenshots: true,
      skip_metadata: false
    )
  end
end

platform :mac do
  desc "Build and notarize macOS app"
  lane :release do
    build_app(
      project: "ClipboardManager.xcodeproj",
      scheme: "ClipboardManager",
      export_method: "developer-id"
    )
    notarize(
      app_path: "build/ClipboardManager.app",
      apple_id: ENV["APPLE_ID"],
      team_id: ENV["TEAM_ID"]
    )
  end
end
```

## Monitoring

### Post-Deployment Checks
```bash
# Check App Store status
xcrun altool --list-apps \
  --apiKey "$API_KEY_ID" \
  --apiIssuer "$API_ISSUER_ID"

# Check TestFlight builds
curl -X GET "https://appstoreconnect.apple.com/alpha/v1/builds" \
  -H "Authorization: Bearer $TOKEN"
```

### Alerts
- Build failure → Slack notification
- Test failure → Email team
- Deploy success → Discord notification
