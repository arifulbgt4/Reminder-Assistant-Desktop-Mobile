# Design System

## Design Principles

1. **Simplicity** - Clean, uncluttered interface
2. **Consistency** - Unified experience across Mac and iOS
3. **Accessibility** - Support for VoiceOver, Dynamic Type
4. **Performance** - Smooth 60fps animations

## Color Palette

### Primary Colors
| Name | Hex | Usage |
|------|-----|-------|
| Primary Blue | #007AFF | Actions, links, selections |
| Secondary Gray | #8E8E93 | Secondary text, borders |
| Background | #FFFFFF | Main background |
| Surface | #F2F2F7 | Cards, surfaces |
| Error Red | #FF3B30 | Errors, destructive actions |
| Success Green | #34C759 | Success states |
| Warning Orange | #FF9500 | Warnings |

### Dark Mode
| Name | Hex | Usage |
|------|-----|-------|
| Primary Blue | #0A84FF | Actions, links |
| Background | #000000 | Main background |
| Surface | #1C1C1E | Cards, surfaces |
| Secondary | #636366 | Secondary text |

## Typography

### Font Hierarchy
```swift
// Large Title - Screen titles
.font(.largeTitle)
.fontWeight(.bold)

// Title 1 - Section headers
.font(.title)
.fontWeight(.semibold)

// Title 2 - Card titles
.font(.title2)
.fontWeight(.medium)

// Body - Main content
.font(.body)

// Caption - Secondary text
.font(.caption)
.foregroundColor(.secondary)
```

## Layout

### Spacing
- Extra Small: 4pt
- Small: 8pt
- Medium: 16pt
- Large: 24pt
- Extra Large: 32pt

### Corner Radius
- Small: 8pt (buttons, inputs)
- Medium: 12pt (cards)
- Large: 16pt (sheets)

## Components

### Clipboard Item Card
```
┌─────────────────────────────────────┐
│ 📋 [Category]              ⋮ •••   │
│                                     │
│ Clipboard content text here that    │
│ can be multiple lines...            │
│                                     │
│ 📱 Mac • 2 min ago                  │
└─────────────────────────────────────┘
```

### Category Pill
```
┌──────────────┐
│ 📂 All Items │
└──────────────┘
```

### Login Button (Google)
```
┌────────────────────────────────────┐
│      🔵 Sign in with Google        │
└────────────────────────────────────┘
```

## Animations

### Transitions
- View push: `.move(edge: .trailing)`
- Modal present: `.slide()`
- Delete: `.opacity` + `.scale`

### Haptics
- Copy to clipboard: `.light`
- Delete: `.medium`
- Sync success: `.success`

## Icons

### SF Symbols Usage
| Action | Symbol |
|--------|--------|
| Clipboard | `doc.on.clipboard` |
| Copy | `doc.on.doc` |
| Pin | `pin` |
| Delete | `trash` |
| Category | `folder` |
| Settings | `gearshape` |
| Sync | `arrow.triangle.2.circlepath` |
| Sign In | `person.circle` |

## Platform Adaptations

### macOS
- Window minimum size: 400x300
- Menu bar integration
- Keyboard shortcuts (⌘+C, ⌘+V)
- Drag and drop support

### iOS
- Bottom navigation
- Swipe actions (delete, pin)
- Long press for context menu
- Dynamic Island support (iOS 16+)

## Accessibility

### VoiceOver Labels
```swift
.accessibilityLabel("Clipboard item: \(content)")
.accessibilityHint("Double tap to copy")
```

### Dynamic Type
```swift
.font(.body)  // Scales with user preference
```

### Contrast
- Minimum contrast ratio: 4.5:1
- WCAG AA compliance
