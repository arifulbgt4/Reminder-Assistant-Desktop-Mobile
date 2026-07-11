# Deployment Guide

## Overview

Deployment targets:
- **macOS**: Direct distribution + Mac App Store
- **iOS**: App Store + TestFlight

## Prerequisites

### Apple Developer Account
- Valid Apple Developer Program membership
- App ID registered for both macOS and iOS
- Provisioning profiles configured

### Supabase Production
- Production Supabase project created
- Environment variables configured
- RLS policies tested

## macOS Deployment

### Direct Distribution (Outside App Store)
```bash
# 1. Archive the app
xcodebuild -project ClipboardManager.xcodeproj \
  -scheme ClipboardManager \
  -archivePath build/ClipboardManager.xcarchive \
  archive

# 2. Export the archive
xcodebuild -exportArchive \
  -archivePath build/ClipboardManager.xcarchive \
  -exportOptionsPlist ExportOptions.plist \
  -exportPath build/export
```

### ExportOptions.plist
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>developer-id</string>
    <key>teamID</key>
    <string>YOUR_TEAM_ID</string>
</dict>
</plist>
```

### Code Signing
- Use Developer ID certificate
- Enable Hardened Runtime
- Notarize with Apple:
```bash
xcrun notarytool submit build/export/ClipboardManager.zip \
  --apple-id your@email.com \
  --team-id TEAM_ID \
  --password app-specific-password \
  --wait
```

## iOS Deployment

### TestFlight (Beta Testing)
```bash
# 1. Archive for iOS
xcodebuild -project ClipboardManager.xcodeproj \
  -scheme ClipboardManager-iOS \
  -archivePath build/ClipboardManager-iOS.xcarchive \
  -sdk iphoneos \
  archive

# 2. Export IPA
xcodebuild -exportArchive \
  -archivePath build/ClipboardManager-iOS.xcarchive \
  -exportOptionsPlist ExportOptions-iOS.plist \
  -exportPath build/export-ios
```

### App Store Submission
```bash
# Upload to App Store Connect
xcrun altool --upload-app \
  --type ios \
  --file build/export-ios/ClipboardManager-iOS.ipa \
  --apiKey YOUR_API_KEY \
  --apiIssuer YOUR_ISSUER_ID
```

## Environment Configuration

### Debug
```swift
#if DEBUG
let supabaseURL = "https://dev-project.supabase.co"
let supabaseKey = "dev-anon-key"
#endif
```

### Release
```swift
#if RELEASE
let supabaseURL = "https://prod-project.supabase.co"
let supabaseKey = "prod-anon-key"
#endif
```

## Versioning

### Semantic Versioning
- **Major**: Breaking changes
- **Minor**: New features
- **Patch**: Bug fixes

### Build Numbers
- Auto-increment in Xcode
- Use `agvtool` for CI/CD

## Post-Deployment

### Verification
1. Test on clean device
2. Verify sync works
3. Check analytics
4. Monitor crash reports

### Rollback Plan
1. Keep previous version archived
2. Have App Store rollback ready
3. Monitor user feedback

## Certificates & Provisioning

### Required Certificates
| Certificate | Purpose |
|-------------|---------|
| Apple Distribution | App Store submission |
| Developer ID | macOS direct distribution |
| Apple Push Services | Push notifications |

### Provisioning Profiles
| Profile | Platform | Purpose |
|---------|----------|---------|
| App Store | iOS | Production |
| Development | iOS | Testing |
| Direct | macOS | Notarized distribution |
