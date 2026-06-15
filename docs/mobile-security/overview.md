# Mobile Security

## Overview

Mobile application security addresses the unique risks introduced by iOS and Android platforms: untrusted distribution channels, device diversity, sensitive local data storage, wireless communication, and the physical exposure of devices. The OWASP Mobile Application Security Verification Standard (MASVS) and its companion testing guide (MASTG) provide the authoritative reference for mobile security.

---

## Mobile Threat Model

Mobile applications face threats distinct from web applications:

| Threat Category | Examples |
|----------------|---------|
| Insecure data storage | Sensitive data in plaintext, unprotected SharedPreferences, SQLite |
| Insecure communication | Cleartext HTTP, disabled certificate validation, weak TLS |
| Authentication weaknesses | Biometric bypass, weak PIN policies, no session expiration |
| Code quality issues | Hardcoded credentials, overly broad permissions, debug builds in production |
| Reverse engineering | Decompilation, runtime instrumentation, code injection |
| Client-side injection | SQLite injection, JavaScript injection in WebViews |
| Physical device compromise | Unencrypted storage, no screen lock |

---

## OWASP Mobile Top 10 (2024)

| Rank | Category |
|------|----------|
| M1 | Improper Credential Usage |
| M2 | Inadequate Supply Chain Security |
| M3 | Insecure Authentication/Authorization |
| M4 | Insufficient Input/Output Validation |
| M5 | Insecure Communication |
| M6 | Inadequate Privacy Controls |
| M7 | Insufficient Binary Protections |
| M8 | Security Misconfiguration |
| M9 | Insecure Data Storage |
| M10 | Insufficient Cryptography |

---

## Android Security

### Android Security Architecture

Android uses a layered security model:

- **Application sandbox**: Each app runs in an isolated process with a unique UID. Apps cannot access each other's data without explicit sharing through IPC.
- **Permissions model**: Apps must declare permissions in `AndroidManifest.xml` and request runtime permissions for sensitive capabilities.
- **SELinux**: Android uses SELinux in enforcing mode from Android 5.0+, defining mandatory access control policies for the OS.
- **Verified boot**: Ensures the system has not been tampered with since the device shipped.
- **Keystore**: Hardware-backed cryptographic key storage (Android Keystore System).

### Android Data Storage

```java
// INSECURE: Storing sensitive data in SharedPreferences without encryption
SharedPreferences prefs = getSharedPreferences("userdata", MODE_PRIVATE);
prefs.edit().putString("api_key", "SECRET_KEY").apply();

// SECURE: Use EncryptedSharedPreferences
MasterKey masterKey = new MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build();

SharedPreferences encryptedPrefs = EncryptedSharedPreferences.create(
    context,
    "encrypted_userdata",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
);
encryptedPrefs.edit().putString("api_key", "SECRET_KEY").apply();
```

**Data storage locations and their risks:**

| Location | Access | Risk |
|----------|--------|------|
| Internal storage (app-private) | App only | Low (sandboxed) |
| External storage / SD card | Any app with READ_EXTERNAL_STORAGE | High (shared) |
| SharedPreferences | App only | Medium (readable if device rooted or backup enabled) |
| SQLite database | App only | Medium (plaintext unless encrypted) |
| Content providers | Controllable | Varies based on exported status |
| Logcat | All apps (pre-Android 4) | High — avoid sensitive data in logs |

```java
// INSECURE: Logging sensitive data
Log.d("Auth", "User logged in with password: " + password);

// SECURE: Never log credentials; redact or omit
Log.d("Auth", "User authentication successful for UID: " + userId);
```

### Android Network Security

```xml
<!-- res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!-- Enforce HTTPS for all connections -->
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <!-- Trust system CAs only (default) -->
            <certificates src="system"/>
        </trust-anchors>
    </base-config>
    
    <!-- Certificate pinning for your API -->
    <domain-config>
        <domain includeSubdomains="true">api.yourapp.com</domain>
        <pin-set expiration="2025-01-01">
            <!-- SHA-256 fingerprint of the leaf or CA certificate -->
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
            <!-- Backup pin (required) -->
            <pin digest="SHA-256">BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

```xml
<!-- AndroidManifest.xml -->
<application
    android:networkSecurityConfig="@xml/network_security_config"
    android:usesCleartextTraffic="false">
