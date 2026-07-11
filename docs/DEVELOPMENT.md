# Development Guidelines

## Overview
Cross-platform clipboard manager for macOS and iOS with Supabase backend and Realm local storage.

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| UI | SwiftUI | Cross-platform native UI |
| Local Storage | Realm | Offline clipboard history |
| Backend | Supabase | Auth, sync, real-time |
| Auth | Supabase Auth (Google) | User authentication |
| Sync | Supabase Realtime | Cross-device sync |

## Project Structure

```
ClipboardManager/
├── App/
│   └── ClipboardManagerApp.swift
├── Features/
│   ├── Auth/
│   │   ├── LoginView.swift
│   │   └── AuthManager.swift
│   ├── Clipboard/
│   │   ├── ClipboardView.swift
│   │   ├── ClipboardManager.swift
│   │   └── ClipboardItem.swift
│   ├── Categories/
│   │   ├── CategoryView.swift
│   │   └── CategoryManager.swift
│   └── Settings/
│       ├── SettingsView.swift
│       └── SyncSettings.swift
├── Services/
│   ├── SupabaseService.swift
│   ├── RealmService.swift
│   ├── ClipboardSyncService.swift
│   └── NotificationService.swift
├── Models/
│   ├── User.swift
│   ├── ClipboardItem.swift
│   └── Category.swift
├── Utils/
│   ├── Extensions.swift
│   ├── Constants.swift
│   └── Helpers.swift
└── Resources/
    ├── Assets.xcassets
    └── Localizable.xcstrings
```

## Development Setup

### Prerequisites
- Xcode 15.0+
- macOS 13.0+ / iOS 17.0+
- Supabase account (supabase.com)
- Realm Swift installed via SPM

### Environment Variables
Create `Config.xcconfig` (not in git):
```
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
GOOGLE_CLIENT_ID=your-google-client-id
```

### Running Locally
1. Clone repository
2. Open `ClipboardManager.xcodeproj`
3. Select target (macOS or iOS)
4. Build and run (⌘+R)

## Code Conventions

### Swift Style
- Use `swift-format` for formatting
- Follow Swift API Design Guidelines
- Use `@Observable` for state management (iOS 17+)

### File Naming
- Views: `FeatureNameView.swift`
- Managers: `FeatureNameManager.swift`
- Models: `ModelName.swift`
- Services: `ServiceName.swift`

### Error Handling
```swift
enum ClipboardError: LocalizedError {
    case syncFailed(String)
    case authRequired
    case networkUnavailable
    
    var errorDescription: String? {
        switch self {
        case .syncFailed(let reason): return "Sync failed: \(reason)"
        case .authRequired: return "Please sign in"
        case .networkUnavailable: return "No internet connection"
        }
    }
}
```

## Key Dependencies

### SPM Packages
```swift
dependencies: [
    .package(url: "https://github.com/supabase/supabase-swift.git", from: "2.0.0"),
    .package(url: "https://github.com/realm/realm-swift.git", from: "10.0.0"),
    .package(url: "https://github.com/google/GoogleSignIn.git", from: "7.0.0")
]
```

## Testing

### Unit Tests
- Test managers and services in isolation
- Mock Supabase/Realm for unit tests
- Aim for 80%+ coverage on business logic

### UI Tests
- Test login flow
- Test clipboard copy/paste
- Test category management
- Test sync between devices

## Branch Strategy

```
main ──────────────────────────────────► Production
  └── develop ────────────────────────► Staging
       ├── feature/clipboard-sync ────► Feature
       ├── feature/google-auth ───────► Feature
       └── bugfix/sync-issue ────────► Bugfix
```
