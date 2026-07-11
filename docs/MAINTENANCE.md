# Maintenance Guide

## Overview

Procedures for maintaining the Clipboard Manager app post-launch.

## Monitoring

### Crash Reporting (Sentry)
```swift
import Sentry

SentrySDK.start { options in
    options.dsn = "https://your-dsn@sentry.io/project-id"
    options.tracesSampleRate = 1.0
    options.environment = "production"
}
```

### Analytics (PostHog / Mixpanel)
```swift
import PostHog

let config = PostHogConfig(apiKey: "phc_your_key", host: "https://app.posthog.com")
PostHog.shared.setup(config)
```

### Key Metrics to Monitor
| Metric | Alert Threshold | Action |
|--------|-----------------|--------|
| Crash Rate | >1% | Investigate immediately |
| ANR Rate | >0.5% | Check performance |
| Sync Failures | >5% | Check Supabase status |
| Login Failures | >2% | Check Google OAuth |

## Updates

### iOS Updates
```bash
# Check for Xcode updates
softwareupdate --list

# Update SPM dependencies
xcodebuild -resolvePackageDependencies

# Build and test
xcodebuild test -scheme ClipboardManager-iOS -destination 'platform=iOS Simulator'
```

### macOS Updates
```bash
# Check compatibility
xcodebuild -scheme ClipboardManager -showBuildSettings | grep MACOSX_DEPLOYMENT_TARGET

# Build universal binary
xcodebuild -scheme ClipboardManager -arch arm64 -arch x86_64 build
```

## Database Migrations

### Realm Migration
```swift
let config = Realm.Configuration(
    schemaVersion: 2,
    migrationBlock: { migration, oldSchemaVersion in
        if oldSchemaVersion < 1 {
            migration.enumerateObjects(ofType: ClipboardItem.className()) { old, new in
                new?["isPinned"] = false
            }
        }
        if oldSchemaVersion < 2 {
            migration.enumerateObjects(ofType: ClipboardItem.className()) { old, new in
                new?["category"] = "Uncategorized"
            }
        }
    }
)
```

### Supabase Schema Changes
```sql
-- Migration: Add new column
ALTER TABLE clipboard_items ADD COLUMN device_name TEXT;

-- Migration: Add index
CREATE INDEX idx_clipboard_items_user_id ON clipboard_items(user_id);

-- Migration: Add RLS policy
ALTER TABLE clipboard_items ENABLE ROW LEVEL SECURITY;
```

## Backup & Recovery

### Database Backup
```bash
# Realm backup
cp ~/Library/Application\ Support/ClipboardManager/default.realm \
   ~/Backups/clipboard_$(date +%Y%m%d).realm

# Supabase backup
pg_dump -h db.supabase.co -U postgres -d your_db > backup.sql
```

### Recovery Procedures
1. **Data Loss**: Restore from backup
2. **Corruption**: Delete and re-sync
3. **Server Issues**: Enable offline mode

## Performance Tuning

### Realm Optimization
```swift
// Lazy loading
let items = realm.objects(ClipboardItem.self)
    .sorted(byKeyPath: "createdAt", ascending: false)
    .freeze() // For background threads

// Background writes
DispatchQueue.global().async {
    try? realm.write {
        realm.add(item)
    }
}
```

### Sync Optimization
```swift
// Batch sync
func syncBatch(_ items: [ClipboardItem]) async throws {
    let records = items.map { $0.toSupabaseRecord() }
    try await supabase.from("clipboard_items").upsert(records).execute()
}
```

## Troubleshooting

### Common Issues

#### Sync Not Working
1. Check network connectivity
2. Verify Supabase credentials
3. Check RLS policies
4. Review Realtime subscription

#### Login Failing
1. Verify Google OAuth config
2. Check redirect URI
3. Review Supabase Auth settings
4. Check keychain access

#### Performance Issues
1. Profile with Instruments
2. Check Realm query performance
3. Review sync frequency
4. Optimize UI rendering

## Security Updates

### Dependency Updates
```bash
# Check for vulnerabilities
swift package audit

# Update dependencies
swift package update

# Check outdated packages
swift package outdated
```

### Certificate Renewal
1. Generate new certificate
2. Update CI/CD secrets
3. Test signing
4. Deploy

## Support

### User Support Channels
- **Email**: support@clipboardmanager.com
- **GitHub Issues**: For bugs
- **Discord**: Community support

### Issue Triage
1. **P0 (Critical)**: Crashes, data loss → Fix within 24h
2. **P1 (High)**: Major feature broken → Fix within 1 week
3. **P2 (Medium)**: Minor issues → Fix in next release
4. **P3 (Low)**: Enhancements → Backlog

## Release Process

### Pre-Release Checklist
- [ ] All tests passing
- [ ] No known critical bugs
- [ ] Documentation updated
- [ ] Version bumped
- [ ] Changelog updated
- [ ] Screenshots updated

### Post-Release Checklist
- [ ] Monitor crash reports
- [ ] Check user feedback
- [ ] Verify analytics
- [ ] Update status page
