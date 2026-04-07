# Concepts — Advanced Native Features

> 📌 **Module context:** Module này survey 6 concepts chính về advanced native features trong Flutter, mapping từ code đã đọc ở [01-code-walk](./01-code-walk.md). Mỗi concept kèm FE bridge cho dev có background React/JavaScript.

---

## Concept 1: Camera Integration Pipeline 🔴 MUST-KNOW

**WHY:** Camera là feature phổ biến nhất trong mobile apps. Cần understand full pipeline từ capture → process → upload.

### Camera vs Gallery Selection

| Source | Pros | Cons | Use Case |
|--------|------|------|----------|
| **Camera** | Real-time, consistent format | User inconvenience | Profile photo, ID capture |
| **Gallery** | User choice, multiple attempts | Inconsistent format, large files | General photo selection |

### Image Processing Pipeline

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Capture   │────▶│    Crop      │────▶│  Compress    │
│  (Camera)    │     │   (Square)   │     │   (Upload)   │
└──────────────┘     └──────────────┘     └──────────────┘
       │                                        │
       └────────────┬───────────────────────────┘
                    ▼
            ┌──────────────┐
            │   Upload     │
            │   (API)      │
            └──────────────┘
```

### Image Quality vs Size

| Quality | File Size (1MB photo) | Use Case |
|---------|----------------------|----------|
| 100% | ~2-5 MB | Archive, print |
| 85% | ~500KB-1MB | Social media, profile |
| 70% | ~200-500KB | General upload, chat |
| 50% | ~100-200KB | Thumbnails, previews |

### Code Patterns

```dart
// Complete image upload pipeline
class ImageUploadPipeline {
  final CameraService _camera = CameraService();
  final ImageProcessingService _processor = ImageProcessingService();
  final ImageUploadService _uploader = ImageUploadService();
  
  Future<UploadResult> uploadAvatarFull({
    ImageSource source = ImageSource.gallery,
  }) async {
    // 1. Capture
    final image = await _camera.pickImage(source: source);
    if (image == null) return UploadResult.cancelled();
    
    // 2. Crop
    final cropped = await _processor.cropToSquare(image);
    if (cropped == null) return UploadResult.error('Crop failed');
    
    // 3. Compress
    final compressed = await _processor.compressForUpload(cropped);
    
    // 4. Upload
    final result = await _uploader.upload(compressed);
    if (result.isError) return result;
    
    // 5. Cleanup temp files
    await _cleanupTempFiles([image, cropped, compressed]);
    
    return result;
  }
  
  Future<void> _cleanupTempFiles(List<File> files) async {
    for (final file in files) {
      if (await file.exists()) {
        await file.delete();
      }
    }
  }
}
```

> 💡 **FE Perspective**
> **Flutter:** Image picker wraps camera/gallery APIs, requires platform config.
> **React/Vue tương đương:** `<input type="file" capture="user">` for camera, `accept="image/*"` for gallery.
> **Khác biệt quan trọng:** Flutter có better control over image processing pipeline.

---

## Concept 2: Location Services Architecture 🟡 SHOULD-KNOW

**WHY:** Location là feature phức tạp với nhiều permission states và accuracy options.

### Location Accuracy Levels

| Accuracy | Battery | Use Case | Response Time |
|----------|---------|----------|--------------|
| **Best** | High | Navigation, tracking | Fast |
| **High** | Medium | Active use | Fast |
| **Medium** | Low | General | Medium |
| **Low** | Very Low | City-level | Fast |
| **Best for Navigation** | High | Turn-by-turn | Fastest |

### Permission Flow

```
┌─────────────────┐
│ App Launch      │
└────────┬────────┘
         ▼
┌─────────────────┐
│ Check Service   │──No──▶ Show settings dialog
│ Enabled?        │
└────────┬────────┘
         │Yes
         ▼
┌─────────────────┐
│ Check Permission│──Denied──▶ Request Permission
└────────┬────────┘
         │Granted
         ▼
┌─────────────────┐
│ Get Location    │
└─────────────────┘
```

### Background Location Considerations

| Platform | Background Support | Limitations |
|----------|-------------------|-------------|
| **iOS** | Requires `always` permission | Limited updates, significant battery |
| **Android** | Requires `always` or foreground service | Android 10+ restricted |

> 💡 **FE Perspective**
> **Flutter:** `geolocator` wraps native location APIs with consistent interface.
> **React/Vue tương đương:** `navigator.geolocation.watchPosition()`.
> **Khác biệt quan trải:** Flutter có unified API across platforms, better background support.

---

## Concept 3: Biometric Authentication 🟡 SHOULD-KNOW

**WHY:** Biometrics provides secure, convenient authentication. Must handle fallback scenarios.

### Biometric Types

| Type | iOS | Android | Security |
|------|-----|---------|----------|
| **Fingerprint** | Touch ID | Fingerprint | Medium |
| **Face** | Face ID | Face Unlock | High |
| **Iris** | — | Iris Scanner | High |

### Authentication Strategies

```dart
enum AuthStrategy {
  biometricOnly,    // Chỉ biometric, không fallback
  biometricFirst,  // Thử biometric trước, fallback to PIN
  pinOnly,          // Chỉ PIN/password
}

