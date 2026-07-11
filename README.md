# Clipboard Manager

Cross-platform clipboard sync manager for macOS and iOS.

## Features

- **Google Sign-In** - Secure authentication
- **Real-time Sync** - Clipboard syncs instantly between devices
- **Categories** - Organize clipboard items by category
- **Pin Items** - Pin important items for quick access
- **Offline Support** - Works without internet, syncs when available
- **Dark Mode** - Beautiful UI in light and dark themes

## Requirements

- macOS 13.0+ / iOS 17.0+
- Xcode 15.0+
- Supabase account (for backend)

## Installation

### Clone Repository
```bash
git clone https://github.com/yourusername/clipboard-manager.git
cd clipboard-manager
```

### Open in Xcode
```bash
open ClipboardManager.xcodeproj
```

### Configure Environment
1. Create `Config.xcconfig` in the project root
2. Add your Supabase credentials:
```
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
GOOGLE_CLIENT_ID=your-google-client-id
```

### Build and Run
1. Select target (macOS or iOS)
2. Press ⌘+R to build and run

## Usage

### Sign In
1. Launch the app
2. Tap "Sign in with Google"
3. Complete Google authentication

### Copy Items
1. Copy text on any device
2. Items appear in clipboard history
3. Tap to copy again

### Organize
1. Create categories
2. Drag items to categories
3. Pin important items

### Sync
- Items sync automatically when online
- Works offline with local Realm storage
- Syncs when connection is restored

## Architecture

### Tech Stack
| Component | Technology |
|-----------|-----------|
| UI | SwiftUI |
| Local Storage | Realm |
| Backend | Supabase |
| Auth | Google OAuth |

### Project Structure
```
ClipboardManager/
├── App/           # App entry point
├── Features/      # Feature modules
├── Services/      # Business logic
├── Models/        # Data models
└── Utils/         # Helpers
```

## Development

### Setup
1. Install Xcode 15+
2. Clone repository
3. Configure environment variables
4. Build and run

### Commands
```bash
# Build macOS
xcodebuild -scheme ClipboardManager build

# Build iOS
xcodebuild -scheme ClipboardManager-iOS build

# Run Tests
xcodebuild test -scheme ClipboardManager-iOS -destination 'platform=iOS Simulator,name=iPhone 15'

# Lint
swiftlint lint --strict
```

### Branch Strategy
- `main` - Production releases
- `develop` - Development branch
- `feature/*` - Feature branches
- `bugfix/*` - Bug fix branches

## Documentation

- [Development Guide](docs/DEVELOPMENT.md)
- [Design System](docs/DESIGN.md)
- [Execution Plan](docs/EXECUTION.md)
- [Deployment Guide](docs/DEPLOYMENT.md)
- [CI/CD Pipeline](docs/CICD.md)
- [Maintenance Guide](docs/MAINTENANCE.md)
- [Agent Context](docs/AGENT_CONTEXT.md)

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests
5. Submit a pull request

## License

MIT License - see [LICENSE](LICENSE) for details

## Support

- **Issues**: [GitHub Issues](https://github.com/yourusername/clipboard-manager/issues)
- **Email**: support@clipboardmanager.com
- **Discord**: [Community Server](https://discord.gg/clipboardmanager)
