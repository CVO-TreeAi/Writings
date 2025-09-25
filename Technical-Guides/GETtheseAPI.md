# API Keys & Environment Setup - Fecon AI Customer Service App

## 🔑 Required API Keys & Accounts

### 1. Anthropic Claude API (CRITICAL)
**Purpose**: AI-powered parts identification, conversation, and analysis
- **Account**: Create at https://console.anthropic.com
- **Plan Required**: Pro ($20/month) or higher for production usage
- **Key Location**: API Keys section in Anthropic Console
- **Usage**: Vision API, conversation, streaming responses
- **Rate Limits**: 10,000 requests/minute (Pro), 50,000 requests/minute (Team)

```bash
# Environment Variable
export ANTHROPIC_API_KEY="sk-ant-api03-..."

# iOS Keychain Storage Key
let ANTHROPIC_API_KEY = "anthropic_api_key"
```

### 2. Convex Backend (CRITICAL)
**Purpose**: Real-time database, authentication, and backend functions
- **Account**: Create at https://www.convex.dev
- **Setup**: Run `npx convex dev` in terminal for guided setup
- **Features**: Real-time subscriptions, auth, file storage, functions
- **Pricing**: Free tier includes 1GB storage, 1M function calls/month

```bash
# Quick Setup Commands
npx convex dev
# Follow prompts to create account and deployment

# Environment Variables (Auto-generated)
export CONVEX_DEPLOYMENT="https://your-deployment-name.convex.cloud"
export CONVEX_DEPLOY_KEY="prod:your-deploy-key"

# iOS Configuration
let CONVEX_DEPLOYMENT_URL = "https://your-deployment-name.convex.cloud"
```

### 3. Apple Developer Account (CRITICAL)
**Purpose**: iOS app development, TestFlight, App Store distribution
- **Account**: https://developer.apple.com ($99/year)
- **Team ID**: Found in Apple Developer Console → Membership
- **Certificates**: iOS Development, iOS Distribution
- **App ID**: com.yourname.feconservice (configure in Apple Developer Portal)

```bash
# Environment Variables
export APPLE_DEVELOPER_TEAM_ID="YOUR_TEAM_ID"
export BUNDLE_ID="com.yourname.feconservice"

# Xcode Configuration
DEVELOPMENT_TEAM = YOUR_TEAM_ID
PRODUCT_BUNDLE_IDENTIFIER = com.yourname.feconservice
```

### 4. Apple Analytics & CloudKit (OPTIONAL)
**Purpose**: App analytics and user data sync
- **Setup**: Enable in Xcode → Signing & Capabilities
- **CloudKit**: Automatic with Apple Developer account
- **Analytics**: App Store Connect → App Analytics

```swift
// CloudKit Configuration
import CloudKit

let container = CKContainer.default()
let publicDatabase = container.publicCloudDatabase
let privateDatabase = container.privateCloudDatabase
```

## 🎨 Visual Assets & Brand Materials

### Fecon Brand Assets (Public Use)
**Source**: https://www.fecon.com (publicly available assets)
- **Logo Files**: Download from website source/media folder
- **Color Palette**: Extract from CSS/brand guidelines
- **Product Images**: Right-click save equipment photos
- **Brand Guidelines**: Follow existing website styling

```swift
// Brand Colors (Extract from fecon.com)
extension Color {
    static let feconPrimary = Color(hex: "#1B365D")  // Dark Blue
    static let feconOrange = Color(hex: "#FF6B35")   // Orange Accent
    static let feconGray = Color(hex: "#6B7280")     // Text Gray
    static let feconSuccess = Color(hex: "#10B981")  // Success Green
}
```

### Test Image Dataset
**Purpose**: Development and testing of parts identification
- **Source**: Fecon website product galleries, manuals, dealer sites
- **Organization**: Group by product line (Bull Hog, FTX, etc.)
- **Format**: High-resolution JPG/PNG, standardized naming

```
Assets/TestImages/
├── BullHog/
│   ├── rotor_systems/
│   ├── cutting_tools/
│   └── hydraulic_components/
├── Mulching_Tractors/
└── Parts_Catalog/
```

## 🛠️ Development Environment Setup

### 1. macOS Development Requirements
```bash
# Xcode (Latest Version)
# Install from Mac App Store or Apple Developer

# Command Line Tools
xcode-select --install

# Swift Package Manager (Included with Xcode)
swift --version

# Convex CLI
npm install -g convex

# Git (for version control)
git --version
```

### 2. iOS Simulator Configuration
```bash
# Install additional simulators
xcrun simctl list devices

# Recommended test devices:
# - iPhone 15 Pro (iOS 17.0) - Latest features
# - iPhone 14 (iOS 16.0) - Backwards compatibility  
# - iPad Pro (iOS 17.0) - Tablet experience
```

### 3. Project Dependencies (Swift Packages)
Add these packages in Xcode → Package Manager:

```swift
// Package.swift dependencies
dependencies: [
    // Convex iOS SDK
    .package(url: "https://github.com/get-convex/convex-swift", from: "0.1.0"),
    
    // HTTP Networking
    .package(url: "https://github.com/Alamofire/Alamofire", from: "5.8.0"),
    
    // Keychain Access
    .package(url: "https://github.com/kishikawakatsumi/KeychainAccess", from: "4.2.2"),
    
    // Image Handling
    .package(url: "https://github.com/onevcat/Kingfisher", from: "7.9.0"),
]
```

## 🧪 Testing & Simulation Setup

### 1. Mock Data Generation
**Purpose**: Simulate real equipment data without live API calls during development