```

### Android Permissions

```xml
<!-- Request only what you need -->
<!-- DANGEROUS permissions require runtime request -->
<uses-permission android:name="android.permission.CAMERA"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>

<!-- Avoid requesting permissions that are not necessary -->
<!-- Do not use: -->
<!-- <uses-permission android:name="android.permission.READ_CONTACTS"/> -->
<!-- unless contact access is a core feature -->
```

```java
// Runtime permission request (required for dangerous permissions on Android 6.0+)
if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
        != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(this,
            new String[]{Manifest.permission.CAMERA},
            CAMERA_PERMISSION_REQUEST);
}
```

---

## iOS Security

### iOS Security Architecture

- **App sandbox**: Each iOS app runs in its own sandbox directory; cannot access other apps' data.
- **Data Protection API**: File encryption tied to device passcode and biometrics. Files can be protected at different levels.
- **Keychain**: Secure storage for small sensitive items (passwords, tokens, certificates).
- **App Transport Security (ATS)**: Enforces HTTPS with TLS 1.2+ and forward secrecy by default.
- **Code signing**: All code must be signed with an Apple-issued certificate.

### iOS Data Protection

```swift
// Store sensitive data in Keychain, not UserDefaults
import Security

class KeychainHelper {
    
    static func save(key: String, value: String) throws {
        let data = value.data(using: .utf8)!
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecValueData as String: data,
            // Most secure: item only accessible when device is unlocked
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]
        
        SecItemDelete(query as CFDictionary)  // Remove existing
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else {
            throw KeychainError.saveFailed(status)
        }
    }
    
    static func retrieve(key: String) throws -> String? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        
        guard status == errSecSuccess,
              let data = result as? Data,
              let value = String(data: data, encoding: .utf8) else {
            return nil
        }
        return value
    }
}
```

**Keychain accessibility options:**

| Option | When Accessible | Device Migration |
|--------|----------------|-----------------|
| `kSecAttrAccessibleWhenUnlocked` | Device unlocked | Migrates to new device |
| `kSecAttrAccessibleAfterFirstUnlock` | After first unlock (persists through lock) | Migrates |
| `kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly` | Only if passcode set; unlocked | Does not migrate |
| `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` | Unlocked; best for secrets | Does not migrate |

### App Transport Security

```xml
<!-- Info.plist: Proper ATS configuration -->
<!-- Do NOT add NSAllowsArbitraryLoads = YES unless absolutely required -->
<!-- If required for specific domains, limit scope -->
<key>NSAppTransportSecurity</key>
<dict>
    <!-- Allow specific exception domain only -->
    <key>NSExceptionDomains</key>
    <dict>
        <key>legacy-api.example.com</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSExceptionMinimumTLSVersion</key>
            <string>TLSv1.2</string>
            <key>NSIncludesSubdomains</key>
            <true/>
        </dict>
    </dict>
</dict>
```

### iOS Biometric Authentication

```swift
import LocalAuthentication

func authenticateWithBiometrics(completion: @escaping (Bool, Error?) -> Void) {
    let context = LAContext()
    var error: NSError?
    
    // Check if biometric authentication is available
    guard context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) else {
        completion(false, error)
        return
    }
    
    context.evaluatePolicy(
        .deviceOwnerAuthenticationWithBiometrics,
        localizedReason: "Authenticate to access your account"
    ) { success, authError in
        DispatchQueue.main.async {
            completion(success, authError)
        }
    }
}
```

---

## Mobile Application Security Testing

### Static Analysis (Mobile SAST)

**Android:**
```bash
# APK analysis
# Extract APK contents
apktool d app.apk -o app_extracted/

