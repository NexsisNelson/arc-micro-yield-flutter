# Arc Micro-Yield Flutter - Setup & Development Guide

## Prerequisites

### System Requirements

- **Windows/Mac/Linux** with 2GB+ RAM
- **Git** for version control
- **Flutter SDK** (≥3.11.0)
- **Dart SDK** (≥3.11.0)

### Platform-Specific Requirements

**Android Development:**
- Android Studio or Android SDK command-line tools
- Minimum API Level 21 (Android 5.0)
- Android Virtual Device (emulator) or physical device

**iOS Development:**
- Xcode 14+ (macOS only)
- CocoaPods
- Minimum deployment target: iOS 12.0
- Physical device or simulator

**Web Development:**
- Modern web browser (Chrome, Firefox, Safari, Edge)
- No additional tools required

**Desktop Development:**
- **Windows:** Windows SDK
- **macOS:** Xcode
- **Linux:** Linux development tools

## Installation

### 1. Install Flutter SDK

**Windows:**
```bash
# Download Flutter from https://flutter.dev/docs/get-started/install/windows
# Extract to a folder (e.g., C:\flutter)
# Add Flutter to PATH:
setx PATH "%PATH%;C:\flutter\bin"
```

**macOS:**
```bash
# Using Homebrew
brew install flutter

# Or download and extract
cd ~
git clone https://github.com/flutter/flutter.git -b stable
export PATH="$PATH:$HOME/flutter/bin"
```

**Linux:**
```bash
cd ~
git clone https://github.com/flutter/flutter.git -b stable
export PATH="$PATH:$HOME/flutter/bin"
```

### 2. Verify Installation

```bash
flutter doctor
# Check for any missing dependencies

flutter --version
# Should show version 3.11.0 or higher
```

### 3. Clone the Repository

```bash
git clone https://github.com/NexsisNelson/arc-micro-yield-flutter.git
cd arc-micro-yield-flutter
```

### 4. Install Dependencies

```bash
# Download pub packages
flutter pub get

# Upgrade to latest compatible versions
flutter pub upgrade

# Get Android dependencies (if developing for Android)
flutter pub get android
```

## Running the Application

### Mobile (Android/iOS)

**Android:**
```bash
# List available Android devices
flutter devices

# Run on default device/emulator
flutter run

# Run in verbose mode (for debugging)
flutter run -v

# Run with specific device
flutter run -d emulator-5554
```

**iOS:**
```bash
# Build and run on iOS simulator
flutter run

# Run on physical device
flutter run -d <device-id>

# Verbose output
flutter run -v
```

### Web

```bash
# Run on Chrome (default)
flutter run -d chrome

# Run on Firefox
flutter run -d firefox

# Run on Safari (macOS only)
flutter run -d safari

# Run on web-server (for debugging)
flutter run -d web-server

# Access at http://localhost:52682
```

### Desktop

**Windows:**
```bash
flutter run -d windows
```

**macOS:**
```bash
flutter run -d macos
```

**Linux:**
```bash
flutter run -d linux
```

## Development Workflow

### Hot Reload

```bash
# While app is running, press 'r' to hot reload
r   - Hot reload (changes in code)
R   - Hot restart (full app restart)
q   - Quit
```

### Debug Mode

```bash
flutter run
# Default: debug mode with hot reload enabled
```

### Release Mode

```bash
flutter run --release
# Optimized performance, no debugging features
```

### Profile Mode

```bash
flutter run --profile
# Performance profiling without debugging overhead
```

## Building for Distribution

### Android

**APK (direct installation):**
```bash
flutter build apk --release
# Output: build/app/outputs/flutter-app.apk
```

**App Bundle (Google Play Store):**
```bash
flutter build appbundle --release
# Output: build/app/outputs/bundle/release/app-release.aab
```

### iOS

**Development Build:**
```bash
flutter build ios --debug
```

**Release Build:**
```bash
flutter build ios --release
# Open build/ios/Runner.xcworkspace in Xcode
```

### Web

```bash
# Build for web (default: HTML renderer)
flutter build web --release
# Output: build/web/

# Build with CanvasKit renderer (better graphics)
flutter build web --release --web-renderer canvaskit

# Deploy to server
# Copy contents of build/web/ to your web server
```

### Windows

```bash
flutter build windows --release
# Output: build/windows/runner/Release/
```

### macOS

```bash
flutter build macos --release
# Output: build/macos/Build/Products/Release/
```

### Linux

```bash
flutter build linux --release
# Output: build/linux/x64/release/bundle/
```

