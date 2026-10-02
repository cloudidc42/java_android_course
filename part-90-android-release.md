# Part 90: Android Release & Google Play Store
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 90.1 App Bundle (AAB) vs APK

```
APK (Android Package):
- ติดตั้งได้โดยตรง
- มีทุก architecture และ resources
- ขนาดใหญ่กว่า (ผู้ใช้ download ทั้งหมด)

AAB (Android App Bundle):
- Google Play สร้าง APK ที่เหมาะกับอุปกรณ์แต่ละเครื่อง
- ลดขนาด download 15-35%
- Required สำหรับ new apps บน Google Play (2021+)
- รองรับ Dynamic Delivery (install feature on demand)
```

```groovy
// Build AAB
./gradlew bundleRelease

// Build APK (for direct install or other stores)
./gradlew assembleRelease
```

---

## 90.2 Release Checklist

```java
// 1. Update version in build.gradle
android {
    defaultConfig {
        versionCode 12          // increment for every release
        versionName "2.3.1"    // semantic versioning: major.minor.patch
    }
}

// 2. Turn off debug features
// In BuildConfig, ENABLE_LOGGING should be false in prod flavor

// 3. ProGuard enabled
// minifyEnabled true, shrinkResources true in release build type

// 4. Remove test accounts / hardcoded secrets
// Check: no API keys, passwords, debug tokens in code

// 5. Check permissions (only what you need)
// Review AndroidManifest.xml - remove unused permissions

// 6. Test on multiple API levels
// Test on API 21 (minimum), API 28, API 33, API 34

// 7. Network Security
// Certificate pinning enabled for production API
// cleartext disabled

// 8. Crash reporting enabled
// FirebaseCrashlytics initialized

// 9. Analytics enabled
// FirebaseAnalytics initialized

// 10. Validate In-App Purchases on server side
// Never trust client-side only purchase verification
```

---

## 90.3 App Signing

```bash
# Generate release keystore (do this ONCE, keep safe!)
keytool -genkey -v \
    -keystore release.jks \
    -alias myapp_key \
    -keyalg RSA \
    -keysize 2048 \
    -validity 10000

# IMPORTANT:
# - Back up release.jks in multiple secure locations
# - If you lose the keystore, you CANNOT update your app on Play Store
# - Never commit keystore to git

# View keystore info
keytool -list -v -keystore release.jks -alias myapp_key

# Get SHA-256 fingerprint (for App Links, Firebase, etc.)
keytool -list -v -keystore release.jks -alias myapp_key | grep SHA256
```

```groovy
// build.gradle - sign with keystore
android {
    signingConfigs {
        release {
            storeFile     file('../keystore/release.jks')
            storePassword 'yourStorePassword'
            keyAlias      'myapp_key'
            keyPassword   'yourKeyPassword'
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

---

## 90.4 Play Store Upload via API

```groovy
// build.gradle - Google Play plugin
plugins {
    id 'com.github.triplet.play' version '3.9.1'
}

play {
    serviceAccountCredentials.set(file("../play-account-key.json"))
    track.set("internal")  // internal, alpha, beta, production
    releaseStatus.set(com.github.triplet.gradle.androidpublisher.ReleaseStatus.DRAFT)
}

// Upload AAB to internal track
// ./gradlew publishReleaseBundle
```

---

## 90.5 Release Notes & Store Listing

```
Play Store Store Listing Checklist:
✅ App icon: 512x512 PNG (no alpha)
✅ Feature graphic: 1024x500 PNG/JPG
✅ Screenshots: phone (min 2), tablet optional
✅ Short description: ≤80 characters
✅ Full description: ≤4000 characters
✅ Privacy Policy URL (required for apps with personal data)
✅ Content rating questionnaire completed
✅ Data safety section filled (what data you collect/share)

Release Notes (Thai):
```

```xml
<!-- fastlane/metadata/android/th-TH/changelogs/12.txt -->
เวอร์ชัน 2.3.1
- แก้ไขปัญหาการโหลดรูปภาพในหน้าสินค้า
- ปรับปรุงประสิทธิภาพการค้นหา
- เพิ่มการรองรับ Android 14
- แก้ไขข้อผิดพลาดเล็กน้อย
```

---

## 90.6 Dynamic Feature Modules

```groovy
// settings.gradle
include ':app', ':feature-ar-preview'

// feature-ar-preview/build.gradle
plugins {
    id 'com.android.dynamic-feature'
}

android {
    compileSdk 34
}

dependencies {
    implementation project(':app')
    implementation 'com.google.ar:core:1.43.0'
}
```

```java
// Install feature on demand
SplitInstallManager splitInstallManager = SplitInstallManagerFactory.create(this);

SplitInstallRequest request = SplitInstallRequest.newBuilder()
    .addModule("feature-ar-preview")
    .build();

splitInstallManager.startInstall(request)
    .addOnSuccessListener(sessionId -> {
        Log.d("DFM", "AR feature installing, sessionId: " + sessionId);
    })
    .addOnFailureListener(e -> {
        Log.e("DFM", "Install failed: " + e.getMessage());
    });

// Monitor installation
splitInstallManager.registerListener(state -> {
    switch (state.status()) {
        case SplitInstallSessionStatus.DOWNLOADING:
            long downloaded = state.bytesDownloaded();
            long total = state.totalBytesToDownload();
            int percent = (int) (downloaded * 100 / total);
            // update progress bar
            break;
        case SplitInstallSessionStatus.INSTALLED:
            // Launch AR activity from dynamic module
            Intent intent = new Intent();
            intent.setClassName(getPackageName(),
                "com.myapp.ar.ArPreviewActivity");
            startActivity(intent);
            break;
        case SplitInstallSessionStatus.FAILED:
            Log.e("DFM", "Failed: " + state.errorCode());
            break;
    }
});
```

---

## 90.7 สรุป Part 90

ในบทนี้คุณได้เรียนรู้:

✅ AAB vs APK differences  
✅ Complete release checklist  
✅ Generate and manage release keystore  
✅ Sign APK/AAB  
✅ Google Play plugin for automated upload  
✅ Store listing checklist  
✅ Release notes in Thai  
✅ Dynamic Feature Modules (install on demand)  

---

*[← Part 89: CI/CD](./part-89-android-cicd.md) | [Part 91: Java Reflection & Annotation Processing →](./part-91-java-reflection.md)*