```swift
// Mock Equipment Data
struct MockFeconData {
    static let sampleEquipment = [
        Equipment(model: "Bull Hog BH74", serialNumber: "BH74-2023-001", 
                 hydraulicFlow: 25.0, currentHours: 342.5),
        Equipment(model: "FTX300", serialNumber: "FTX-2023-045",
                 hydraulicFlow: 50.0, currentHours: 1205.2)
    ]
}
```

### 2. Local AI Testing
**Purpose**: Develop without consuming Claude API credits
```swift
// Development Mode Toggle
#if DEBUG
let USE_MOCK_AI = true
let ANTHROPIC_API_KEY = "mock_key_for_development"
#else
let USE_MOCK_AI = false
// Use real API key from Keychain
#endif
```

### 3. Parts Database Simulation
**Purpose**: Local parts catalog for offline development
```json
{
  "parts_catalog": [
    {
      "part_number": "BH46HM0-02",
      "name": "Viking Axe Cutting Tool",
      "compatibility": ["BH74", "BH85", "BH99"],
      "category": "cutting_tools",
      "price": 89.99,
      "in_stock": true
    }
  ]
}
```

## 🔒 Security Configuration

### 1. Keychain Setup (iOS)
```swift
import KeychainAccess

let keychain = Keychain(service: "com.yourname.feconservice")

// Store API keys securely
try keychain.set(apiKey, key: "anthropic_api_key")
try keychain.set(convexUrl, key: "convex_deployment_url")

// Retrieve with error handling
let apiKey = try? keychain.get("anthropic_api_key")
```

### 2. Network Security
```swift
// App Transport Security (Info.plist)
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <false/>
    <key>NSExceptionDomains</key>
    <dict>
        <key>convex.cloud</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <false/>
            <key>NSExceptionMinimumTLSVersion</key>
            <string>TLSv1.2</string>
        </dict>
    </dict>
</dict>
```

### 3. Privacy Permissions (Info.plist)
```xml
<!-- Camera Access -->
<key>NSCameraUsageDescription</key>
<string>Camera access is required to identify Fecon equipment parts and provide maintenance guidance.</string>

<!-- Microphone (for voice commands) -->
<key>NSMicrophoneUsageDescription</key>
<string>Microphone access enables voice commands for hands-free operation in the field.</string>

<!-- Photo Library -->
<key>NSPhotoLibraryUsageDescription</key>
<string>Photo library access allows you to analyze existing equipment photos for parts identification.</string>

<!-- Location (for job site tracking) -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>Location access helps track equipment usage patterns and optimize maintenance schedules.</string>
```

## 📱 iOS Configuration Files

### 1. Info.plist Key Settings
```xml
<key>CFBundleName</key>
<string>Fecon Service</string>

<key>CFBundleDisplayName</key>
<string>Fecon AI</string>

<key>CFBundleVersion</key>
<string>1.0.0</string>

<key>LSRequiresIPhoneOS</key>
<true/>

<key>UIRequiredDeviceCapabilities</key>
<array>
    <string>arm64</string>
    <string>camera-flash</string>
</array>

<!-- Supported Orientations -->
<key>UISupportedInterfaceOrientations</key>
<array>
    <string>UIInterfaceOrientationPortrait</string>
    <string>UIInterfaceOrientationLandscapeLeft</string>
    <string>UIInterfaceOrientationLandscapeRight</string>
</array>
```

### 2. Entitlements (App.entitlements)
```xml
<key>com.apple.developer.icloud-container-identifiers</key>
<array>
    <string>iCloud.com.yourname.feconservice</string>
</array>

<key>com.apple.developer.ubiquity-kvstore-identifier</key>
<string>$(TeamIdentifierPrefix)com.yourname.feconservice</string>

<key>com.apple.security.app-sandbox</key>
<true/>

<key>com.apple.security.network.client</key>
<true/>
```

## 🚀 Quick Start Checklist

### Before You Begin Development:
- [ ] **Apple Developer Account**: Active and Team ID retrieved
- [ ] **Anthropic API Key**: Created and tested with sample request
- [ ] **Convex Account**: Set up via `npx convex dev`
- [ ] **Xcode**: Latest version installed with iOS 17+ SDK
- [ ] **Brand Assets**: Downloaded from fecon.com and organized
- [ ] **Test Images**: Collected equipment photos for development

### Environment Validation Commands:
```bash
# Verify development environment
swift --version                    # Swift 5.9+
xcode-select --print-path         # Xcode tools installed
npx convex --version              # Convex CLI installed
curl -H "x-api-key: $ANTHROPIC_API_KEY" \
  https://api.anthropic.com/v1/messages  # API key works
```

### First Run Setup:
1. Clone/create project in Xcode
2. Add Swift packages via Package Manager
3. Configure environment variables in Xcode scheme
4. Run `npx convex dev` in terminal for backend setup
5. Build and run on simulator to verify setup

---

## 📞 Support Resources

### Documentation Links:
- **Anthropic API**: https://docs.anthropic.com/claude/reference/getting-started
- **Convex iOS SDK**: https://docs.convex.dev/client/ios
- **SwiftUI**: https://developer.apple.com/documentation/swiftui/
- **ARKit**: https://developer.apple.com/documentation/arkit/
- **Vision Framework**: https://developer.apple.com/documentation/vision/

### Troubleshooting:
- **API Key Issues**: Verify key format and rate limits in Anthropic console
- **Convex Connection**: Check deployment URL and internet connectivity
- **iOS Simulator**: Reset simulator if experiencing performance issues
- **Xcode Build Errors**: Clean build folder (Cmd+Shift+K) and rebuild

**This setup provides everything needed to build a production-quality Fecon AI customer service app that will impress and secure your employment opportunity.**