# Decompile to Java source
jadx -d output_dir/ app.apk

# Search for hardcoded secrets
grep -r "api_key\|secret\|password\|token" app_extracted/
grep -r "http://" app_extracted/  # Cleartext URLs

# Check manifest for dangerous configurations
cat app_extracted/AndroidManifest.xml | grep -E "android:debuggable|android:allowBackup|android:exported"
# android:debuggable="true" in production = critical finding
# android:allowBackup="true" = allows adb backup of app data
```

**iOS:**
```bash
# IPA analysis (unzipped)
unzip app.ipa -d app_extracted/
cd app_extracted/Payload/App.app/

# Check for debugging enabled
otool -vh AppBinary | grep PIE  # Should have PIE enabled
otool -vh AppBinary | grep stack  # Stack canaries

# Search for hardcoded strings
strings AppBinary | grep -E "http://|api_key|password|secret"

# Check for encryption
otool -l AppBinary | grep -A4 LC_ENCRYPTION_INFO  # Should show cryptid = 1
```

### Dynamic Analysis with Frida

Frida is a dynamic instrumentation toolkit that allows runtime analysis and manipulation of mobile applications.

```bash
# Install Frida
pip install frida-tools

# List running processes
frida-ps -U  # USB connected device
frida-ps -D <device_id>  # Specific device

# Intercept a function
frida -U -l hook_script.js com.example.app
```

```javascript
// hook_script.js - intercept SSL pinning
Java.perform(function() {
    // Bypass OkHttp certificate pinning
    var OkHttpClient = Java.use('okhttp3.OkHttpClient');
    var CertificatePinner = Java.use('okhttp3.CertificatePinner');
    
    CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function(hostname, peerCertificates) {
        console.log('[*] Bypassing certificate pinning for: ' + hostname);
        return;  // Skip the check
    };
    
    // Intercept and log SharedPreferences reads
    var SharedPreferences = Java.use('android.app.SharedPreferencesImpl');
    SharedPreferences.getString.implementation = function(key, defValue) {
        var result = this.getString(key, defValue);
        if (result != null) {
            console.log('[*] SharedPreferences.getString(' + key + ') = ' + result);
        }
        return result;
    };
});
```

### OWASP MASVS Compliance

The MASVS defines security requirements in two levels:

**MASVS-L1 (Standard Security):** Baseline for all mobile applications.
**MASVS-L2 (Defense in Depth):** Additional controls for high-value applications (banking, healthcare).

| Category | MASVS-L1 | MASVS-L2 |
|----------|---------|---------|
| Storage | Encrypt sensitive data at rest | Hardware-backed key storage |
| Cryptography | Use standard algorithms, no hardcoded keys | Additional key management controls |
| Authentication | Proper session management | Biometric authentication with hardware key |
| Network | TLS enforced, certificate validation | Certificate pinning |
| Platform | Permissions minimal, IPC secured | Anti-tampering, biometric validation |
| Code | No debug, no sensitive data in logs | Root/jailbreak detection, obfuscation |

---

## Mobile DevSecOps

### Automated Mobile Security Scanning

```yaml
# GitHub Actions: Mobile security scan
name: Mobile Security Scan
on: [push, pull_request]

jobs:
  android_scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: MobSF Static Analysis
        run: |
          docker run -it --rm \
            -v $(pwd)/app.apk:/tmp/app.apk \
            opensecurity/mobile-security-framework-mobsf:latest \
            python manage.py runscans --apk /tmp/app.apk
      
      - name: Check for hardcoded secrets
        run: |
          pip install trufflehog3
          trufflehog3 --format json . > secrets_report.json
```

**MobSF (Mobile Security Framework)**: Open source tool for automated static and dynamic analysis of Android and iOS applications.

```bash
# Run MobSF via Docker
docker pull opensecurity/mobile-security-framework-mobsf:latest
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest

# Access at: http://localhost:8000
# Upload APK or IPA for analysis
```