class AuthService {
  Future<AuthResult> authenticate(AuthStrategy strategy) async {
    switch (strategy) {
      case AuthStrategy.biometricOnly:
        return await _biometricOnly();
      case AuthStrategy.biometricFirst:
        return await _biometricFirst();
      case AuthStrategy.pinOnly:
        return await _pinOnly();
    }
  }
  
  Future<AuthResult> _biometricFirst() async {
    final success = await _biometricAuth.authenticate(
      reason: 'Authenticate to continue',
    );
    
    if (success) return AuthResult.success();
    
    // Biometric failed → try PIN/password
    return await _pinAuth.authenticate();
  }
}
```

### Security Considerations

| Concern | Mitigation |
|---------|------------|
| Device not secured | Require PIN/pattern with biometric |
| Biometric not enrolled | Prompt to enroll or fallback |
| Too many failures | Lock out, require full login |
| Biometric changed | Re-authenticate with password |

> 💡 **FE Perspective**
> **Flutter:** `local_auth` uses Secure Enclave (iOS) / StrongBox (Android) for hardware-backed security.
> **React/Vue tương đương:** WebAuthn API with platform authenticators.
> **Khác biệt quan trọng:** Flutter biometric is hardware-backed, web is more flexible but less secure.
>
> ⚠️ **WebAuthn vs local_auth:** `local_auth` uses platform-native biometric APIs (TouchID/FaceID on iOS, Fingerprint on Android) with hardware-backed security (Secure Enclave/StrongBox). WebAuthn is a web standard for passwordless authentication using the browser's authenticator API. Both serve similar purposes but have different security models — `local_auth` is tied to device hardware, while WebAuthn works across devices via roaming authenticators.

---

## Concept 4: Push Notification Architecture 🔴 MUST-KNOW

**WHY:** Push notifications are critical for user engagement. Need to handle 3 app states correctly.

### App States & Notification Handling

```
┌─────────────────────────────────────────────────────────────┐
│                      Push Flow Diagram                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐                                           │
│  │   FCM       │                                           │
│  │   Server    │                                           │
│  └──────┬──────┘                                           │
│         │                                                   │
│         ▼                                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Device (FCM SDK)                        │   │
│  │                                                      │   │
│  │  ┌──────────────────────────────────────────────┐   │   │
│  │  │ App State: Foreground                         │   │   │
│  │  │ → onMessage stream fires                     │   │   │
│  │  │ → Show in-app notification OR custom UI      │   │   │
│  │  └──────────────────────────────────────────────┘   │   │
│  │                                                      │   │
│  │  ┌──────────────────────────────────────────────┐   │   │
│  │  │ App State: Background                         │   │   │
│  │  │ → Notification shown by system               │   │   │
│  │  │ → User tap → onMessageOpenedApp stream      │   │   │
│  │  └──────────────────────────────────────────────┘   │   │
│  │                                                      │   │
│  │  ┌──────────────────────────────────────────────┐   │   │
│  │  │ App State: Terminated                         │   │   │
│  │  │ → System shows notification                  │   │   │
│  │  │ → User tap → app launches → getInitialMessage│   │   │
│  │  └──────────────────────────────────────────────┘   │   │
│  │                                                      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Notification Types

| Type | Trigger | Visibility |
|------|---------|------------|
| **Data Message** | App in foreground | Custom handling |
| **Notification Message** | System notification shown | Automatic |
| **Notification + Data** | Both | System + custom |

### Local vs Remote Notifications

| Aspect | Local Notifications | Remote (FCM/APNs) |
|--------|---------------------|-------------------|
| Trigger | App-generated | Server-pushed |
| Internet | Not required | Required |
| Personalization | Limited | Full (per-user) |
| Use Case | Reminders, timers | Alerts, updates |

> 💡 **FE Perspective**
> **Flutter:** Firebase Messaging handles FCM (Android) và APNs (iOS) via unified API.
> **React/Vue tương đương:** Web Push API with Service Worker.
> **Khác biệt quan trọng:** Mobile push can work when app closed, web push requires browser open.

