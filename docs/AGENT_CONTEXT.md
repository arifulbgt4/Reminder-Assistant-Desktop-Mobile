# Agent Context

## Project Overview

**Name**: Clipboard Manager (future: ReminderAssistant)
**Purpose**: Cross-platform clipboard sync manager for macOS and iOS
**Goal**: Default clipboard for Mac and iOS with Google login, real-time sync, and pinned text by category

## Technical Stack

| Component | Technology | Notes |
|-----------|-----------|-------|
| Platform | macOS 13+ / iOS 17+ | Universal app |
| Language | Swift 5.0 | |
| UI Framework | SwiftUI | |
| Local Database | Realm Swift | Offline storage |
| Backend | Supabase | Auth + Realtime sync |
| Auth Provider | Google OAuth | Via Supabase Auth |
| Build System | Xcode 15+ | |

## Project Structure

```
ClipboardManager/
├── App/                    # App entry point
│   └── ClipboardManagerApp.swift
├── Features/               # Feature modules
│   ├── Auth/              # Google login
│   │   ├── LoginView.swift
│   │   └── AuthManager.swift
│   ├── Clipboard/         # Clipboard management
│   │   ├── ClipboardView.swift
│   │   ├── ClipboardManager.swift
│   │   └── ClipboardItem.swift
│   ├── Categories/        # Category system
│   │   ├── CategoryView.swift
│   │   └── CategoryManager.swift
│   └── Settings/          # App settings
│       ├── SettingsView.swift
│       └── SyncSettings.swift
├── Services/              # Business logic
│   ├── SupabaseService.swift
│   ├── RealmService.swift
│   └── ClipboardSyncService.swift
├── Models/                # Data models
│   ├── User.swift
│   ├── ClipboardItem.swift
│   └── Category.swift
└── Utils/                 # Helpers
    ├── Extensions.swift
    └── Constants.swift
```

## Current State

### Implemented
- [x] Basic SwiftUI app structure
- [x] macOS and iOS targets
- [x] "Welcome User" placeholder screen

### In Progress
- [ ] Google authentication
- [ ] Clipboard monitoring
- [ ] Realm database setup
- [ ] Supabase integration

### Not Started
- [ ] Real-time sync
- [ ] Category management
- [ ] Pinned items
- [ ] Settings UI

## Code Patterns

### SwiftUI View Pattern
```swift
struct FeatureView: View {
    @StateObject private var viewModel = FeatureViewModel()
    
    var body: some View {
        VStack {
            // Content
        }
        .task {
            await viewModel.loadData()
        }
    }
}
```

### ViewModel Pattern
```swift
@Observable
class FeatureViewModel {
    var items: [Item] = []
    var isLoading = false
    
    func loadData() async {
        isLoading = true
        defer { isLoading = false }
        // Load data
    }
}
```

### Service Pattern
```swift
class SupabaseService {
    static let shared = SupabaseService()
    
    func fetchItems() async throws -> [ClipboardItem] {
        // Implementation
    }
}
```

## Design Decisions

1. **Shared Code**: Single SwiftUI codebase for both platforms
2. **Offline First**: Realm for local storage, sync when available
3. **Real-time**: Supabase Realtime for instant sync
4. **Google Auth**: Via Supabase for simplicity
5. **Categories**: User-defined with pinned items

## Commands

### Build
```bash
# macOS
xcodebuild -scheme ClipboardManager build

# iOS
xcodebuild -scheme ClipboardManager-iOS build
```

### Test
```bash
xcodebuild test -scheme ClipboardManager-iOS -destination 'platform=iOS Simulator,name=iPhone 15'
```

### Lint
```bash
swiftlint lint --strict
```

## Environment Variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `SUPABASE_URL` | Supabase project URL | Yes |
| `SUPABASE_ANON_KEY` | Supabase anonymous key | Yes |
| `GOOGLE_CLIENT_ID` | Google OAuth client ID | Yes |

## API Reference

### Supabase Tables
- `users` - User profiles
- `clipboard_items` - Clipboard entries
- `categories` - User categories

### Supabase Realtime
- `clipboard_changes` - Real-time clipboard updates
- `category_changes` - Real-time category updates

## Common Tasks

### Adding a New Feature
1. Create feature folder in `Features/`
2. Add View, ViewModel, Manager
3. Add to navigation
4. Add tests
5. Update documentation

### Adding a New Model
1. Create model in `Models/`
2. Add Realm schema
3. Add Supabase mapping
4. Add to service layer

### Fixing a Bug
1. Locate relevant code
2. Write failing test
3. Fix the bug
4. Verify test passes
5. Update changelog

## Notes for AI Assistants

- **Platform**: Always check if feature works on both macOS and iOS
- **Offline**: All features should work offline with Realm
- **Sync**: Changes should sync when online
- **Accessibility**: Add VoiceOver labels
- **Performance**: Avoid blocking main thread
