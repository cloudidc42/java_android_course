# Part 55: Play Store Publishing
## หลักสูตร Java & Android Development - ระดับ Professional Android

---

## 55.1 เตรียมแอปก่อน Publish

```
Checklist ก่อน publish:
□ Version code และ version name ถูกต้อง
□ applicationId ไม่ซ้ำกับแอปอื่น
□ minSdk และ targetSdk เหมาะสม
□ Remove all debug logs
□ debuggable = false
□ ProGuard/R8 เปิดใช้งาน
□ Test บน real device
□ Test offline mode
□ รูปภาพและ icon ขนาดถูกต้อง
□ Screenshot ของแต่ละหน้าจอ
□ Privacy Policy URL พร้อม
□ App description เขียนเรียบร้อย
```

---

## 55.2 App Icon Sizes

```
Launcher Icons (PNG):
- mdpi:    48 x 48 px   (res/mipmap-mdpi)
- hdpi:    72 x 72 px   (res/mipmap-hdpi)
- xhdpi:   96 x 96 px   (res/mipmap-xhdpi)
- xxhdpi: 144 x 144 px  (res/mipmap-xxhdpi)
- xxxhdpi:192 x 192 px  (res/mipmap-xxxhdpi)

Adaptive Icon (Android 8+):
- Foreground: 108 x 108 dp (res/mipmap-xxxhdpi/ic_launcher_foreground.png)
- Background: 108 x 108 dp (res/mipmap-xxxhdpi/ic_launcher_background.png)
- Safe zone:  66 x 66 dp center

Play Store:
- High-res icon: 512 x 512 px (PNG, <= 1MB)
- Feature graphic: 1024 x 500 px
- Screenshots: min 320px, max 3840px
  Phone: 2 - 8 screenshots
  Tablet: up to 8 screenshots
```

---

## 55.3 Build Release AAB

```groovy
// build.gradle
android {
    defaultConfig {
        applicationId "com.yourcompany.yourapp"
        minSdk 26
        targetSdk 34
        versionCode 1
        versionName "1.0.0"
    }
    
    signingConfigs {
        release {
            storeFile file("../keystore/release.jks")
            storePassword "keystorePassword"
            keyAlias "release"
            keyPassword "keyPassword"
        }
    }
    
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                'proguard-rules.pro'
        }
    }
    
    // Bundle config
    bundle {
        language { enableSplit = true }
        density { enableSplit = true }
        abi { enableSplit = true }
    }
}
```

```bash
# Build release bundle
./gradlew bundleRelease

# Output: app/build/outputs/bundle/release/app-release.aab
```

---

## 55.4 Create Keystore

```bash
# Generate keystore (do this ONCE, keep safe forever!)
keytool -genkey -v -keystore release.jks \
  -alias release \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -dname "CN=Your Name, OU=Your Unit, O=Your Company, L=Bangkok, ST=Bangkok, C=TH"

# Enter keystore password, key password

# Verify keystore
keytool -list -v -keystore release.jks -alias release

# IMPORTANT: Back up your keystore!
# If you lose it, you CANNOT update your app on Play Store
```

---

## 55.5 Play Store Listing (Thai)

```
App title (50 chars max):
"ชื่อแอป - คำอธิบายสั้น"

Short description (80 chars):
"คำอธิบายสั้นที่ดึงดูดความสนใจ บอกสิ่งที่แอปทำได้"

Full description (4000 chars):
- ย่อหน้าแรก: ประโยชน์หลักของแอป
- ฟีเจอร์หลัก (bullet points)
- วิธีใช้งาน
- Call to action

Keywords ที่ควรใส่ใน description:
- คำที่ผู้ใช้ค้นหา
- ชื่อหมวดหมู่แอป
- ฟีเจอร์สำคัญ
```

---

## 55.6 Release Track Workflow

```
Internal Testing → Closed Testing → Open Testing → Production

Internal Testing:
- Max 100 testers (Google account)
- เข้าถึงได้ทันทีหลัง upload
- ใช้สำหรับ QA team

Closed Testing (Alpha/Beta):
- ผู้ทดสอบต้องรับเชิญ
- เหมาะสำหรับ beta users

Open Testing:
- ใครก็ได้ทดสอบ
- แสดงใน Play Store (marked as "Early Access")

Production:
- ทุกคนเห็นและดาวน์โหลดได้
- Phased rollout (10% → 50% → 100%)
```

---

## 55.7 App Versioning Strategy

```java
// Semantic Versioning: MAJOR.MINOR.PATCH
// 1.0.0 → Initial release
// 1.1.0 → New feature
// 1.1.1 → Bug fix
// 2.0.0 → Breaking change / major redesign

// Version code: always increasing integer
// Each release must have higher versionCode than previous

// Automate versionCode with date:
def getVersionCode = {
    return new Date().format('yyyyMMddHH').toInteger()
}

// Or sequential from git:
def getVersionCode = {
    'git rev-list --count HEAD'.execute().text.trim().toInteger()
}
```

---

## 55.8 In-App Updates

```java
// Check for updates with Play Core library
// build.gradle: implementation 'com.google.android.play:app-update:2.1.0'

public class UpdateManager {
    
    private final AppUpdateManager appUpdateManager;
    private final Activity activity;
    private static final int UPDATE_REQUEST_CODE = 100;
    
    public UpdateManager(Activity activity) {
        this.activity = activity;
        appUpdateManager = AppUpdateManagerFactory.create(activity);
    }
    
    public void checkForUpdate() {
        appUpdateManager.getAppUpdateInfo().addOnSuccessListener(info -> {
            if (info.updateAvailability() == UpdateAvailability.UPDATE_AVAILABLE) {
                
                if (info.isUpdateTypeAllowed(AppUpdateType.IMMEDIATE)) {
                    // Force immediate update
                    try {
                        appUpdateManager.startUpdateFlowForResult(
                            info, AppUpdateType.IMMEDIATE,
                            activity, UPDATE_REQUEST_CODE);
                    } catch (IntentSender.SendIntentException e) {
                        Log.e("Update", "Failed to start update flow");
                    }
                } else if (info.isUpdateTypeAllowed(AppUpdateType.FLEXIBLE)) {
                    // Background flexible update
                    try {
                        appUpdateManager.startUpdateFlowForResult(
                            info, AppUpdateType.FLEXIBLE,
                            activity, UPDATE_REQUEST_CODE);
                    } catch (IntentSender.SendIntentException e) {
                        Log.e("Update", "Failed to start flexible update");
                    }
                    
                    // Register install state listener
                    appUpdateManager.registerListener(state -> {
                        if (state.installStatus() == InstallStatus.DOWNLOADED) {
                            showUpdateReadySnackbar();
                        }
                    });
                }
            }
        });
    }
    
    private void showUpdateReadySnackbar() {
        Snackbar.make(activity.getWindow().getDecorView(),
            "อัปเดตพร้อมแล้ว", Snackbar.LENGTH_INDEFINITE)
            .setAction("รีสตาร์ท", v -> appUpdateManager.completeUpdate())
            .show();
    }
}
```

---

## 55.9 สรุป Part 55

ในบทนี้คุณได้เรียนรู้:

✅ Publish checklist  
✅ Icon sizes (mdpi ถึง xxxhdpi)  
✅ Build release AAB  
✅ Create and manage keystore  
✅ Play Store listing tips  
✅ Release tracks (Internal → Production)  
✅ Semantic versioning  
✅ In-app updates (Immediate, Flexible)  

---

*[← Part 54: CI/CD](./part-54-android-cicd.md) | [Part 56: Android Architecture Patterns →](./part-56-android-architecture.md)*