---

## Concept 5: Deep Linking Types 🟡 SHOULD-KNOW

**WHY:** Deep links connect your app to specific content from external sources (URLs, QR codes, other apps).

### Deep Link Types Comparison

| Type | Format | Security | Setup Complexity |
|------|--------|----------|-------------------|
| **Custom Scheme** | `myapp://path` | Low (any app can claim) | Easy |
| **Universal Links** | `https://myapp.com/path` | High (verified domain) | Medium |
| **App Links** | `https://myapp.com/path` | High (verified domain) | Medium |

### Android App Links Setup

```xml
<!-- AndroidManifest.xml -->
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <data 
        android:scheme="https"
        android:host="myapp.com"/>
</intent-filter>
```

### iOS Universal Links Setup

```xml
<!-- Associated Domains (entitlements) -->
<key>com.apple.developer.associated-domains</key>
<array>
    <string>applinks:myapp.com</string>
</array>
```

### Routing Implementation

```dart
// Deep link route definitions
class DeepLinkRoutes {
  static const routePatterns = {
    r'/profile/(\w+)': (matches) => ProfileRoute(userId: matches[1]),
    r'/order/(\w+)': (matches) => OrderDetailRoute(orderId: matches[1]),
    r'/invite\?code=(\w+)': (matches) => InviteRoute(code: matches['code']),
  };
  
  static Route? parseAndNavigate(Uri uri) {
    for (final entry in routePatterns.entries) {
      final regex = RegExp(entry.key);
      final match = regex.firstMatch(uri.path);
      if (match != null) {
        final groups = match.groups(List.generate(match.groupCount, (i) => i + 1));
        return entry.value(groups);
      }
    }
    return null;
  }
}
```

> 💡 **FE Perspective**
> **Flutter:** Deep links work like URL routing in web apps.
> **React/Vue tương đương:** `react-router` với nested routes, `Linking` API (React Native).
> **Khác biệt quan trọng:** Mobile deep links can launch app from closed state.

---

## Concept 6: Permission Request Flow 🟢 AI-GENERATE

**WHY:** Permissions are gatekeepers for native features. Understanding the flow helps handle edge cases.

### Permission Request Best Practices

```dart
class PermissionHelper {
  /// Request permission với explanation
  Future<PermissionStatus> requestWithExplanation({
    required Permission permission,
    required String title,
    required String explanation,
    required String buttonText,
  }) async {
    // Check current status
    final status = await permission.status;
    
    if (status.isGranted) {
      return status;
    }
    
    if (status.isDenied) {
      // First time or previously denied (not permanently)
      // Show explanation dialog first
      final shouldRequest = await _showExplanationDialog(
        title: title,
        explanation: explanation,
        buttonText: buttonText,
      );
      
      if (shouldRequest) {
        return await permission.request();
      }
      return PermissionStatus.denied;
    }
    
    if (status.isPermanentlyDenied) {
      // User denied permanently → open settings
      await _openAppSettings();
      return PermissionStatus.denied;
    }
    
    return status;
  }
  
  Future<bool> _showExplanationDialog({
    required String title,
    required String explanation,
    required String buttonText,
  }) async {
    // Show dialog with explanation
    // Return true if user wants to proceed
    return await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: Text(title),
        content: Text(explanation),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Cancel'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, true),
            child: Text(buttonText),
          ),
        ],
      ),
    ) ?? false;
  }
  
  Future<void> _openAppSettings() async {
    await Geolocator.openAppSettings();
  }
}
```

### Permission Status States

| Status | Meaning | Action |
|--------|---------|--------|
| `denied` | First request or user denied | Show explanation, request again |
| `deniedForever` | User permanently denied | Open app settings |
| `restricted` | System restriction (parental controls) | Inform user |
| `granted` | Permission given | Proceed with feature |

> 💡 **FE Perspective**
> **Flutter:** Permission packages provide unified API across platforms.
> **React/Vue tương đương:** Browser Permission API (`navigator.permissions.query()`).
> **Khác biệt quan trọng:** Mobile permissions are more granular and persistent.

---

## Concept Map — How They Connect

```
Concept 1: Camera Pipeline    → Pick → Crop → Compress → Upload
Concept 2: Location API      → Check → Request → Track
Concept 3: Biometric Auth    → Check → Authenticate → Fallback
Concept 4: Push Notifications → Token → Foreground → Background → Terminated
Concept 5: Deep Linking      → URI → Parse → Navigate
Concept 6: Permissions       → Check → Explain → Request → Handle Denial
```

**Integration Flow:**

```
User Action → Permission Check → Native Feature → Process → API → State Update
```

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