## Testing

### Unit Tests

```bash
flutter test test/widget_test.dart
```

### All Tests

```bash
flutter test
# Run all tests in test/ directory
```

### Test with Coverage

```bash
flutter test --coverage
# Generates coverage/lcov.info

# View coverage report
genhtml coverage/lcov.info -o coverage/report
open coverage/report/index.html
```

## Code Analysis

### Lint Analysis

```bash
flutter analyze
# Check for code style and potential issues
```

### Format Code

```bash
flutter format lib/
# Automatically format Dart files

flutter format .
# Format all files in project
```

### Fix Issues

```bash
dart fix lib/
# Automatically apply fixes for detected issues
```

## Debugging

### Debug Console

```bash
# Print debug messages
print('Debug message');
debugPrint('Debug print');

# Using debugger
import 'dart:developer' as developer;
developer.debugger();
```

### DevTools

```bash
# Launch DevTools
flutter pub global activate devtools
devtools

# Or while app is running
flutter run
# Press 'd' to open DevTools
```

### Logging

```dart
import 'package:flutter/foundation.dart';

if (kDebugMode) {
  print('Debug only');
}

if (kReleaseMode) {
  print('Release only');
}
```

## IDE Setup

### VS Code

**Extensions to install:**
- Flutter (Dart Code)
- Dart (Dart Code)
- Awesome Flutter Snippets

**launch.json configuration:**
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Flutter",
      "request": "launch",
      "type": "dart",
      "program": "lib/main.dart",
      "args": ["-d", "chrome"]
    }
  ]
}
```

### Android Studio

1. Install Flutter plugin
2. Create new Flutter project
3. Configure Android SDK path

### Xcode (macOS)

1. Open project: `ios/Runner.xcworkspace`
2. Select target device
3. Build and run

## Common Issues

### Issue: "flutter: command not found"

**Solution:**
- Add Flutter to PATH
- Verify installation: `flutter doctor`

### Issue: Android SDK not found

**Solution:**
```bash
flutter config --android-sdk /path/to/android/sdk
flutter doctor --android-licenses
```

### Issue: Pod install fails (iOS)

**Solution:**
```bash
cd ios
pod deintegrate
pod install
cd ..
```

### Issue: Build cache issues

**Solution:**
```bash
flutter clean
flutter pub get
flutter run
```

## Web3 Integration Setup

### web3dart Configuration

```dart
import 'package:web3dart/web3dart.dart';
import 'package:http/http.dart';

// Initialize Web3 client
final ethClient = Web3Client(
  'https://arc-rpc-endpoint.example.com',
  Client(),
);

// Connect to smart contract
final contractABI = [
  // Contract ABI here
];

final contract = DeployedContract(
  ContractAbi.fromJson(jsonEncode(contractABI), 'MicroYieldVault'),
  EthereumAddress.fromHex('0x...'),
);
```

## Performance Optimization

### Release Build Options

```bash
# Optimize for size
flutter build apk --release --split-per-abi

# Enable obfuscation
flutter build apk --release --obfuscate --split-debug-info=.
```

### DevTools Performance Profiler

1. Run app: `flutter run`
2. Open DevTools: Press 'd'
3. Go to Performance tab
4. Record and analyze

## Deployment Checklist

- [ ] Run `flutter test` - all tests pass
- [ ] Run `flutter analyze` - no errors
- [ ] Update version in `pubspec.yaml`
- [ ] Build release version
- [ ] Test on multiple devices
- [ ] Review error handling
- [ ] Check permissions (Android/iOS)
- [ ] Set up crash reporting
- [ ] Create app icons and splash screens
- [ ] Deploy to store

## Useful Commands

```bash
# View all available commands
flutter help

# Check project status
flutter doctor

# List connected devices
flutter devices

# Update Flutter
flutter upgrade

# Clean build artifacts
flutter clean

# Analyze app size
flutter build apk --analyze-size
```

## Resources

- [Flutter Documentation](https://flutter.dev/docs)
- [Dart Language Tour](https://dart.dev/guides/language/language-tour)
- [Flutter Samples](https://github.com/flutter/samples)
- [web3dart Documentation](https://pub.dev/packages/web3dart)

## Getting Help

- Check Flutter documentation
- Search [Stack Overflow](https://stackoverflow.com/questions/tagged/flutter)
- Open issue on GitHub repository
- Check [Flutter GitHub Issues](https://github.com/flutter/flutter/issues)

---

Happy coding! 🚀
